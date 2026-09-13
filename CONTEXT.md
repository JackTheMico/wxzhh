# 9jianhu (九键虎)

九键虎是基于大衍/虎码字根系统，将 26 个英文字母无重码压缩至移动端 9 键九宫格的形码输入方案。

## Language

**二按一选 (Two-press-one-select)**:
连续敲击两个字母所属的九键数字键（1~9）后，敲击第 3 击数字键（1~9）以确定两字母相对位置组合的无重码定位算法。
_Avoid_: 坐标编码, 笛卡尔查表

**渐进式双轨输入 (Progressive Dual-track Input)**:
四字词输入时，前 4 击直出高频候选（零心算），第 5 击支持按首字所属位置（1/2/3）精准收窄，打满 6 击依然兼容原有二按一选严格无重码规则的输入模式。
_Avoid_: 纯盲打模式, 强制六击

**成语与四字流打词库 (Four-character Streaming Vocabulary)**:
由全量 2.3 万条四字成语与高频现代汉语四字词合并构成的特权词汇集合，通过真实语料词频统一加权，专供 4 击与 5 击快速出字。
_Avoid_: 全量四字词库, 纯成语表

**二字高频双轨流打 (Two-character Dual-track Streaming)**:
高频二字词既支持 4 击自然流打（首字 2 键 + 次字 2 键），又支持 6 击无重码盲打（首字 2 键 + 选位 + 次字 2 键 + 选位）。在 4 击空间内依据真实语料词频与四字成语统一竞争，高频二字词优先呈现。
_Avoid_: 纯 6 击二字词, 二字词专有独立键盘

**三字词渐进式流打 (Three-character Progressive Streaming)**:
三字词支持 4 击原生虎码流打（各字首码 + 末字次码）强置顶出词，达成“多字词全 4 击出词”大一统心智模型；同时对 Top 800 超高频三字词支持 3 击辅助可见（字1+字2+字3首键），不抢单字首选，即点即出。
_Avoid_: 强制末字次根心算选位, 6 击伪字对

**成语滤镜 (Idiom Filter)**:
挂载在 Rime 滤镜管道中的 Lua 模块，利用内存成语及高频四字词倒排索引，在用户输入 4 码和 5 码数字时向候选流前排动态注入高频词。
_Avoid_: 词典派生, 静态四码表

**定长顶屏 (Fixed-length Auto-commit)**:
当输入码长达到方案终态上限（6 码）且候选唯一时，无需敲击空格键，字符直接自动提交上屏的机制。
_Avoid_: 自动选字, 空格盲打

**双轨词典契约 (Dual-dictionary Architecture Contract)**:
在万象虎（tiger）、二三整句（mets）与原码整句（xumn）中，26 个一简字（如 u→的, f→一, t→我）与单字强依赖 `char_word` (`tigress.dict.yaml`)，由 `switch_translator` 动态桥接并受 char/word 开关控制；主词典 `script_translator` (`tiger.dict.yaml`) 专司整句多字词。任何重构与精简必须显式保留 `switch_translator`，杜绝一简字出字通道断供。
_Avoid_: 纯单一主词典假设, 误裁 switch_translator

**轻量反查契约 (Lightweight Reverse-lookup Contract)**:
保留 `'`（单引前缀）与 `` ` ``（反引后缀）引导的拼音反查（`yin_add_user`）及字根拆分提示通道。反查通道属于事件驱动，未输入引导符时零额外算力消耗，移动端必须完整保留。
_Avoid_: 粗暴砍除反查, 移动端盲目加载 300MB 语法模型

**全维度黄金回归基线 (Full-dimension Golden Test Baseline)**:
任何方案流水线或词库改动，交付前必须执行全维度自动化回归测试，强制覆盖：1码一简字（u, f, t, r, o, c, n, j, m, d）、2码字/词（jx）、4码词汇（dgrn）、拼音反查（'ni），确保 100% 全绿且零异常日志。
_Avoid_: 仅测长编码, 漏测一简字

