# 江城链察

本目录是江城链察原项目提交 `cb4b048` 的可移植源码快照，包含前后端源码、依赖锁文件、测试、概念媒体和演示材料。计划目标为 `bincode-model/wuhan-chain-investigator`，仓库是否存在及公开状态须以实际 GitHub 核验为准。

本快照排除本机 Codex 配置、完整交付 Harness、GitHub 门禁工作流、Git 历史、部署记录、凭据、运行数据及重复源码压缩包。业务目录中的 6 条 Harness 运行规则仍保留。原项目未被修改。本次只准备提交材料，没有重新执行构建、回归、联网调查或完整交付验收。历史测试记录与演示页中的数值只代表注明时间的执行摘要，不是本轮测试结果。

面向汉客松赛题二「以太坊链上异动调查 Agent」的独立项目。把交易、回执和 finalized 检查点汇成可复核证据，计算实际费用与成功外层转移值，再区分事实、线索和待验证解释。

部署目标：<https://wutiantian.cn/eth-investigator/>。线上状态见发布核验记录。 本项目有独立源码与后端，不依赖江城验真运行，不修改原西安网站。

## 解决的问题

- 浏览器里的交易值容易被误认成实际转出：失败交易的成功外层值为0，实际交易费仍单独计算。
- 只看 Gas 限额估不准费用：使用回执 `gasUsed × effectiveGasPrice`，存在 Blob 费用时加上实际 Blob 费用。
- 查一笔历史交易需要反复对照：核对交易、回执、规范区块、交易索引与 finalized 高度，并分开交易时间和采集时间。
- 告警常常没有解释依据：每条线索附证据字段、替代解释、下一步核查与明确局限，不把大额或失败直接认定为攻击。

## 当前可以运行

实时模式固定读取 Ethereum PublicNode。用户可以调查一笔格式有效的交易哈希，也可以抽样当前 finalized 区块前6笔交易。历史交易只要达到当前 finalized 高度并通过关联检查，就能调查；不会因交易发生得早而当作资料过期。

程序计算执行 Gas 费、Blob 数据费、合计实际费用、尝试值、成功外层值和 Gas 使用比例。规则包含失败、大额尝试值、较高 Gas 比例、最大额度 approve 调用形状。100 ETH 与95%是项目观察阈值，不代表恶意标准。地址只显示公开字段，不推断真实身份；成功外层值也不是账户净资产变化。

目录包括 **15项 Skills、5个工具岗位、6条 Harness 规则**。采集、验收、量化、解释和报告由确定性 Python 工具完成，没有连接语言模型。证据JSON可下载并离线重放；合成示例有独立标识和空的链上浏览器链接。实时读取失败明确报错，不自动替换成示例。

## 本地启动

需要 Python 3.9+、Node.js 22.12+。后端使用 Python 标准库。

```bash
npm install
python3 -m server.main --port 8775 --data-dir ./data
```

另开终端运行 `npm run dev`，访问 <http://127.0.0.1:5202/>。本地运行不要设置生产 `PUBLIC_ORIGIN`。

```bash
npm test
npm run lint
npm run typecheck
npm run build
```

生产后端启动方式：

```bash
PUBLIC_ORIGIN=https://wutiantian.cn python3 -m server.main --port 8775 --data-dir /独立数据目录
```

反向代理将 `/eth-investigator/api/` 映射到 `127.0.0.1:8775/api/`，并提供构建出的 `dist/`。后端只监听回环地址；当前无调查历史数据库，运行目录为独立预留目录。调查结果由浏览器下载保存。

## 验证与材料

`tests/test_chain.py` 与 `tests/test_http.py` 覆盖整数与费用计算、Blob字段、失败语义、指定历史交易、错误区块关联、pending拒绝、时效、篡改重放、请求体积和来源限制。本次43项后端测试通过；真实finalized区块6笔抽样与指定单笔调查均完成重放验证。执行 `npm test` 获取当前测试结果；合成夹具测试与真实运行分别记录，不能相互替代。

- [Skills Excel](public/resources/skills.xlsx) · [CSV](public/resources/skills.csv) · [JSON目录](public/resources/catalog.json)
- [API约定](docs/api-contract.json) · [官方赛题映射](docs/requirements-map.md)

## 实现边界与来源

当前使用单一公共 RPC，尚未独立验证以太坊共识或跨源真实性。范围是单笔交易或单区块有限样本，不提供全链监控、跨区块资金追踪、内部调用 trace、代币余额、实际 allowance 或商业身份归因。回执未签名、未上链；离线重放验证内部一致性，不证明快照本身真实。

赛题依据：[汉客松 S1 & ETH Wuhan 2026 选手手册](https://tokenark.feishu.cn/docx/Vn3hdD7s6okrftx9583cYgganMg)。技术接口依据：[Ethereum JSON-RPC](https://ethereum.org/en/developers/docs/apis/json-rpc/)。旧武汉原型的有界RPC读取与证据重放方法在此独立改写，并新增指定交易、精确费用及证据解释。原八行业展陈、行业规划清单和其他赛题没有作为本项目能力保留。


## 独立交付

网页页脚提供本项目的可编辑 PPT、PDF、使用说明和独立源码下载。源码包排除运行数据库、部署凭据和其他项目内容；在干净目录执行 `npm ci` 后可单独构建。`source.zip` 是按需生成文件，未包含在本快照中。需要网页源码下载入口时，先运行 `python3 scripts/package-source.py`，再运行 `npm run build`。仅构建不会生成源码压缩包。


## 2026-10-07 前端迭代与来源披露

本轮范围由作者确认：先做前端与无需凭据的整改。新版候选聚焦可读字号、主操作、结果与证据层级、立体流程卡片、移动端与低动效适配。发布状态以本轮门禁和部署回执为准。

| 分类 | 内容 | 来源与边界 |
| --- | --- | --- |
| 本轮之前已有 | 固定 Ethereum RPC 工具、规则校验、证据下载及重放、Skills 清单 | 延续 2026-10-06 独立项目；不把旧功能计作本轮新增 |
| 本轮新增 | 前端信息架构、字号体系、可控制的 3D 流程卡片、资料入口、整改披露 | 本仓库实现；动画是流程示意，不是模型运行或真实交易直播 |
| 视觉参考 | Verdikt 的单一主任务；Scam Shield 的证据账本；FinancialFocus 提示词中的卡片空间运动 | 参考站仅借鉴设计思路；本次原视频来自用户提示词2，不复制银行卡信息、支付记录或业务结论 |
| 已有第三方组件 | React、Vite、Lucide、Motion；玻璃着色器改编自 liquid-glass-js | 保留依赖锁文件、原始许可证及 public/licenses/liquid-glass-js-MIT.txt |

参考链接：[Verdikt](https://verdikt.shadrakbessanh.me/) · [Scam Shield](https://scam-shield-rouge.vercel.app/ledger) · [Three.js CSS 3D 官方示例](https://threejs.org/examples/css3d_periodictable.html)。本轮卡片使用 CSS 3D，不声称引入了 Three.js。

## 后续接入与待验证事项

大模型角色、跨主体签名与链上信誉登记尚未接入。未部署 BOT Chain 合约，未产生主网交易，也不能据此宣称覆盖 BOT Chain 分赛道或获得晋级资格。钱包地址代表可控制的链上账户，本身不保证真实身份、可信服务或抵御女巫攻击。质押和罚没需要明确的争议裁决与退出规则，不作为已实现能力宣传。

多来源交叉校验、真实服务切换、内部调用与地址标签仍以代码和真实执行证据为准，未做的保持待办。测试通过数只反映已执行范围。

本轮前端完善不等同于完成整份参赛整改清单。组队口径、官方截止时间、奖励与合约网络要求应以主办方当前确认结果为准；诊断文件中的建议不能替代官方通知。

完整交付 Harness 与页面里的业务运行规则为两套不同记录。开发、验证、验收和上线分别记录，不能互相替代。

### 本次追加：提示词 2 / 提示词 3

首页采用提示词2的三幕全屏视频，滚轮下翻到下一幕、上翻回到上一幕，连续惯性不会重复跳页。提示词3的双图鼠标光圈、1800px滚动影片及末端行业场景已移到 #/workspace 二级页。文案分别对应两项武汉比赛赛题。原工作台、记录、Skills/Agent/Harness、证据与下载统一放在 #/workspace 第二页；换页保留业务状态。三段原视频经官方浏览器下载并本地压缩，滚动影片用无信用卡品牌的科技场景；双层行业图保留桌面和移动WebP。来源和具体适配见 docs/SCENE-DESIGN-20261007.md，素材哈希见 docs/SCENE-ASSETS-20261007.json。概念视觉不代表运行成果，大模型与链上接入状态不变。
