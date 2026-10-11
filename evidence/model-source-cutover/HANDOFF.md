# Model source cutover verifier handoff

- 唯一验证目录：`/Users/naoer/.codex/worktrees/model-cutover-cl_3aw16`；共享两树未写，现有 CURRENT sole pointer 未改。详见 `REPORT_CN.md` 和 `summary.json`。
- 根可集成：`integration-payload/payload.json` 是准确实际布局4份manifest/6个local值；`emit_locators.py` 仅生成、不会写原树。Model的三个provider用既有source-root规范locator；三个直接消费者指新Model真实路径。
- 固定CLI SHA256 `1d7baa7c7314113671415acfa20279b1eedb6ae6d04f286988a00da285e769c3`。baseline和r4各7个production root PASS；各126原有相关VM PASS；52生产/66测试源码SHA保持；单一词法resolver identity＋实际authority已在隔离和拟实际图核验。
- 准确原始制品范围：baseline是逐字复制旧manifest后的隔离新build。7组全部raw mv/dump/digest相同，重定位0；另与保存的实际Core旧production dump相同。仍须根捕获actual post-integration raw、锁和源身份，不能推定链上package/type origin/linkage或权限运行结果。
- generated Model/Core/Application/3例锁变化为locator/manifest_digest；Native生成锁不变。输入包锁因install-dir保持未改，隔离生成锁禁止直接带路径复制到实际布局。
- 根下一步：核before SHA→保存实际旧Model/manifests/locks→安装payload→旧Model实体移至frozen并定位symlink新仓→actual production roots/126VM→回写原source-state和evidence。原Owner、1459pending、全目标、未发布/未部署界限均保留。
- 回滚：恢复保存manifest/锁及旧Model实体；保留所有验证记录。无RPC/签名/链切换/commit/push。第1–3轮实际失败诊断全部保留。
- 已用原 `phase_handoff.py --dry-run` 生成保存 `HANDOFF-helper-draft.md`，未在实际仓写新handoff或CURRENT。
