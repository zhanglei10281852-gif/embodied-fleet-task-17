# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

后台引擎处理完过期机器人租约后，TasksPreempted 仍为 0，任务也没有离开原先的 claimed 状态。请修复回收结果在调度链中的传递，让成功回收项反映到统计和任务状态。相关生产实现之外不改，测试用例必须保留原状，不能跳过回收验证或放宽状态检查。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/embodied-fleet-task-17
- 仓库地址：https://github.com/zhanglei10281852-gif/embodied-fleet-task-17.git
- parent SHA：bc3f129f74a704dac1340df463cc6f4aa633be06

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/embodied-fleet-task-17.git bug-repro
cd bug-repro
git checkout --detach bc3f129f74a704dac1340df463cc6f4aa633be06
go test ./internal/engine -run ^TestEngine_PreemptExpiredClaim$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/engine -run ^TestEngine_PreemptExpiredClaim$ -count=1
--- FAIL: TestEngine_PreemptExpiredClaim (0.21s)
    engine_test.go:141: expected at least 1 preemption
FAIL
FAIL	github.com/zhanglei10281852-gif/embodied-fleet-go/internal/engine	0.215s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/engine -run ^TestEngine_PreemptExpiredClaim$ -count=1
--- FAIL: TestEngine_PreemptExpiredClaim (0.47s)
    engine_test.go:141: expected at least 1 preemption
FAIL
FAIL	github.com/zhanglei10281852-gif/embodied-fleet-go/internal/engine	0.677s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

引擎处理过期领取后，成功回收项必须传递到 TasksPreempted，且对应任务不能继续停留在 claimed，应进入既定的 assigned 或 preempted 状态。定向引擎测试须从修复前失败变为修复后通过，engine 包及全量回归通过，不改回收测试或放宽统计和状态断言。
