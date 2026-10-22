# 实验 1 记录：第 13 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 13 组 |
| 仓库地址 | <https://github.com/SS80194/SE3309_2026_lab1> |
| 代码载体 | 统一示例（Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go 1.26.9 |
| 合并方式 | 规则集当前只允许 squash；PR #3～#5 使用 squash，PR #1 在规则修正前误用了 merge commit，见问题 2 |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | `name=enroll-conflict`<br>`version=1.0.0`<br>`build_no=1`<br>`run_id=local`<br>`commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`go=go1.26.9` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | `diff` 无输出，BUILD_INFO 没有变化；二进制 sha256 从 `f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e` 变为 `42f8c4d664826d3278de74fa2c8bb66fe52105d22040d77db27eac9e70755ae8`。 |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | `vcs=git`、`vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`、`vcs.time=2026-10-12T16:26:07Z` 均相同；第一次为 `vcs.modified=false`，未提交修改后为 `vcs.modified=true`。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 可以确认制品名、版本、构建号、Go 版本和构建时的 HEAD，但不能确认构建主机与构建时间。工作区含未提交改动时，`commit` 仍指向 HEAD，真实输入代码却不等于该提交，因此“用哪份代码构建”会出错；须结合 `vcs.modified` 或强制干净工作区判断。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | `protect-main`（Active，默认分支，Bypass list 为空）：Restrict deletions；Block force pushes；Require a pull request before merging（审批 1 人，只允许 squash）；Require status checks to pass（必需检查为 `ci`）。2026-10-21 20:11（北京时间）通过 GitHub API 复核。 |
| 组长直接 `git push origin main` 被拒的终端输出 | `remote: error: GH013: Repository rule violations found for refs/heads/main.`<br>`remote: - Changes must be made through a pull request.`<br>`! [remote rejected] HEAD -> main (push declined due to repository rule violations)` |
| 第一轮：失败的 PR 运行 | [ci #2（failure，19 秒）](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568009947) |
| 第一轮：修复后通过的 PR 运行 | [ci #3（success，17 秒）](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568759228)；PR 重新打开后另有 [ci #4（success，18 秒）](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39570065522) |
| 第一轮：合入 main 后的运行（含制品） | [ci #5（success，19 秒，含 artifact）](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39570884805) |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | 主动把 `Section.Valid` 的星期上界从 7 改为 6，使合法周日被拒绝；旧测试只验证周三合法和 0/8 非法，没有覆盖周日边界。 |
| 第二轮：仅含缺陷时仍然通过的运行 | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39574266980> |
| 第二轮：补测试后失败的运行 | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39574372699> |
| 第二轮：撤销缺陷后通过的运行 | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39574475189> |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39583519545> |
| Release v1.0.0 | <https://github.com/SS80194/SE3309_2026_lab1/releases/tag/v1.0.0> |
| Release v1.0.1 | <https://github.com/SS80194/SE3309_2026_lab1/releases/tag/v1.0.1> |

### 任务 4 第二轮：周日边界漏测演练

PR：<https://github.com/SS80194/SE3309_2026_lab1/pull/2>；分支：`lab1/gate-round2`。

这是主动注入并撤销的边界错误，不是声称原始 main 已有该错误。业务依据是 README 规定 `weekday` 合法值为 1～7，`conflict.go` 字段注释也明确 7 代表周日。

| 阶段 | 提交 | 检查结果 |
|---|---|---|
| 有错误，只有原测试 | [`84fe211`](https://github.com/SS80194/SE3309_2026_lab1/commit/84fe211e955903c83ea4800b97b8e9ec2e038cae) | `make fmt-check vet test` 通过，说明原测试漏掉合法周日 |
| 有错误，补充边界测试 | [`afdc3da`](https://github.com/SS80194/SE3309_2026_lab1/commit/afdc3dacc69d523369c31ce02614b3fb53e9ccf7) | 格式与 vet 通过；测试因周日被拒绝而失败 |
| 撤销错误，保留新测试 | [`9cd02da`](https://github.com/SS80194/SE3309_2026_lab1/commit/9cd02da15db6b38fcef186e597361fad21d46ffe) | `make fmt-check vet test` 全部通过 |

第二阶段的实际失败输出：

```text
--- FAIL: TestValidWeekdayBoundaries
    --- FAIL: TestValidWeekdayBoundaries/周日有效
        conflict_test.go:74: Section.Valid(weekday=7) = false, want true
FAIL
```

新增 `TestValidWeekdayBoundaries` 覆盖 -1、0、1、6、7、8，兼顾合法边界和非法输入。最终 `conflict.go` 与演练前 main 完全相同；保留新增测试与本节证据。失败来自测试发现行为错误，不是格式检查、编译或环境失败。

本地与 CI 均使用 Go 1.26.9；本地三阶段也分别执行相同检查。PR 通过另一位成员审批后，合入 main；是否已合并须以 PR 实际状态及 main 运行记录为准。

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 4 |
| 提交号（SHA） | 235f574fcbae2d91b566d20b182a6d4bd817489f | eef126cd1e3baa7b98db46e6f5db908290684a89 |
| 引入该提交的 PR | #3 <https://github.com/SS80194/SE3309_2026_lab1/pull/3> | #4 <https://github.com/SS80194/SE3309_2026_lab1/pull/4> |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39583364244> | <https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39584461409> |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | 027877CDE04D7B48DE15948DD8AE1F3B7523CE1CEFE30E6DBB205B7DA6536D00 | B8BB5AA56CF7533FDAC13B582EF02032F42CA6D72F2CC1BCBA7FB2D81118E5A4 |

仅凭构建号不能唯一确定运行，因为 `ci` 和 `release` 的 `github.run_number` 分别独立计数。v1.0.0 的构建号为 1：在 `ci` 中对应 [ci #1](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39560794400)，在 `release` 中对应 [release #1](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39583364244)。制品中的全局唯一 `run_id` 才能直接定位实际构建运行。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`027877cde04d7b48de15948dd8ae1f3b7523ce1cefe30e6dbb205b7da6536d00`。与上表中发布当时的值一致。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39583364244
commit=235f574fcbae2d91b566d20b182a6d4bd817489f
go=go1.26.9
== 3. 提交 235f574fcbae2d91b566d20b182a6d4bd817489f
作者: ShawneeLiu <3204035715@qq.com>
时间: 2026-10-21T18:15:59+08:00
标题: feat: add release workflow
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/SS80194/SE3309_2026_lab1/commit/235f574fcbae2d91b566d20b182a6d4bd817489f
构建它的运行: https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39583364244
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=4
run_id=39584461409
commit=eef126cd1e3baa7b98db46e6f5db908290684a89
go=go1.26.9
== 3. 提交 eef126cd1e3baa7b98db46e6f5db908290684a89
作者: ShawneeLiu <3204035715@qq.com>
时间: 2026-10-21T18:22:57+08:00
标题: docs: record v1.0.0 release info and release gate evidence
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 4
提交页面（含关联的 PR）: https://github.com/SS80194/SE3309_2026_lab1/commit/eef126cd1e3baa7b98db46e6f5db908290684a89
构建它的运行: https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39584461409
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
运行 v1.0.1：
{"version":"1.0.1","commit":"eef126cd1e3baa7b98db46e6f5db908290684a89","build_no":"4","go":"go1.26.9"}

换回 v1.0.0：
{"version":"1.0.0","commit":"235f574fcbae2d91b566d20b182a6d4bd817489f","build_no":"1","go":"go1.26.9"}
```

执行顺序说明：v1.0.1 在任务 6 的本组小改动 PR 之前已由发布负责人发布，因此它实际对应 PR #4 的报告更新提交。为保持已发布版本不可变，本组不删除或重用 v1.0.1；任务 6 小改动另走 PR，如需发布则使用后续版本号。

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-21 16:16:14 | [PR #1](https://github.com/SS80194/SE3309_2026_lab1/pull/1) |
| 第一次 `ci` 失败（运行结束） | 2026-10-21 16:16:38 | [run 39568009947](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568009947) |
| 推送修复提交 | 2026-10-21 16:22:15 | [commit 90874f8](https://github.com/SS80194/SE3309_2026_lab1/commit/90874f8406797e3b3ee1b77731269006101ca410) |
| `ci` 通过（运行结束） | 2026-10-21 16:22:38 | [run 39568759228](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568759228) |
| 合入 main | 2026-10-21 16:39:04 | [merge commit 5276085](https://github.com/SS80194/SE3309_2026_lab1/commit/527608575a3091fb2af3f4965506e9250f5057f4) |

以上均为北京时间（UTC+8）。

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#1](https://github.com/SS80194/SE3309_2026_lab1/pull/1) | 2026-10-21 16:12:55 | 2026-10-21 16:39:04 | 2026-10-21 18:16:39（v1.0.0） | 2 小时 3 分 44 秒 |
| [#3](https://github.com/SS80194/SE3309_2026_lab1/pull/3) | 2026-10-21 18:09:11 | 2026-10-21 18:15:59 | 2026-10-21 18:16:39（v1.0.0） | 7 分 28 秒 |
| [#4](https://github.com/SS80194/SE3309_2026_lab1/pull/4) | 2026-10-21 18:19:34 | 2026-10-21 18:22:58 | 2026-10-21 18:25:34（v1.0.1） | 6 分 00 秒 |

以上均为北京时间（UTC+8）。“提交到发布的间隔”从该 PR 第一个提交的时间算至对应 Release 的发布时间；#1 的改动随之后建立的首个 Release v1.0.0 一并交付，因此间隔明显长于发布配置 PR #3。

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| [#1](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39560794400) | push（main） | 46 秒 | 首次成功，产出 artifact |
| [#2](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568009947) | pull_request（#1） | 19 秒 | 故意引入错误，检查失败 |
| [#3](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39568759228) | pull_request（#1） | 17 秒 | 修复后通过 |
| [#4](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39570065522) | pull_request（#1 reopened） | 18 秒 | 重新打开 PR 后再次通过 |
| [#5](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39570884805) | push（main） | 19 秒 | 合入 main 后通过，产出 artifact |

这 5 次运行的中位数为 19 秒。

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

所用数据：第一轮失败运行 19 秒；失败结束于 16:16:38，修复提交于 16:22:15；CI 中位数 19 秒；PR #3、#4 从首次提交到发布分别为 7 分 28 秒和 6 分。第一轮中，错误使 CI 失败并阻止当时的合并，修复且由另一位组员审批后才继续；从失败开始计算，额外花费 5 分 56 秒，其中等待失败 CI 为 19 秒，定位和修改为 5 分 37 秒。一次 CI 约 19 秒，占上述两个较短发布间隔的约 4.2% 和 5.3%，因此本次实验中自动检查本身的等待成本较小。第二轮说明原有测试没有覆盖星期日边界，只有补上针对性测试后才暴露缺陷，因此 gate 只能拦住现有检查能够识别的问题。测试本身仍无法发现共同需求理解错误、未写断言的边界、生产环境配置、性能和安全等问题，还需要评审、监控及其他验证手段。上述“阻止合并”的结论成立条件是 main 规则确实把 `ci` 设为 required status check；2026-10-21 20:11（北京时间）复核时该条件已经满足。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：PR #1 通过运行中 `github.sha=c6bb30b6530400e66a32abd88d2cb03f7208d576`，`pull_request.head.sha=90874f8406797e3b3ee1b77731269006101ca410`，合入 main 后对应提交为 `527608575a3091fb2af3f4965506e9250f5057f4`。两者不相同：前者是 GitHub 为 PR 检查生成的临时合并提交，后者是分支头提交。PR #1 实际使用 merge commit 合并，所以 head SHA 能在 main 的祖先历史中找到，而临时 `github.sha` 不能；最终 main 上代表这次合并的是 `5276085...`。本组原定只使用 squash，后续 PR #3～#5 的确使用 squash，此时 PR 的 head SHA 也不会等于 main 上新生成的 squash 提交，例如 PR #3 的 head 为 `cc296cd...`、main 提交为 `235f574...`。这说明 PR 构建可能针对临时合并提交或之后会被改写的分支提交，不能作为最终发布血缘的稳定标识。待发布制品因此只能从已落在 main 的提交或指向它的不可变 tag 构建，才能让制品内 commit、仓库历史与 Release 相互追溯；这一结论的条件是 tag 指向经过 gate 的 main 提交且发布后不被移动。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，固定 `VERSION=9.9.9`、`BUILD_NO=100`、`RUN_ID=1000`、`COMMIT=527608575a3091fb2af3f4965506e9250f5057f4` | `13a8ab796df93072d821423697ae88bd59422d3f8e012247ea732f764f7f028d` | `b18af5dba521bf8773ff8d02cb8782d4694b96700795f16281bcf6b852596c71` |
| ② | 与 ① 完全相同，再构建一次 | `13a8ab796df93072d821423697ae88bd59422d3f8e012247ea732f764f7f028d` | `b18af5dba521bf8773ff8d02cb8782d4694b96700795f16281bcf6b852596c71` |
| ③ | 仅将 `BUILD_NO` 改为 `101` | `27a05d3fdc81a9e89f2ecb10b70962615ec72cc539c2241d4443932aca78289a` | `408b4b99bb671509e4da9a90dc143db1541349fb95d54b674f0656acc1217c1b` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `c7f236fa022189b3364ea1fce6656bea6fa0efea0576e7e46ff8b002f8350419` | `92a1cf0a5c19c39be9f8c60824866b999620d4d77fceb28ba79182dfff76b5c0` |
| ⑤ | 同 ④，间隔数秒后再构建一次 | `c7f236fa022189b3364ea1fce6656bea6fa0efea0576e7e46ff8b002f8350419` | `139200b0480dd7cb83726f30ee36fabc23c5122d8609064220875cc147a88933` |
| Actions | 对 main 的 [CI #5](https://github.com/SS80194/SE3309_2026_lab1/actions/runs/39570884805) 执行 Re-run all jobs | **待有 Actions 写权限的成员重跑后填写 attempt 1/2** | **待有 Actions 写权限的成员重跑后填写 attempt 1/2** |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

所用数据：本地构建 ①～⑤ 的哈希如表；任务 0 还观察到工作区修改会令 Go 构建信息从 `vcs.modified=false` 变为 `true`。①与②完全相同，说明固定输入并把 tar 内文件时间归零时结果可重复；③改变了写入程序的构建号，所以二进制和归档都改变。④、⑤的二进制相同而 tar 不同，因为去掉 `--mtime=@0` 后归档记录了不同文件时间；④的二进制又不同于①，是因为临时修改 Makefile 使 Go 嵌入了 `vcs.modified=true`。制品应包含版本、main/tag 提交号、构建号和运行号等可追溯的确定性信息，不应包含无必要的当前时间、主机路径或其他易变环境数据，并应在干净且固定的工具链环境中构建。`release` 在同一提交上重新编译，是为了注入正式版本号并生成 Release 附件，但它没有直接提升 CI 的既有主干制品，严格说与“一次构建、多处部署”有冲突，代价是重复计算时间以及两次环境或输入漂移造成不同二进制的风险。Actions 行的结论还以同一次 main 运行完成第二次 attempt 为条件；当前账号无重跑权限，不能把尚未取得的数据伪造进报告。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

所用数据：实际下载的 v1.0.0 文件 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 校验通过，回退前 `/version` 为 `{"version":"1.0.1","commit":"eef126cd1e3baa7b98db46e6f5db908290684a89","build_no":"4","go":"go1.26.9"}`，切换后为 `{"version":"1.0.0","commit":"235f574fcbae2d91b566d20b182a6d4bd817489f","build_no":"1","go":"go1.26.9"}`。v1.0.1 出现缺陷时，应保留现有 tag 和附件，先回退到已验证的 v1.0.0，再通过新 PR 修复并发布递增版本（例如 v1.0.2）。删除 v1.0.1 tag 后复用同一版本号会让不同用户拿到同名异物，破坏缓存、审计记录和哈希所代表的供应链身份。任务 6 的回退直接运行已保存的 v1.0.0 Release 归档，不需要重新构建；上述两次输出证明实际运行版本、提交和构建号一起恢复。若改为从源码重建，只有源码、工具链、环境、构建参数和归档元数据全部固定且工作区干净时才可能与原附件逐字节相同，问题 3 已证明易变元数据会改变结果。若攻击者把附件和同目录下的新 `.sha256` 一起替换，单独运行 `trace.sh` 仍可能校验通过，但会与第 4 节已经外部记录的 v1.0.0 哈希 `027877cde04d7b48de15948dd8ae1f3b7523ce1fe30e6dbb205b7da6536d00` 不符。GitHub Immutable releases 可进一步锁定发布的 tag 和附件，降低事后篡改风险；该结论以功能已由仓库管理员启用为条件。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 未做 |
| O2 缓存对反馈时间的影响 | 未做 |
| O3 迁移到小组项目 | 未做 |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| SS80194 | 尚无（需补一个本人发起并合入的 PR） | #1、#2、#3、#4、#5 |
| qeeenn | #1 | 无 |
| ARMANDSnow | #2（已审批，当前尚未合入） | 无 |
| ShawneeLiu | #3、#4、#5 | 无 |
| qk3256 | #6（待审批、合入） | 无 |
