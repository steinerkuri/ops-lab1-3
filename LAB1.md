# 实验 1 记录：第 3 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 3 组 |
| 仓库地址 | <https://github.com/steinerkuri/ops-lab1-3> |
| 代码载体 | 统一示例 |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9` |
| 合并方式 | squash |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | 见下方“第一次构建的 BUILD_INFO”。 |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | BUILD_INFO 没有变化；二进制文件的 sha256 发生变化。第一次：`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`；第二次：`1d14f317067f0345b881d1eb5a31c47e09654a858c638319871df4359afc2060`。 |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | 两次的 `vcs.revision` 和 `vcs.time` 相同；第一次为 `vcs.modified=false`，修改但未提交后，第二次变为 `vcs.modified=true`。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 可以根据 `name`、`version` 和 `build_no` 判断该包是什么，但 BUILD_INFO 没有记录构建机器和构建时间。`commit` 只能表示当时的 HEAD 提交，无法记录未提交的工作区修改。因此存在未提交修改时，BUILD_INFO 会让人误以为制品完全来自该 `commit`，不能准确回答使用了哪份代码构建。 |

### 第一次构建的 BUILD_INFO

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | 规则名：Require a pull request before merging、Require status checks to pass、Restrict deletions、Block force pushes；审批人数：1；必需检查名称：`ci`。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 见下方“直接推送被拒的终端输出”。 |
| 第一轮：失败的 PR 运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38848100553> |
| 第一轮：修复后通过的 PR 运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38851149470> |
| 第一轮：合入 main 后的运行（含制品） | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38927338456> |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | `weeksOverlap` 的周次交集判断漏掉了“双周对双周”：现有用例只覆盖单周对每周、双周对每周和单周对单周，把判断改成只认单周对单周后，双周班之间同一时段会被判为不冲突，`make test` 仍然全部通过。 |
| 第二轮：仅含缺陷时仍然通过的运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996089401> |
| 第二轮：补测试后失败的运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996208812> |
| 第二轮：撤销缺陷后通过的运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996316866> |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099258350> |
| Release v1.0.0 | <https://github.com/steinerkuri/ops-lab1-3/releases/tag/v1.0.0> |
| Release v1.0.1 | <https://github.com/steinerkuri/ops-lab1-3/releases/tag/v1.0.1> |

### 任务 5：发布与门禁验证

正常发布：`v1.0.0` 触发的 [release #1](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099035350) 成功。下载 Release 中的 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 和同名 `.sha256` 附件，在附件目录执行校验，结果如下；发布时的原始哈希同时记录在第 4 节。

```text
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
660288abe22e794edfe4a6f24c171cd9bfdfa94c3e2943f7cb6f6bd7e2807765  enroll-conflict_1.0.0_linux_amd64.tar.gz
```

反向测试：在 `scratch/gate-test` 分支创建未合入 main 的空提交，推送该分支及 `v0.0.1` Tag。[release #2](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099258350) 在 Release gate 失败，退出码为 `1`；Go 安装、质量检查、打包和创建 Release 步骤全部跳过，本次运行未执行发布。Gate 日志：

```text
Error: Release rejected: tag v0.0.1 points to 02d4ad4f6a66b8ba532a31192e745569a0710801, which is not an ancestor of origin/main.
Error: Process completed with exit code 1.
```

测试后已删除远程和本地的 `v0.0.1` Tag、`scratch/gate-test` 分支，保留正式版本 `v1.0.0` 及上述 Actions 运行记录。

### 直接推送被拒的终端输出

```text
(base) PS D:\labs_projects\Fall2026\OPS\lab1\ops-lab1-3> git push origin main
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 279 bytes | 279.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/steinerkuri/ops-lab1-3/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Changes must be made through a pull request.
remote:
remote: - Required status check "ci" is expected.
remote:
To https://github.com/steinerkuri/ops-lab1-3.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/steinerkuri/ops-lab1-3.git'
```

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | `03bf499674114709e5a6ccba93cabdb3506bc9e8` | `b432503aa2c5f32582e89e4a8e647c17a13c7211` |
| 引入该提交的 PR | [#4](https://github.com/steinerkuri/ops-lab1-3/pull/4) | [#6](https://github.com/steinerkuri/ops-lab1-3/pull/6) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | [release #1（39099035350）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099035350) | [release #3（39233851037）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39233851037) |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `660288abe22e794edfe4a6f24c171cd9bfdfa94c3e2943f7cb6f6bd7e2807765` | `b615394a5b8f809cfa7caf7bfb5ef9f69692131393d785cf06dd07a1d3c6c4da` |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？

不能。构建号在每个工作流内独立计数，`ci` 和 `release` 都会出现同一个号。v1.0.0 的构建号为 `1`：在 `ci` 中对应 [ci #1（38708020682）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38708020682)，在 `release` 中对应 [release #1（39099035350）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099035350)；该制品的 `BUILD_INFO` 记录了 `run_id=39099035350`，因此实际构建它的是 release #1，不是 ci #1。v1.0.1 同理，构建号为 `3`：`ci` 中的 3 号运行是 [ci #3（38840413626）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38840413626)，`release` 中的 3 号运行是 [release #3（39233851037）](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39233851037)，其 `run_id=39233851037` 指向 release #3。仅凭构建号不能唯一定位运行，只有 `run_id` 全局唯一。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`660288abe22e794edfe4a6f24c171cd9bfdfa94c3e2943f7cb6f6bd7e2807765`。与上表中发布当时的值一致，说明 v1.0.1 的发布没有覆盖 v1.0.0 的附件。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39099035350
commit=03bf499674114709e5a6ccba93cabdb3506bc9e8
go=go1.26.9
== 3. 提交 03bf499674114709e5a6ccba93cabdb3506bc9e8
作者: Jikuan Zhang <zhangjikuan77@163.com>
时间: 2026-10-18T11:06:38+08:00
标题: task5 (#4)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/steinerkuri/ops-lab1-3/commit/03bf499674114709e5a6ccba93cabdb3506bc9e8
构建它的运行: https://github.com/steinerkuri/ops-lab1-3/actions/runs/39099035350
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39233851037
commit=b432503aa2c5f32582e89e4a8e647c17a13c7211
go=go1.26.9
== 3. 提交 b432503aa2c5f32582e89e4a8e647c17a13c7211
作者: ruiyang sun <2391401143@qq.com>
时间: 2026-10-19T09:41:31+08:00
标题: docs: 记录 ci 与 release 工作流及发布方式 (#6)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/steinerkuri/ops-lab1-3/commit/b432503aa2c5f32582e89e4a8e647c17a13c7211
构建它的运行: https://github.com/steinerkuri/ops-lab1-3/actions/runs/39233851037
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
{"version":"1.0.1","commit":"b432503aa2c5f32582e89e4a8e647c17a13c7211","build_no":"3","go":"go1.26.9"}
{"version":"1.0.0","commit":"03bf499674114709e5a6ccba93cabdb3506bc9e8","build_no":"1","go":"go1.26.9"}
```

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

以下时间均从 GitHub 页面读取，为北京时间（UTC+8）。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-16 11:57:19 | <https://github.com/steinerkuri/ops-lab1-3/pull/2> |
| 第一次 `ci` 失败（运行结束） | 2026-10-16 11:57:38 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38848100553> |
| 推送修复提交 | 2026-10-16 12:32:01 | <https://github.com/steinerkuri/ops-lab1-3/commit/c946bbd4577ca762bf82e778c5d9af304be21885> |
| `ci` 通过（运行结束） | 2026-10-16 12:32:48 | <https://github.com/steinerkuri/ops-lab1-3/actions/runs/38851149470> |
| 合入 main | 2026-10-16 23:51:37 | <https://github.com/steinerkuri/ops-lab1-3/pull/2> |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#2](https://github.com/steinerkuri/ops-lab1-3/pull/2) | 2026-10-16 11:30 | 2026-10-16 23:51 | [2026-10-18 13:00](https://github.com/steinerkuri/ops-lab1-3/releases/tag/v1.0.0) | 2天1小时30分 |
| [#4](https://github.com/steinerkuri/ops-lab1-3/pull/4) | 2026-10-17 20:41 | 2026-10-18 11:06 | [2026-10-18 13:00](https://github.com/steinerkuri/ops-lab1-3/releases/tag/v1.0.0) | 16小时19分 |
| [#6](https://github.com/steinerkuri/ops-lab1-3/pull/6) | 2026-10-19 01:10 | 2026-10-19 09:41 | [2026-10-19 17:57](https://github.com/steinerkuri/ops-lab1-3/releases/tag/v1.0.1) | 16小时47分 |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| [ci #1](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38708020682) | push | 44 秒 | 首次主干 CI，成功生成 artifact |
| [ci #4](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38848100553) | PR | 16秒 | 第一轮：初始提交，测试失败 |
| [ci #5](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38851149470) | PR | 20秒 | 第一轮：修复后测试通过 |
| [ci #7](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38927338456) | push | 31秒 | 第一轮：合入 main 后构建并生成 artifact |
| [ci #8](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996089401) | PR | 16秒 | 第二轮：仅加入缺陷，现有测试仍通过 |
| [ci #9](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996208812) | PR | 16秒 | 第二轮：补充测试后失败 |
| [ci #10](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38996316866) | PR | 19秒 | 第二轮：撤销缺陷后通过 |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

**所用数据**：§5 时间线——PR #2 发起于 11:57:19，ci #4 失败于 11:57:38（16 秒），修复提交于 12:32:01，ci #5 通过于 12:32:48（20 秒），合入 main 为 23:51:37。§5 提交到发布间隔：#2 为 2 天 1 小时 30 分，#4 为 16 小时 19 分，#6 为 16 小时 47 分。§5 ci 时长：PR 触发为 16～20 秒，push 到 main 为 31～44 秒。

**结论**：第一轮 gate 拦住的是测试能表达的回归：`weeksOverlap` 被改成恒真后，单周班与双周班被判为冲突，ci #4 失败，合并被阻止。第二轮 gate 没有拦住只含缺陷的 PR：双周对双周的漏判不在现有用例里，ci #8 仍然通过；补上用例后 ci #9 失败，说明 gate 的有效范围等于测试覆盖的范围。第一轮因失败多花约 35 分 10 秒（11:57:38 到 12:32:48），其中人工定位与修改 34 分 23 秒，等待 ci 共 36 秒（失败 16 秒、修复后 20 秒），其余约 11 秒是推送与排队。一次 ci 约 16～20 秒，占 #4 的提交到发布间隔（16 小时 19 分）约 0.03%，占 #6（16 小时 47 分）约 0.03%，不到千分之一。第二轮补充测试之后，测试本身仍无法发现两类问题：一是没有被写进用例的场景（用例缺失，缺陷仍会漏过），二是流程与配置类问题（tag 打错提交、版本号写错、runner 或依赖版本漂移、审批漏掉），它们不在测试的判定范围内。

**成立条件**：结论以测试可靠且覆盖相关行为、ci 稳定运行为前提；样本只有两轮，ci 时长受共享 runner 负载影响；#2 的合入比修复晚约 11 小时，主要是等人在线审批，这部分等待不属于 gate 引入的成本，因此占比用 #4、#6 计算更合适。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：[PR #2](https://github.com/steinerkuri/ops-lab1-3/pull/2) 合并前后的 CI 日志。

| 运行 | 触发事件 | `github.sha` | `github.event.pull_request.head.sha` |
|---|---|---|---|
| ci #6 | `pull_request` | `74011e54bf103d9009f30906e035ae6749073c38` | `270f32dc47f7c36c5d90ab365d5948488f120517` |
| [ci #7](https://github.com/steinerkuri/ops-lab1-3/actions/runs/38927338456) | `push` 到 main | `46437cdde842d86d530679bb423b2684fe990d03` | 空（本次事件不是 PR） |

`github.sha` 与 `pull_request.head.sha` 不同。

`github.sha`：含义取决于触发事件，比如 ci #6 就是 GitHub 将 PR 分支上最新提交与当前 main 内容预合并后，构建的一个临时测试提交；ci #7 是正式 main 提交。

`pull_request.head.sha`：当前 PR 分支上的最新提交。

该项目中，只有 ci #7 的 `github.sha` 能在 main 的历史中找到。

本组使用的合并方式是 squash and merge，即同意 PR 时，在 main history 中创建一个新提交记入 main history。该新提交包含本次 PR 的最终改动。

待发布制品只能从 main 上的提交/tag 构建，是为了建立一个清楚的对应关系：

发布的压缩包 → 确定的 main 提交及对应的 PR、审批和测试记录。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

本地实验使用独立工作目录 `/tmp/ops-lab1-question3`，固定提交为 `a52d6326c167f6955eb48bd12c46610340f9e794`，Go 为 `1.26.9`，目标为 `linux/amd64`、`CGO_ENABLED=0`；①开始前工作区干净。①②④⑤的参数均为 `VERSION=1.0.0 BUILD_NO=100 RUN_ID=1000 COMMIT=a52d6326c167f6955eb48bd12c46610340f9e794`，③仅将 `BUILD_NO` 改为 `101`；这些版本号与运行号是本地实验参数，不代表已发布的 v1.0.0 或真实 Actions 运行。④⑤只临时删除 tar 命令的 `--mtime=@0`，实验后已执行 `git checkout -- Makefile` 并确认工作区恢复干净。

Actions 对比选择 main 上的 push 运行 [ci #30](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39747186247)，提交与本地实验相同；查看该运行的 attempt 1 和 attempt 2，证据均来自 `Package main build` 步骤最后两行。两次均成功，构建参数均为 `VERSION=0.0.0-main`、`BUILD_NO=30`、`RUN_ID=39747186247` 和上述 `COMMIT`，第一次总耗时 33 秒，重跑后 26 秒；哈希比较的是这两次 Actions 构建，不与参数不同的本地构建直接比较。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | `5e974dc5145bc88b8f009d5d7e8e43cdfcc0eb14e6ac61ee3d1f7c533d67300c` | `7d73602dc4cc277c1cfc409283cea2e740e63cec2e45927a1d9bb7510d48fdbe` |
| ② | 与 ① 完全相同，再构建一次 | `5e974dc5145bc88b8f009d5d7e8e43cdfcc0eb14e6ac61ee3d1f7c533d67300c` | `7d73602dc4cc277c1cfc409283cea2e740e63cec2e45927a1d9bb7510d48fdbe` |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | `d1627f8b3b63859b2c9d5eb28f7df3b27e590be66602989afa2e3c596902d814` | `8f7c5bdbee6780811c1d92e6d34245204e35951023b8fdce6074e8e10c5efd40` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `5e974dc5145bc88b8f009d5d7e8e43cdfcc0eb14e6ac61ee3d1f7c533d67300c` | `6484288b323cd2b997e69a005293d633ff099a34c946a2631a605ed47fac7ec7` |
| ⑤ | 同 ④，间隔几秒后再构建一次 | `5e974dc5145bc88b8f009d5d7e8e43cdfcc0eb14e6ac61ee3d1f7c533d67300c` | `1c82c011ad87c8eb9df91c716d7716392ed0cd8f101dffec1f972b9d078f8482` |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | attempt 1、2 均为 `f948a814e04f398ae4960f77dcbb331b10d31db777253a8d4625a661a11cfa6e` | attempt 1、2 均为 `bef37139132c83b44c0507734feaca5b32c12c22113ed6e0e9cc6d42551d5541` |

补充证据：①④⑤的 `go version -m` 输出均未出现 `vcs.revision`、`vcs.time` 或 `vcs.modified` 行，其构建设置相同：

```text
path    enroll-conflict
mod     enroll-conflict (devel)
build   -buildmode=exe
build   -compiler=gc
build   -trimpath=true
build   CGO_ENABLED=0
build   GOARCH=amd64
build   GOOS=linux
build   GOAMD64=v1
```

`tar --full-time -tvzf` 的输出显示，④中 `BUILD_INFO` 和二进制文件的归档修改时间均为 `2026-10-22 17:42:03`，⑤中均为 `2026-10-22 17:42:13`；以上为终端原样输出，两次相差 10 秒。

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

①与②的二进制和压缩包哈希均相同，说明在本次固定源码、工具链、构建参数及环境的条件下，重复构建得到了相同制品；③仅改变构建号，但它同时通过链接参数写入二进制并写入 `BUILD_INFO`，所以两个哈希都改变了。④⑤的二进制与①相同，但去掉 `--mtime=@0` 后，归档保存了实际文件修改时间，④⑤之间文件时间又发生变化，因此④⑤的压缩包既不同于①，也彼此不同，即使保留 `gzip -n` 也不能消除 tar 内部的时间差异。本次①④⑤均未显示 Go 的 `vcs.*` 信息，不能把差异归因于 `vcs.modified`，而任务 0 已观察到包含 VCS 信息的构建会记录工作区修改状态，所以其他环境中④的二进制仍可能因该状态变化而不同。Actions 的两次尝试保持构建号 `30`、运行 ID 和提交不变，两个哈希也分别相同，说明这次重跑可复现，但不能据此保证更换工具链、依赖、文件权限或构建环境后仍然相同。制品应包含版本号、完整提交号、构建号、运行 ID 和工具链版本等可核对的追溯信息，并保留准确的工作区状态信息；不应为了追求相同哈希而隐藏脏工作区，也不应混入每次变化的当前构建时间、机器专属绝对路径等无关信息，归档顺序、属主和时间则应统一。`release` 重新编译是因为 main CI 的版本号是 `0.0.0-main`，正式版本需从 Tag 得到并通过链接参数写入二进制，同时还要写入 release 自己的构建号和运行 ID，直接发布 CI 压缩包或只改文件名不能完成这些替换。这在 CI 制品到 Release 制品之间偏离了“一次构建、多处部署”，代价是重复测试与编译耗时，以及重新构建时环境变化的风险，因此 release 必须重新进行质量检查，不能把 CI 制品的验证直接当作发布制品的全部验证。正式 Release 生成以后，各次部署和回退应下载并校验同一份附件，不再编译，这样部署阶段仍遵守“一次构建、多处部署”。

运行编号说明依据：[GitHub Actions 变量文档](https://docs.github.com/en/actions/reference/workflows-and-actions/variables)规定，重跑保留 `GITHUB_RUN_NUMBER` 和 `GITHUB_RUN_ID`，递增 `GITHUB_RUN_ATTEMPT`；本工作流没有把 attempt 写入制品。


**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

**所用数据**：§4 表——v1.0.0 由 release #1（`run_id=39099035350`）从 `03bf499` 构建，sha256 `660288ab…`；v1.0.1 由 release #3（`run_id=39233851037`）从 `b432503` 构建，sha256 `b615394a…`。两次 `/version` 分别是 `1.0.1`/`build_no=3` 和 `1.0.0`/`build_no=1`。两个 Release 的 `immutable` 都是 `false`。

**结论**：v1.0.1 发现缺陷，应发 v1.0.2 并在 v1.0.1 的说明里写明；要立刻止血，先用 v1.0.0 的附件退回去。不要删 tag 复用版本号——两版 sha256 已经不同，同号重发就分不清新旧了。回退用 Release 里的 `.tar.gz` 解压即用，不用重建，`/version` 会直接报出 `1.0.0` 和 `build_no=1`。从源码重建则拿不到同一个文件：默认参数是 `dev`/`local`，还有 `vcs.modified`、`vcs.time` 和 Go 版本要一致。附件被换、`.sha256` 一起换，`trace.sh` 发现不了，它只校验两者互洽，不校验来源。但 §4 记的 sha256 能发现——重下再算一次就对不上。

**成立条件**：版本号、tag、内容一一对应且发布后不变，§4 的值是发布当时从 Release 算的。当前 `immutable=false`，平台不设防；开了 Immutable releases 才能强制附件不可变。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | [image job 日志链接](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39749956054/job/119727947779)；[镜像摘要](sha256:60ab86e8cedb12c019c49d629c143495660ae98de32bc37ecf7055f7351379e5) x86_64 主机运行，具体记录见下面O1说明。 |
| O2 缓存对反馈时间的影响 | 见下面O2说明 |
| O3 迁移到小组项目 | 未做 |

O1说明：
- `image` job 通过 `needs: release` 等待发布完成，下载该版本的压缩包和 `.sha256`，校验附件及 BUILD_INFO 中的版本、提交号，再使用仓库已有的 Dockerfile 构建镜像，镜像内不重新编译。镜像地址使用小写仓库路径 `ghcr.io/steinerkuri/ops-lab1-3`。镜像推送到远端后，按 digest 启动容器，检查 `/version`，并逐字节比较容器内的程序、BUILD_INFO 与 Release 附件是否一致；digest 和运行结果写入该次 Actions 的 Summary。容器运行主机使用 GitHub Actions 的 `ubuntu-24.04` runner，并通过 `uname -m` 记录实际架构。无需在 ECS 安装 Docker（因为sudo命令需要密码，故没有用服务器）。

- 镜像 job 放在同一个工作流中，是因为本实验使用 `GITHUB_TOKEN` 创建 Release，该操作产生的 `release` 事件不会触发另一个工作流。
- GHCR 镜像包默认私有，关联仓库后可继承访问权限，但不会因此自动继承仓库的公开可见性。本次使用无登录状态的空 Docker 配置，按 digest pull镜像成功，说明该镜像支持匿名拉取。

- 实测记录（2026-10-22，北京时间 UTC+8）：

| 项目 | 实际结果 |
|---|---|
| 发布版本及时间 | `v1.0.2`，2026-10-22 16:44:32 |
| tag 对应的 main 提交 | [`a52d6326c167f6955eb48bd12c46610340f9e794`](https://github.com/steinerkuri/ops-lab1-3/commit/a52d6326c167f6955eb48bd12c46610340f9e794) |
| 构建运行 | `release #4`，actions分配的`run_id=39749956054`；`release` 和 `image` 两个 job 均成功 |
| 使用的 Release 附件 | `enroll-conflict_1.0.2_linux_amd64.tar.gz` 及其 `.sha256`；校验结果 `OK` |
| 附件 sha256 | `01f18f381c6c8626052f61b452a4e201863ac3406bd1ac1de720eeaee45459b0` |
| 镜像名称及 tag | `ghcr.io/steinerkuri/ops-lab1-3:v1.0.2` |
| 镜像 digest | `sha256:60ab86e8cedb12c019c49d629c143495660ae98de32bc37ecf7055f7351379e5` |
| 镜像平台 | `linux/amd64` |
| 运行主机 | GitHub Actions `ubuntu-24.04` runner，执行 `uname -m` 输出 `x86_64` |
| 容器内容校验 | 程序和 `BUILD_INFO` 均通过命令 `cmp`，与 Release 附件中对应文件一致 |
| 匿名拉取 | 日志输出 `Anonymous pull: succeeded (no registry credentials).`，成功 |

Actions日志：

```text
Runtime host: GitHub Actions ubuntu-24.04; architecture: x86_64
Running image by digest: ghcr.io/steinerkuri/ops-lab1-3@sha256:60ab86e8cedb12c019c49d629c143495660ae98de32bc37ecf7055f7351379e5
{"version":"1.0.2","commit":"a52d6326c167f6955eb48bd12c46610340f9e794","build_no":"4","go":"go1.26.9"}
```

可复现的运行命令（按照 digest运行， 需要支持该镜像平台的 Docker 环境）

```sh
docker run --rm --platform linux/amd64 -p 127.0.0.1:18080:8080 \
  ghcr.io/steinerkuri/ops-lab1-3@sha256:60ab86e8cedb12c019c49d629c143495660ae98de32bc37ecf7055f7351379e5
```

在另一个终端执行 `curl http://127.0.0.1:18080/version` 就可以查看版本。

O2说明：
实验 Draft PR（未合并）：<https://github.com/steinerkuri/ops-lab1-3/pull/11>

实验预期：统一示例没有第三方依赖，因此预期 `setup-go` 缓存只能节省少量时间，缓存收益可能被共享 runner 的负载、网络、初始化状态等波动掩盖。实验使用同一个 Draft PR：先显式设置 `cache: true`，初次运行后执行4次 **Re-run all jobs**；再改为 `cache: false`，同样执行初次运行和4次 Re-run。两组均记录完整 workflow 时长，项目代码及其他 CI 步骤保持不变。

| 缓存设置 | Actions 运行 | Attempt 1 | Attempt 2 | Attempt 3 | Attempt 4 | Attempt 5 | 平均值 | 中位数 | 范围 |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `cache: true` | [run 39740200681](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39740200681) | 14秒 | 19秒 | 24秒 | 22秒 | 18秒 | 19.4秒 | 19秒 | 14～24秒 |
| `cache: false` | [run 39741509602](https://github.com/steinerkuri/ops-lab1-3/actions/runs/39741509602) | 32秒 | 36秒 | 37秒 | 37秒 | 29秒 | 34.2秒 | 36秒 | 29～37秒 |

本次实验中，有缓存组平均用时19.4秒，无缓存组平均用时34.2秒；关闭缓存后平均增加14.8秒，约比有缓存组长76.3%，没有支持“缓存只能节省少量时间”的运行前预期。`setup-go` 的缓存不仅涉及模块下载目录，也可能复用 `GOCACHE` 中的 Go 构建输出，因此即使示例没有第三方依赖，`go vet`、`go test` 及相关编译步骤仍可能受益。与此同时，缓存查询、下载、解压和保存本身也存在开销，所以缓存对总时长的影响取决于避免的编译时间能否抵消缓存操作成本。

单次时长会受到共享 runner 的 CPU、磁盘、网络、初始化状态和 GitHub 服务负载影响；缓存是否命中、缓存文件大小及恢复速度也会产生波动。本实验每组只有5次，并且先连续运行有缓存组、再连续运行无缓存组，没有交错或随机化，两个时间段的 runner 负载差异可能成为混杂因素。记录的是完整 workflow 时长，其中还包括 checkout、Actions 初始化等与 Go缓存无关的步骤。因此，本实验只能说明在本次时间段和当前仓库中观察到有缓存组更快，不能断言14.8秒的差异全部由缓存造成，也不能将该结果直接推广到其他项目。

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| steinerkuri | #1、#12、#16 | #6、#9、#10、#13、#14、#15 |
| 6waterMMM | #2、#9、#13 | #1、#12 |
| scarlet-mei | #3、#10 | #2、#16 |
| jikuanZhang | #4、#5、#14 | #3 |
| grave-kakaka | #6、#15 | #4、#5 |
