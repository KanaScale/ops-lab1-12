# 实验 1 记录：第 12 组

现场检查时将打开本文件，按其中的链接核对。证据以链接或文本形式给出；main 受保护后，本文件的修改也通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 12 组 |
| 仓库地址 | https://github.com/KanaScale/ops-lab1-12 |
| 代码载体 | 统一示例（`enroll-conflict`） |
| runner 与语言版本 | CI 已使用 `ubuntu-24.04`、Go `1.26.9`；任务 0 本地使用 Go `1.26.9` |
| 合并方式 | squash（仓库已设置只允许 squash merging） |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | `name=enroll-conflict`<br>`version=1.0.0`<br>`build_no=1`<br>`run_id=local`<br>`commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`go=go1.26.9` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | `diff` 输出“BUILD_INFO 没有变化”。二进制 sha256 发生变化：第一次 `f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`；修改后 `c3d34955681cdc98ee1f1d57270812f6e358b7817e269de1c9b42a77ccfb12ba`。 |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | 两次均为 `vcs=git`、`vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`、`vcs.time=2026-10-12T16:26:07Z`；`vcs.modified` 从 `false` 变为 `true`。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 能从 `name`、`version` 知道包的名称和版本；没有主机标识和构建时间，不能确定在哪台机器、何时构建。`commit` 仍是原提交号，但第二次构建含未提交改动，因此仅凭它会误以为制品完全来自该提交；无法准确确定所用代码。`vcs.time` 是提交时间，不是构建时间。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 任务 2：首次 CI 与主干制品 | [成功的 ci 运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39020295552)；artifact：ci-460db78f01fb5fab539badcb6f67763a662bb766-1；trace.sh 校验和通过，追溯到提交 460db78f01fb5fab539badcb6f67763a662bb766、构建号 1、运行号 39020295552。 |
| 规则集里启用的规则 | [lab1-main-merge-gate](https://github.com/KanaScale/ops-lab1-12/rules/25298191)：Active，仅适用于默认分支 main，绕过名单为空；Restrict deletions、Block force pushes、Require a pull request before merging（至少 1 人审批）、Require status checks to pass（必需检查 `ci`）。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 组长在 main 上创建空提交 `ea2d38f` 后执行 `git push origin main`，收到以下输出；验证后已将本地 main 恢复到 `origin/main`：<br>remote: error: GH013: Repository rule violations found for refs/heads/main.<br>remote: Review all repository rules at https://github.com/KanaScale/ops-lab1-12/rules?ref=refs%2Fheads%2Fmain<br>remote: - Changes must be made through a pull request.<br>remote: - Required status check "ci" is expected.<br>To https://github.com/KanaScale/ops-lab1-12.git<br> ! [remote rejected] main -> main (push declined due to repository rule violations)<br>error: failed to push some refs to 'https://github.com/KanaScale/ops-lab1-12.git' |
| 第一轮：失败的 PR 运行 | [PR #2](https://github.com/KanaScale/ops-lab1-12/pull/2) 的 [首次 ci 运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39214106124)：`make test` 发现单周班与双周班被误判为冲突，检查失败时 PR 状态为 BLOCKED。 |
| 第一轮：修复后通过的 PR 运行 | [修复提交 0c1513b](https://github.com/KanaScale/ops-lab1-12/commit/0c1513b02777843a8c855d895ae2555445a9a09b) 恢复单双周互不冲突的判断；[ci 运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39214314302) 通过。PR 的最终代码差异只新增“双周对双周冲突”测试用例。 |
| 第一轮：合入 main 后的运行（含制品） | [PR #2](https://github.com/KanaScale/ops-lab1-12/pull/2) 经 CynicDora 审批后 squash 合并，main 提交为 [7998cbf](https://github.com/KanaScale/ops-lab1-12/commit/7998cbf66ecb753c4bda4e0dc298b0d51118e9d9)；[main 上成功的 ci 运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39216123207) 上传 artifact `ci-7998cbf66ecb753c4bda4e0dc298b0d51118e9d9-8`。 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | 将 Section.Valid 的结束节次判断从 End < Start 改成 End <= Start，会把 Start=End 的单节课程错判为无效；原有 TestValid 只测试多节合法课程和其他非法输入，没有覆盖起止节次相等的合法边界。 |
| 第二轮：仅含缺陷时仍然通过的运行 | [PR #4 的 ci #11](https://github.com/KanaScale/ops-lab1-12/actions/runs/39223510487)：仅含缺陷的提交 6ac8747，ci 成功。 |
| 第二轮：补测试后失败的运行 | [PR #4 的 ci #12](https://github.com/KanaScale/ops-lab1-12/actions/runs/39224774268)：新增单节课程合法性测试后，ci 在 Test 步骤失败。 |
| 第二轮：撤销缺陷后通过的运行 | [PR #4 的 ci #13](https://github.com/KanaScale/ops-lab1-12/actions/runs/39225332486)：撤销缺陷后通过；[#4](https://github.com/KanaScale/ops-lab1-12/pull/4) 经 Har0v0i86 审批并 squash 合入，main 只新增测试。 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release 运行 39232007830](https://github.com/KanaScale/ops-lab1-12/actions/runs/39232007830)：未合入 main 的提交 7440b80 在 Release gate 步骤失败，后续打包与创建 Release 均跳过；测试用 v0.0.1 tag 和 scratch/gate-test 分支已删除。 |
| Release v1.0.0 | [v1.0.0](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.0)，发布于 2026-10-19 17:40:27（北京时间）；任务 6 发布前的附件复核见第 4 节。 |
| 任务 6：小改动、审批与合并 | CynicDora 的 [PR #6](https://github.com/KanaScale/ops-lab1-12/pull/6) 为 `/version` 增加 `Cache-Control: no-store` 及响应头测试；[PR ci #17](https://github.com/KanaScale/ops-lab1-12/actions/runs/39238364986) 通过，fireDragon198 于 2026-10-19 18:54:03（北京时间）正式 APPROVED，18:55:26 squash 合并为 [3fccca8](https://github.com/KanaScale/ops-lab1-12/commit/3fccca88908d51c86316dce5fcadc2c5dd14ca3f)，[main ci #18](https://github.com/KanaScale/ops-lab1-12/actions/runs/39240920512) 通过。 |
| Release v1.0.1 | [v1.0.1](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.1)，发布于 2026-10-19 18:57:31（北京时间）；[release #3](https://github.com/KanaScale/ops-lab1-12/actions/runs/39241133673) 的主干祖先检查、fmt-check、vet、test、打包与校验均通过。 |

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | `f03b6ea350257855fb5862eacaff5efcfcee81f0` | `3fccca88908d51c86316dce5fcadc2c5dd14ca3f` |
| 引入该提交的 PR | [#5](https://github.com/KanaScale/ops-lab1-12/pull/5) | [#6](https://github.com/KanaScale/ops-lab1-12/pull/6) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | [release #1 / run_id 39231789999](https://github.com/KanaScale/ops-lab1-12/actions/runs/39231789999) | [release #3 / run_id 39241133673](https://github.com/KanaScale/ops-lab1-12/actions/runs/39241133673) |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `ad67bf9b7ee40853a874b1e8ebedb7c676ee0e4ba8c70d49bef426d39c577870` | `16f9d837187746840f8e662a04ec76fef5dadcb26a631567b09b774ff257e494` |

v1.0.0 的上述 SHA-256 取自发布附件的 `.sha256`，并在 2026-10-19 18:27:48（北京时间，发布 v1.0.1 之前）重新下载核验为一致，作为后续回退的独立对照值；采集主机为 Ubuntu / x86_64。

仅凭构建号不能唯一确定是哪次运行。v1.0.0 的构建号为 1，分别存在 [ci #1 / run_id 39020295552](https://github.com/KanaScale/ops-lab1-12/actions/runs/39020295552) 和 [release #1 / run_id 39231789999](https://github.com/KanaScale/ops-lab1-12/actions/runs/39231789999)。两个工作流各自编号，须使用 `BUILD_INFO` 中的 `run_id` 定位实际构建。

发布 v1.0.1 之后，于 2026-10-19 18:58:38（北京时间）开始在同一台 WSL Ubuntu / x86_64 主机上重新下载两个版本的 Release 附件并采集以下输出；v1.0.0 的 sha256 为 `ad67bf9b7ee40853a874b1e8ebedb7c676ee0e4ba8c70d49bef426d39c577870`，与发布附件的 `.sha256`、原始 [release #1 日志](https://github.com/KanaScale/ops-lab1-12/actions/runs/39231789999) 以及发布 v1.0.1 前独立保存的值完全一致。v1.0.1 的哈希也与其 `.sha256` 和 [release #3 日志](https://github.com/KanaScale/ops-lab1-12/actions/runs/39241133673) 一致；两个 tag 均与各自 `BUILD_INFO` 的提交号一致，以下两次 `trace.sh` 的退出码均为 0。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39231789999
commit=f03b6ea350257855fb5862eacaff5efcfcee81f0
go=go1.26.9
== 3. 提交 f03b6ea350257855fb5862eacaff5efcfcee81f0
作者: Har0v0i86 <564197920@qq.com>
时间: 2026-10-19T17:38:36+08:00
标题: feat: add tag-triggered release workflow (#5)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/KanaScale/ops-lab1-12/commit/f03b6ea350257855fb5862eacaff5efcfcee81f0
构建它的运行: https://github.com/KanaScale/ops-lab1-12/actions/runs/39231789999
exit_code=0
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39241133673
commit=3fccca88908d51c86316dce5fcadc2c5dd14ca3f
go=go1.26.9
== 3. 提交 3fccca88908d51c86316dce5fcadc2c5dd14ca3f
作者: CynicDora <hyzhyzsh@sjtu.edu.cn>
时间: 2026-10-19T18:55:26+08:00
标题: fix: prevent caching of version responses (#6)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/KanaScale/ops-lab1-12/commit/3fccca88908d51c86316dce5fcadc2c5dd14ca3f
构建它的运行: https://github.com/KanaScale/ops-lab1-12/actions/runs/39241133673
exit_code=0
```

任务 6 第 4 步，从下载的 `.tar.gz` 中解包并直接运行二进制，未重新构建；先运行 v1.0.1，停止后在同一个 `127.0.0.1:18080` 端口运行 v1.0.0，两次 `/version` 的输出如下。

运行 v1.0.1（2026-10-19 18:58:46，北京时间）：

```json
{"version":"1.0.1","commit":"3fccca88908d51c86316dce5fcadc2c5dd14ca3f","build_no":"3","go":"go1.26.9"}
```

换回 v1.0.0（2026-10-19 18:58:47，北京时间）：

```json
{"version":"1.0.0","commit":"f03b6ea350257855fb5862eacaff5efcfcee81f0","build_no":"1","go":"go1.26.9"}
```

两次响应的四个字段均与各自 `BUILD_INFO` 一致；v1.0.1 响应包含 `Cache-Control: no-store`，v1.0.0 没有该响应头，符合本次改动。验证结束后，已停止本次启动的服务。

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-19 15:08:46（北京时间，GitHub PR 创建时间） | [PR #2](https://github.com/KanaScale/ops-lab1-12/pull/2) |
| 第一次 `ci` 失败（运行结束） | 2026-10-19 15:08:59（北京时间） | [失败运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39214106124) |
| 推送修复提交 | 2026-10-19 15:10:38～15:10:43（北京时间；前者为 GitHub 提交时间，后者为推送触发的 CI 创建时间） | [修复提交](https://github.com/KanaScale/ops-lab1-12/commit/0c1513b02777843a8c855d895ae2555445a9a09b)、[CI 运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39214314302) |
| `ci` 通过（运行结束） | 2026-10-19 15:10:59（北京时间） | [通过运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39214314302) |
| 合入 main | 2026-10-19 15:27:37（北京时间；CynicDora 于 15:27:28 审批） | [PR #2](https://github.com/KanaScale/ops-lab1-12/pull/2)、[squash 提交](https://github.com/KanaScale/ops-lab1-12/commit/7998cbf66ecb753c4bda4e0dc298b0d51118e9d9)、[main ci](https://github.com/KanaScale/ops-lab1-12/actions/runs/39216123207) |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#4](https://github.com/KanaScale/ops-lab1-12/pull/4) | [6ac8747](https://github.com/KanaScale/ops-lab1-12/commit/6ac8747f4e7180e6df3c3c65fd658101db21f23c)：2026-10-19 16:21:48（北京时间） | 2026-10-19 17:02:30（北京时间） | [v1.0.0](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.0)：2026-10-19 17:40:27（北京时间） | 1 小时 18 分 39 秒 |
| [#5](https://github.com/KanaScale/ops-lab1-12/pull/5) | [2ca520e](https://github.com/KanaScale/ops-lab1-12/commit/2ca520ed65467d359f6c01517e7a743ded79c5c4)：2026-10-19 17:22:57（北京时间） | 2026-10-19 17:38:37（北京时间） | [v1.0.0](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.0)：2026-10-19 17:40:27（北京时间） | 17 分 30 秒 |
| [#6](https://github.com/KanaScale/ops-lab1-12/pull/6) | [b9f9aaa](https://github.com/KanaScale/ops-lab1-12/commit/b9f9aaa91547e7c01fcc4ccb84e0e95eae961ecc)：2026-10-19 18:29:57（北京时间，author/committer 时间相同） | 2026-10-19 18:55:26（北京时间） | [v1.0.1](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.1)：2026-10-19 18:57:31（北京时间） | 27 分 34 秒 |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| [ci #11 / 39223510487](https://github.com/KanaScale/ops-lab1-12/actions/runs/39223510487) | PR #4（仅有缺陷） | 9 秒 | `ci` job：2026-10-19 16:32:41～16:32:50（北京时间），成功；不含排队时间。 |
| [ci #12 / 39224774268](https://github.com/KanaScale/ops-lab1-12/actions/runs/39224774268) | PR #4（补充测试） | 16 秒 | `ci` job：2026-10-19 16:43:17～16:43:33（北京时间），测试失败；不含排队时间。 |
| [ci #13 / 39225332486](https://github.com/KanaScale/ops-lab1-12/actions/runs/39225332486) | PR #4（撤销缺陷） | 10 秒 | `ci` job：2026-10-19 16:47:54～16:48:04（北京时间），成功；不含排队时间。 |
| [ci #17 / 39238364986](https://github.com/KanaScale/ops-lab1-12/actions/runs/39238364986) | PR #6 | 13 秒 | `ci` job：2026-10-19 18:34:03～18:34:16（北京时间），成功；不含排队时间。 |
| [ci #18 / 39240920512](https://github.com/KanaScale/ops-lab1-12/actions/runs/39240920512) | push main（PR #6 squash 合并） | 17 秒 | `ci` job：2026-10-19 18:55:31～18:55:48（北京时间），成功并上传主干制品；不含排队时间。 |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

所用数据：第 3 节 [PR #2](https://github.com/KanaScale/ops-lab1-12/pull/2) 和 [PR #4](https://github.com/KanaScale/ops-lab1-12/pull/4) 的检查记录、第 5 节第一轮时间线、五次 ci job 时长（9、16、10、13、17 秒）及 #4、#5、#6 的提交到发布间隔（1 小时 18 分 39 秒、17 分 30 秒、27 分 34 秒）。

第一轮的 gate 拦住了现有测试能发现的单双周误判，第二轮仅含单节课程缺陷时 ci 却通过，补上边界测试后才拦住它。从第一轮 PR 创建的 15:08:46 到首次失败反馈的 15:08:59 用了 13 秒，到修复后通过的 15:10:59 共 2 分 13 秒；若以首次失败到再次通过为额外延迟口径，则是 2 分钟。其中失败后人工修改至修复提交的 15:10:38 约 99 秒，提交到新运行创建的 15:10:43 约 5 秒，新运行给出通过结果再用 16 秒；首次失败前另有 13 秒反馈等待。这五次 ci job 平均 13 秒，单次约占上述三个提交到发布间隔的 0.28%、1.24%、0.79%，但这个比例不含排队、人工审批和发布工作流。补测试只改善已知边界的检测，对未写入断言的行为、部署配置、兼容性及制品来源问题仍不能保证发现；这些数字仅代表本仓库当时的 runner 与小规模测试。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：[PR #4 的 ci #13 日志](https://github.com/KanaScale/ops-lab1-12/actions/runs/39225332486/job/117970107953) 打印 github.sha=6dfdbc6cf7ad57dcd865934539f90fbc8e06e08c、pull_request.head.sha=332e05fb1c22c982ad6b3754f44a9012691fbeb6；[PR #4](https://github.com/KanaScale/ops-lab1-12/pull/4) 合入后的 main 提交是 [7b2cf19](https://github.com/KanaScale/ops-lab1-12/commit/7b2cf19f6e8b04555439a702724ee1381263420e)。

这两个 SHA 不相同：前者是 GitHub 为 PR 合并结果生成的临时提交，后者是 PR 分支实际提交。检查 main 的提交历史，两者都不在其中，因为本组使用 squash 合并，GitHub 为最终改动创建了新的 7b2cf19 提交。PR 的 ci 因此验证的是待合并结果，却不能把该临时提交或分支头直接当作已经进入 main 的发布来源。待发布制品应以经过审批和主干检查的 main 提交或指向它的 tag 构建，才能让 BUILD_INFO、Release 和仓库历史对应起来。以上 SHA 关系基于本组的 pull_request 检查与 squash 合并方式；换用其他合并方式时，历史中的提交关系需重新核对。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | 固定 `VERSION=9.9.9`、`BUILD_NO=100`、`RUN_ID=123456`、`COMMIT=f03b6ea350257855fb5862eacaff5efcfcee81f0` | `3fd60c2a89934e66ac012549144a4862fc2462887f7dfe341c6f23a22b27a39d` | `564c6a59cce16533f452a2a9ad5607b35f076cf9b290717bdcbcc690ceeece8b` |
| ② | 与①完全相同，再构建一次 | `3fd60c2a89934e66ac012549144a4862fc2462887f7dfe341c6f23a22b27a39d` | `564c6a59cce16533f452a2a9ad5607b35f076cf9b290717bdcbcc690ceeece8b` |
| ③ | 仅将 `BUILD_NO` 改为 `101` | `4bf128a3c5561e24d9c7731635823914d241a6e85b22f1aa807ae3251382463b` | `d248ffeef3ec9dc2c5517ca9328a2517c68154c02cb1a12b0ef241278dd15531` |
| ④ | 临时去掉 Makefile 中的 `--mtime=@0`，其余同① | `934c1b096ced6d32947f3cb45d0c12be65a966406668b24988571ce16ac6dd81` | `a32ca6e0073a364b779122b6c3cc9bd2ddbc6e96408552d950fa30c1c28a3719` |
| ⑤ | 同④，间隔数秒再次构建 | `934c1b096ced6d32947f3cb45d0c12be65a966406668b24988571ce16ac6dd81` | `1b00d7df348fc47d6c3bf0b328f0c4cb3d152f0792ecaf41394214aa6b6216e0` |
| Actions | [main 的 ci 运行 39227100247](https://github.com/KanaScale/ops-lab1-12/actions/runs/39227100247)，比较首次与 Re-run all jobs | 两次均为 `48a85fa25c0994e7661cbad9d419350b5f0fa2dcd1f9e40e28db49664e9bbb5e` | 两次均为 `18f0e8c62030fa1a8d69b107a8893061888d096c824093815475b8ff6c51ca51` |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

所用数据：组员 fireDragon198 在普通仓库检出中记录上述①～⑤：main 提交 `f03b6ea350257855fb5862eacaff5efcfcee81f0`，WSL `linux/amd64`，Go `1.26.9`，①②使用表内固定参数，④⑤只临时修改 Makefile 且事后恢复。`go version -m` 中①为 `vcs=git`、`vcs.revision=f03b6ea350257855fb5862eacaff5efcfcee81f0`、`vcs.time=2026-10-19T09:38:36Z`、`vcs.modified=false`；④的 revision 和 time 相同，但 `vcs.modified=true`。Actions 是同一 main push 运行的两次 attempt，`run_number` 和 `run_id` 相同，仅 `run_attempt` 变化。原始记录未保存 `GOFLAGS`、本地 tag 列表及 tar/gzip 等工具版本，故表中哈希是本组当时实测值，不保证换一个检出环境后逐字节相同。

复核时，在相同提交、构建参数和 Go 1.26.9 的普通克隆中，暂时隐藏后来创建的 `v1.0.0` tag 后，①与④的二进制哈希分别复现为 `3fd60c2a…a39d` 和 `934c1b09…dd81`；该 tag 可见时，Go 写入的模块版本信息不同，哈希也会变化。先前独立 `git worktree` 的 `.git` 为指向主仓库的文件，在本机 Go 1.26.9 下未写入 `vcs.*`，因此其④二进制没有变化；本表统一采用组员普通检出的原始记录。压缩包哈希在本次跨检出复核中未逐字节复现，表中保留组员原始测量值。

①②的二进制和归档哈希均相同，说明在组员当时固定的构建输入下能够重复；③仅改变构建号，链接参数和 `BUILD_INFO` 随之改变，所以两个哈希都变化。④的二进制与①不同，正如任务 0 所见，未提交的 Makefile 修改使 Go 自动记录的 `vcs.modified` 从 `false` 变为 `true`；取消固定归档时间又使 tar.gz 改变。⑤与④的二进制相同，但归档哈希不同，说明不固定文件修改时间会破坏压缩包的字节重复性。Actions 重跑两次的哈希均相同，因为该运行的提交、构建号和运行号不变，而 attempt 未写入制品。制品应记录版本、提交、构建号、运行号和工具链等追溯信息，并避免无控制的构建时间、绝对路径及文件元数据；复现时还须控制 Go 构建选项、Git 状态与 tag 可见性。`release` 重新编译是为了将正式 tag 版本号写入二进制和 `BUILD_INFO`，而 main CI 制品使用 `0.0.0-main`。因此本实验没有严格实现“一次构建、多处部署”，代价是额外构建时间以及两次环境或产物不一致的风险；以上结论只在相应源码、工具链、依赖、构建参数和打包环境可比时成立。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

所用数据：第 4 节两次 `/version`、两个版本的 `BUILD_INFO`、两次退出码为 0 的追溯输出，以及 v1.0.0 发布前后均为 `ad67bf9b7ee40853a874b1e8ebedb7c676ee0e4ba8c70d49bef426d39c577870` 的包哈希；[v1.0.0 Release API](https://api.github.com/repos/KanaScale/ops-lab1-12/releases/tags/v1.0.0) 与 [v1.0.1 Release API](https://api.github.com/repos/KanaScale/ops-lab1-12/releases/tags/v1.0.1) 在本次采集时均返回 `immutable: false`。

如果 v1.0.1 发布后发现缺陷，应先部署已验证的 v1.0.0 恢复服务，再通过新分支、PR、CI 和审批修复，合入后发布 v1.0.2 等新的修订版本。不能删除 v1.0.1 的 tag 并复用版本号，否则先后下载者可能拿到不同文件，版本、提交和构建的对应关系失去唯一性，应保留原记录并在发布说明中注明问题。本次直接运行 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 解包得到的 `enroll-conflict_1.0.0_linux_amd64/enroll-conflict`，没有重新构建，运行输出从 `1.0.1 / 3fccca8… / build_no=3` 变为 `1.0.0 / f03b6ea… / build_no=1`，且都与各自 `BUILD_INFO` 一致。v1.0.0 发布前后重新下载的包哈希相同，支持本次回退复用了原附件的字节内容。从源码重建同一个 tag 不能直接保证与原附件相同，只有源码及工作区状态、Go 工具链、目标平台、VERSION/BUILD_NO/RUN_ID/COMMIT 和归档参数等输入一致时才具备可重复条件，问题 3 中改变构建号或归档时间参数会改变相关输入，不能把重编译等同于取回原附件。如果有人同时替换包与其 `.sha256`，并保留指向 main 中真实提交的 `BUILD_INFO`，现有 `trace.sh` 仍可能通过，因为它没有验证包内字节与该提交的真实构建结果之间的可信绑定。独立保存且未被篡改的原始哈希能通过不一致发现替换，GitHub 的 Immutable releases 则能锁定已发布的附件和关联 tag 并生成发布证明，但设置只影响之后的新 Release，本次两个 Release 均未启用该保护（[官方用途](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases)、[适用范围](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/establish-provenance-and-integrity/prevent-release-changes)）。以上回退成立于原附件可信且可获取、主机兼容 linux/amd64、旧版本兼容当前配置及数据的条件下，本示例没有持久化数据迁移，真实服务若有数据库变更还需兼容迁移与恢复方案。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 已完成：[#9](https://github.com/KanaScale/ops-lab1-12/pull/9) 合入工作流；[v1.0.2 发布与镜像运行](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383152714) 均成功。摘要、架构和匿名拉取结果见下方。 |
| O2 缓存对反馈时间的影响 | 已完成：[缓存开启 PR #10 的 CI](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520) 与[缓存关闭、保持未合并的 PR #11](https://github.com/KanaScale/ops-lab1-12/pull/11) 各运行 6 次；逐次数据和分析见下方。 |
| O3 迁移到小组项目 | 未做 |

### O1 镜像发布与运行证据

- `v1.0.2` 指向 main 提交 [`bf9c5943322be6dccf543a47aba0f08cbb615f4e`](https://github.com/KanaScale/ops-lab1-12/commit/bf9c5943322be6dccf543a47aba0f08cbb615f4e)。[Release](https://github.com/KanaScale/ops-lab1-12/releases/tag/v1.0.2) 由同一 [release 工作流](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383152714) 创建；随后 [image job](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383152714/job/118496799001) 下载正式附件，`sha256sum -c` 输出 `enroll-conflict_1.0.2_linux_amd64.tar.gz: OK`，并核对附件内 `BUILD_INFO` 的版本 `1.0.2`、提交 `bf9c5943322be6dccf543a47aba0f08cbb615f4e`。构建上下文是解包后的二进制与 `BUILD_INFO`，Dockerfile 只执行 `COPY`，镜像 job 没有重新编译。
- GHCR 镜像 tag 为 `ghcr.io/kanascale/ops-lab1-12:v1.0.2`；本次推送产生的不可变摘要引用为 `ghcr.io/kanascale/ops-lab1-12@sha256:99a3084c81bcbbf77c102d4abb770cf9ce04d0f4bd81216de4328e403420e9f7`。实际运行的主机架构 `uname -m` 为 `x86_64`，拉取镜像后检查的平台为 `linux/amd64`。Actions 使用下列摘要引用运行容器，而非使用可变 tag：
```bash
docker run -d --rm -p 127.0.0.1:18080:8080 ghcr.io/kanascale/ops-lab1-12@sha256:99a3084c81bcbbf77c102d4abb770cf9ce04d0f4bd81216de4328e403420e9f7
```

- 容器的 `GET http://127.0.0.1:18080/version` 实测响应如下，与 v1.0.2 的提交、版本和 `BUILD_INFO` 一致。用空白 Docker 客户端配置对同一摘要执行匿名 `docker pull`，日志结果为 `Anonymous pull: yes`；因此该镜像在本次检查时无需登录即可拉取。

```json
{"version":"1.0.2","commit":"bf9c5943322be6dccf543a47aba0f08cbb615f4e","build_no":"4","go":"go1.26.9"}
```

发布 job 使用仓库的 `GITHUB_TOKEN` 创建 Release；按 [GitHub 的工作流触发规则](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow)，该令牌引起的普通 Release 事件不会再启动另一个工作流，所以镜像 job 使用 `needs: release` 接在同一个 `release.yml` 中。Dockerfile 的 `org.opencontainers.image.source` 标签用于关联仓库；GHCR 包的实际匿名可拉取性以上述独立客户端检查为准，而不只凭仓库是否公开推断。

### O2 缓存对 CI 反馈时间的影响

**预期与方法。** 本示例没有第三方 Go 依赖，模块下载缓存预计收益很小；但 Go 构建缓存仍可能减少标准库编译、`go vet` 和 `go test` 的时间。以 `actions/setup-go@v7` 默认缓存为实验组，以仅增加 `cache: false` 的 [PR #11](https://github.com/KanaScale/ops-lab1-12/pull/11) 为对照组。按照指导书，**PR #11 不合并**。为避免 main push 特有的打包和上传步骤混入比较，缓存组采用此前的 [PR #10 的 CI](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520)，无缓存组采用 [PR #11 的 CI](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232)；两者都是 `pull_request` 事件，运行同名 `ci` job、Ubuntu 24.04 和 Go 1.26.9。两组业务代码和测试相同；两次 PR 的文档和缓存配置提交不同。两组分别执行初次运行及 5 次 `Re-run all jobs`，下表列出全部 12 次成功运行；主要比较只用两组各 5 次重跑（attempt 2～6），避免混入首次触发与重跑之间的差异。

“反馈时间”取 GitHub Actions 每次 attempt 的 `updated_at - run_started_at`，即工作流开始至完成的秒数；另列 `ci` job 的 `completed_at - started_at`，以及 `setup-go` 步骤的时间。时间戳精度为秒，因此短步骤显示 0 秒不代表没有执行。表中每个 attempt 都链接到对应 job，可查看日志；缓存组 2～6 的日志均有 `Cache hit for` 和 `Cache restored successfully`，无缓存组 2～6 的日志均显示 `cache: false` 且无缓存命中。

| 组别 | attempt / job 证据 | 反馈时间（秒） | job 时间（秒） | setup-go（秒） |
|---|---|---:|---:|---:|
| 有缓存，初次运行 | [1](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118498663820) | 17 | 14 | 2 |
| 有缓存 | [2](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118528835135) | 18 | 14 | 3 |
| 有缓存 | [3](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118529608369) | 14 | 10 | 2 |
| 有缓存 | [4](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118529734882) | 14 | 11 | 1 |
| 有缓存 | [5](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118529860243) | 28 | 24 | 9 |
| 有缓存 | [6](https://github.com/KanaScale/ops-lab1-12/actions/runs/39383771520/job/118530795374) | 14 | 10 | 2 |
| 无缓存，初次运行 | [1](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118503003810) | 30 | 27 | 2 |
| 无缓存 | [2](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118524287529) | 44 | 37 | 3 |
| 无缓存 | [3](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118524600583) | 33 | 29 | 1 |
| 无缓存 | [4](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118524851065) | 33 | 29 | 0 |
| 无缓存 | [5](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118525085419) | 30 | 27 | 1 |
| 无缓存 | [6](https://github.com/KanaScale/ops-lab1-12/actions/runs/39385120232/job/118530926622) | 31 | 26 | 1 |

**结果与局限。** attempt 2～6 的反馈时间：有缓存为 18、14、14、28、14 秒（均值 17.6、中位数 14、范围 14～28），无缓存为 44、33、33、30、31 秒（均值 34.2、中位数 33、范围 30～44）；本组样本的均值差为 16.6 秒。对应 job 均值为 13.8 与 29.6 秒。`setup-go` 本身在有缓存组反而平均耗时 3.4 秒，无缓存组为 1.2 秒；缓存收益主要体现在后续 `Vet`（平均 3.2 对 19.2 秒）与 `Test`（平均 0.8 对 3.2 秒）。这与“无第三方依赖仍可能受益于 Go 构建缓存”的预期一致，但不能把 16.6 秒当作稳定的加速承诺：各组只有 5 次重跑，执行时间不交错、提交并非逐字相同，共享 runner 负载和缓存服务延迟也会波动；有缓存组第 5 次的 `setup-go` 就花了 9 秒。计时还未包含 PR 创建到工作流启动之前的等待时间，不能代表真实团队从提交到获得反馈的完整周期。

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| KanaScale | [#1](https://github.com/KanaScale/ops-lab1-12/pull/1)、[#2](https://github.com/KanaScale/ops-lab1-12/pull/2)、[#3](https://github.com/KanaScale/ops-lab1-12/pull/3)、[#9](https://github.com/KanaScale/ops-lab1-12/pull/9) | [#5](https://github.com/KanaScale/ops-lab1-12/pull/5)、[#7](https://github.com/KanaScale/ops-lab1-12/pull/7) |
| CynicDora | [#6](https://github.com/KanaScale/ops-lab1-12/pull/6)、[#7](https://github.com/KanaScale/ops-lab1-12/pull/7) | [#2](https://github.com/KanaScale/ops-lab1-12/pull/2)、[#3](https://github.com/KanaScale/ops-lab1-12/pull/3) |
| fireDragon198 | [#4](https://github.com/KanaScale/ops-lab1-12/pull/4) | [#6](https://github.com/KanaScale/ops-lab1-12/pull/6) |
| Har0v0i86 | [#5](https://github.com/KanaScale/ops-lab1-12/pull/5) | [#1](https://github.com/KanaScale/ops-lab1-12/pull/1)、[#4](https://github.com/KanaScale/ops-lab1-12/pull/4)、[#9](https://github.com/KanaScale/ops-lab1-12/pull/9) |
