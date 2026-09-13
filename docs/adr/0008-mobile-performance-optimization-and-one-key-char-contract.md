# ADR 0008: 移动端（Trime）性能手术式优化与一简字/虎单隐式契约规范

## 状态
已通过 (Accepted)

## 上下文 (Context)
在移动端（Android 同文输入法 / Trime）上：
1. **历史卡顿根因**：
   `tiger`（万象虎）、`mets`（二三整句）、`xumn`（原码整句）因继承万象拼音庞大流水线，加载了 303.7MB 的 `wanxiang-lts-zh-hans.gram` 语法模型、147 万条词的 `zuci/jichu.pro`（生成 112.1MB 的 `table.bin`）、7 个按键拦截 Lua 处理器，以及在滤镜中频繁调用 LevelDB DFS 递归深搜与每次击键强制垃圾回收（`collectgarbage("step", 20)`），导致移动端击键严重卡顿掉帧。
2. **重构回归教训**：
   在精简 `engine/translators` 时，将 `switch_translator` 误判为冗余模块并移除。然而在虎码方案中存在重要的**双轨分工契约**：
   - 26 个字母的**一简字**（如 `u`→的, `f`→一, `t`→我）与单字存放在 `tigress.dict.yaml`（`char_word` 配置项）；
   - 主词典 `tiger.dict.yaml`（`8105` 字表）中的一简字编码带有单引后缀（如 `￣u'`、`￣f'`），无法直接单键出字；
   - `lua_translator@*switch_translator` 是动态桥接并加载 `char_word` 的核心通道。
   一旦遗漏该翻译器，会导致所有 26 个一简字全部丢失。

## 决议 (Decision)
1. **显式固化轻量引擎与一简契约**：
   - 在 `tiger.schema.yaml`（及继承该方案的 `mets`、`xumn`）中，显式声明轻量引擎流水线，将 `lua_translator@*switch_translator` 固化在 `translators` 前排，永久保障一简字与虎单/虎词通道畅通。
   - 保留 `affix_segmentor@yin_add_user` 与 `script_translator@yin_add_user`，永久守护单引号 `'` 前缀与反引号 `` ` `` 后缀的拼音反查与字根拆分提示功能。
   - 保留 `history_translator@historySJ`（按 `z` 重复）与 `quick_symbol_text2`（双击单引号 `''` 重复）的双轨重复上屏能力。
2. **彻底禁用 303MB 语法模型**：
   - 显式设置 `grammar: { language: "" }`。形码重码率极低，断句依赖编码规则而非语义打分，禁用后释放数百兆内存并消除 Viterbi 动态规划打分延迟。
3. **基础词库精细剪枝**：
   - 新增 `zuci/jichu_lite.dict.yaml`，提取原 147.5 万词中权重 $\ge 200$ 的 32 万条高频核心词汇。`tiger.table.bin` 从 112MB 缩减至 24MB，保障常用二字、三字、四字词整句出词不碎裂。
4. **Lua 运行时去阻滞化**：
   - 注释掉 `lua/db.lua` 中每击强制 GC `collectgarbage("step", 20)`。
   - 改造 `lua/auto_commit.lua`，在码长 $< 4$ 时直接流式返回，恢复 Rime 惰性求值。
   - 修复 `lua/quick_symbol_text2.lua` 中处理器返回值类型规范（返回整数状态码 `2`，杜绝返回布尔值报错）。
5. **建立全维度黄金回归测试集（Golden Regression Suite）**：
   - 方案的任何变更，必须通过包含 **1码一简字（`u`, `f`, `t`, `r`, `o`, `c`, `n`, `j`, `m`, `d`）、2码字/词（`jx`）、4码词汇（`dgrn`）、拼音反查（`'ni`）** 的全维度自动化测试，100% 通过方可交付。

## 影响 (Consequences)
- **正面**：
  - 彻底解决 Trime 上三套 26 键虎码方案的击键掉帧卡顿，性能与九键虎完全一致；
  - 一简字、虎单虎词、拼音反查、重复上屏、整句组词全部完好；
  - 沉淀了自动化回归测试基线，从流程上杜绝此类回归 Bug 再次发生。
- **负面**：
  - 剔除了极低频冷僻网络词汇（权重 $< 200$ 的长尾词），但在移动端属于合理取舍。
