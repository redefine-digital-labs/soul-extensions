# Model 唯一源码权威切换：隔离候选验证

结论：`candidate-r4` 已保留业务源码和权限，在隔离副本中形成一致依赖图。Model、完整 Core、Native 的生产构建通过，原有 Model 相关跨包 VM 126/126 通过。实际两树尚未写入；源码权威切换由根协调任务统一执行。准确可审核载荷为 `integration-payload/payload.json`，不是把隔离相对路径直接复制到实际仓。

## 范围、基线和限制

- 实际输入：`/Users/naoer/.codex/worktrees/soul-dof-v1-rebuild/soulidity`；实际新 Model 候选：`/Users/naoer/Documents/Claude/Projects/soulidity-ecosystem/soul-extensions/move/soulidity_model`。
- 隔离工作目录为本目录。`baseline/` 逐字复制实际旧 Model、Core、Application 和公开 producer manifest；Core 仍为 `../soulidity_model`，旧 Model 仍为 `../soulidity_standard`、`../soulidity_foundation`、`../soulidity_visual`。Standard/Community/Visual 原定位链接在隔离副本中指向隔离复制源。
- 因此 baseline 是对实际旧 manifest 的隔离新鲜构建，不是实际目录的历史原产物，也不是先改成候选 manifest 后的伪基线。
- 固定 CLI 为实际 `output/contract-rebuild/local-ptb-2026-10-08/tooling/sui`，SHA256 `1d7baa7c7314113671415acfa20279b1eedb6ae6d04f286988a00da285e769c3`。生产构建均带 `--dump-bytecode-as-base64 --no-tree-shaking --with-unpublished-dependencies`；VM 使用默认 gas 和 package-size。
- 写入只发生在本隔离目录。无签名、RPC、链切换、源码删守卫/导出 Cap/UID、Git commit/push、远端、部署或产品行为修改。

## 依赖发现与最小可执行候选

Sui 对本地包使用归一化词法定位路径作为 dependency identity。文件系统 realpath/inode 相同并不足以让两个定位路径在 compiler graph 中合并。

| 轮次 | 实际改动 | 生产结果和明确诊断 |
| --- | --- | --- |
| candidate | 旧 Model 链接到新仓；Foundation 的 `.deps` 改定位链接 | Model 单独 PASS；Core FAIL，旧 Model 定位路径锚点下 `../../.deps/soulidity_foundation` 不存在 |
| candidate-r2 | Core/Application/model_producer 显式依赖新 Model | Model PASS；Core FAIL：Foundation 地址重复；`.deps` symlink 与原 Foundation 路径虽然 inode 相同，resolver identity 不同 |
| candidate-r3 | 新 Model Foundation 直接指真实 retained Foundation | Model PASS；Core FAIL：Standard 地址重复；新仓 Standard 真实路径与原定位链接仍是不同 resolver identity |
| candidate-r4 | 新 Model 三个 local provider 全部使用 retained 的规范定位路径；三个 Model 消费者使用新 Model 真实目录 | Model/Core/Native 生产 PASS；单一 resolver identity 与单一实际 authority 同时成立 |

最终只需 **4 份 manifest、6 个 local 字符串**：新 Model 的 Standard/Foundation/Visual 三项，以及 Core/Application/model_producer 的 Model 各一项。各提供方、版本、生产源码、公开 API、abilities、权限、费用和守卫均无改动。Foundation 不必先搬仓，也没有第二套可编辑源码。

`emit_locators.py` 接受 `--authority-root --ecosystem-root --output`，只向输出目录生成这些准确 manifest 和完整 before 副本。它按当前 checkout 布局重新生成直接相对路径，无须在脚本中硬编码主机路径。模型的 provider 身份固定为 source-root 的既有 `move/{soulidity_standard,soulidity_foundation,soulidity_visual}`；Standard/Visual 仍真实解析到已迁新仓。

这是当前本地施工的可重建 locator 方案。它不是不可变发布 pin；移动 checkout 后须重生成并重验。远端 CI/独立发布仍需沿原授权固定 immutable release，而不能把本机定位路径当发布证据。

## 消费者和源码证明

`consumer-references.json` 保存实际树 235 条引用，包含 25 份生产源码路径（包括 Model 自己的 4 份）、23 份测试路径、7 份工具脚本和相关文档。Model 的活跃直接包消费者是 Core、Application 和 IndependentModelProducer；OutputExtension/VisualCaptureExtension 间接经 Application 消费 Model。Native 不直接消费 Model，仍完整保留并独立做生产 root build。

新市场候选另有 Model 引用，但指 `candidate-dependencies/soulidity_model` 的冻结只读构建图。该候选未切原活跃消费者，本轮不将其改为另一维护权威、不冒充已完成市场独立发布。原脚本/记录的 Model 旧路径由根切换后的 locator symlink 继续定位同一实际维护源。

`source-equivalence.json`：52 份生产源码和 66 份测试源码均逐 SHA 保持；四份 Model 源码与新仓候选逐 SHA 相同。此验证不把生产声明存在当语义验收。

## 构建、VM 和制品证据

逐命令证据和原日志在 `evidence/baseline/` 与 `evidence/candidate-r4/`。受影响既有测试均在两种图运行：Source 14、Components 8、Compact binding 4、Assembly/Scene 26、Material/Physical 74，共 126/126 PASS。Model 自身没有既有单包测试，零测试不计覆盖。Application 与公开示例的独立 production build 结果见最终 `summary.json`。

`artifact-comparison/*.json` 逐 v7 模块解析 address tables 和其索引，在每个比较图建立 package 自身地址双射，只允许对 AddressIdentifiers、ModuleHandles/FriendDecls 做重定位，然后要求全字节相等。Model 的 99、Core 的 174、Native 的 141 模块在本组比较中 **raw 全字节已经相等**，重定位数为 0；对应 dump JSON 和 digest 也相同。这些按每个 build 计数，不能加总成互异模块数量。

另直接读取既存实际 Core 生产产物 `soulidity-ecosystem/build-evidence/retained-core-after-three-migrations/build.json`。其原 evidence 指明真实 `--path` 指实际 Core；该文件与隔离 candidate-r4 dump 逐字相同，SHA256 均为 `35115d3b9c5b027712d918b445f1cbf22e1af6da9d46d0af19807bf3cef2c8dd`、52 个 dump 模块。精确路径与界限见 `historical-actual-core-comparison.json`。

以上只证明给定 CLI、源码和未发布编译图。根实际导入后仍须捕获新的 actual raw/lock/source identities，不能把隔离或历史比较当作已完成 actual 切换。未发布 dummy 地址/编译 digest 不能证明链上 package ID、type origin、升级 linkage 或真实权限接受。不同 published graph 不因源码等价而等同。

输入 package 锁未被修改，因为命令使用独立 `--install-dir`；成功生成的锁在各 evidence 目录。Model/Core 的生成锁变化限于 local source locator 和相应 manifest_digest；Native 生成锁相同。相关 diff 在 artifact-comparison 中。根应在实际布局重建并保留准确生成锁，不复制隔离路径的锁。

## 根集成步骤与回滚

1. 先确认实际源码仍匹配 source-inputs-before 和新 Model 的四 SHA；冻结实际旧 Model 整包、受影响四 manifest/锁，以及新仓旧 Model manifest/锁与 `.deps` Foundation 输入。
2. 逐项核验 `integration-payload/payload.json` 的 before_sha256；安装其四份准确实际布局 manifest。
3. 将实际旧 Model 实体目录移入已有 frozen/evidence 位置保存；旧目录建 symlink 指新 Model。新 Model 是唯一可编辑 authority。失去活跃 import 的 `.deps` Foundation 保留为明确历史只读输入或移入历史 evidence，不作为依赖 fallback。
4. 用同一固定 CLI 在实际新 Model、保留 Core、Native、Application 和相关公开示例做 production root build。保留 actual source hash、raw dumps、生成 lock、命令日志；再运行这 126 条相关 VM，验证旧源码 locator 与新 Model 指向相同 authority。
5. 更新原 workspace/module/source_ownership/dependency-pins/README 的准确 source-state 与 evidence；Model 可标 active-source，不能标独立部署或完整产品已验收。原 Owner、记录、里程碑、CURRENT sole handoff、1459 duties 和主 goal 均保留。
6. 回滚只恢复已保存四 manifest/锁，移除新建 locator link 并移回冻结旧 Model；保留两轮 evidence。无链状态可回滚项，因为本轮未执行链动作。

## 原记录附件摘要

手册 v1.0 已读 1–5、8.3–8.4、9、12；OPERATING_MODEL 第8节。原 EXTENSIONS/Model 记录沿现有 `module.json` 原责任入口追加：阶段 Test；状态隔离 candidate 验证通过、actual integration pending；证据本目录；退出条件为实际4 manifest安装、唯一 authority/source hash、actual root builds/locks与126VM。权限/资金业务未改，金融模型与 EVM 专项不适用；独立生产发布、链上 writer/issuer/linkage、完整旅程与外部审计待核验。内部 AI 验证不等于具备资格的外部审计或生产放行。
