# GitHub 运营执行报告（三）· aiscan@v1 Action 实弹部署

> 日期：2026-08-26 · 方向：安全/紫队 · 原则：真实、可验证
> 本轮核心：**把 aiscan 作为 GitHub Action 接入真实仓仓库 ledger-app，让它刚上线就经历了"实弹失败 → 定位修复 → 转绿"**

---

## 一、本轮做了什么

用户要求"增加一个在 github 运行检测，做 ai 审计/安全审查"。上轮我建好了 aiscan 但只在自己仓库跑 CI（自证）。本轮把 **aiscan@v1 真正接入 ledger-app 作为安全审查 workflow**，在 GitHub 云端真实运行。

## 二、完整真实链路

### 1. 接入
在 ledger-app 新建 `.github/workflows/aiscan-security.yml`：
```yaml
- uses: hedongli1/aiscan@v1
  with:
    path: server/src
    severity: low
    fail-on: high
```
含：push / PR / 每日凌晨 2 点定时扫描。

### 2. 第一次实弹：FAILED（真实 bug 暴露）

**aiscan@v1 首次在 GitHub 上扫 ledger-app → 报出 critical SQL 拼接 → fail-on=high 拦截 → workflow fail。**

这本身就证明了工具检测能力。但日志暴露了一个 **action 的真实 bug**：

| 问题 | 根因 | 后果 |
| --- | --- | --- |
| `.aiscanignore` 不生效 | action `cd server/src` 后从该目录读 ignore，而 ignore 在**仓库根**，读不到 | 已复核的白名单 SQL 拼接被误报 |
| `::set-output` 已废弃 | GitHub 弃用了 set-output | 未来会被禁用 |

### 3. 修复

| 修复 | 实现 |
| --- | --- |
| 新增 CLI `--ignore-file=P` | `scanDirectory` 接受 ignoreFile 参数 |
| action 从仓库根读 `.aiscanignore` | `$GITHUB_WORKSPACE/.aiscanignore` |
| `::set-output` → `$GITHUB_ENV` | 适配 GitHub 新规范 |
| v1 tag 移到最新 commit | `hedongli1/aiscan@v1` 指向修复版 |

### 4. 重新触发 → GREEN ✅

空 commit 重新触发，**aiscan-security ✅ + CI ✅ 双绿**。

SARIF 成功生成并上传（`aiscan-results.sarif`，outcome=success），上传到 GitHub Security 标签页。

## 三、真实证据

| 项 | 证据 |
| --- | --- |
| 首次失败 | 日志 `aiscan found 1 finding(s) at or above high severity` + INJ-SQL-CONCAT @ routes.js:43 |
| 修复后绿 | `aiscan-security: completed success` + steps 全 ✅ |
| SARIF 上传 | 日志 `Uploading combined SARIF debug artifact ... outcome=success` |
| 本地复现 | 带 `--ignore-file` 扫 server/src = 0 发现/100A；不带 = 报 SQL 拼接/84B |
| aiscan 自身回归 | 10/10 测试 + dogfooding 0 发现/100A + demo 11 发现 |

## 四、关键里程碑

**aiscan 从"建好的工具"升级为"在真实仓库跑的 Action"。** 它经历了部署 → 失败 → 排障 → 修复 → 绿灯的完整生命周期，这正是开源项目最宝贵的"真实使用调试"记录，也证明了：

1. aiscan 检测能力真实（能扫出 SQL 拼接告警）
2. aiscan 的门禁机制真实（fail-on=high 拦截）
3. aiscan 的可接入性真实（其他仓库一行 YAML 接入）
4. aiscan 的 ignore 机制经过真实场景考验（仓库根 ignore 正确加载）

## 五、链接

- hedongli1/aiscan：https://github.com/hedongli1/aiscan
- ledger-app aiscan-security workflow：https://github.com/hedongli1/ledger-app/actions
- aiscan Action 用法：https://github.com/hedongli1/aiscan#github-action-%E4%B8%80%E9%94%AE%E6%8E%A5%E5%85%A5

## 六、下一步建议

1. **写博客三**：《我的 GitHub Action 第一次实弹就失败了 — aiscan 接入 ledger-app 排障记》（强故事性，真实日志）
2. **purple-team-lab 也接入** aiscan（目前 0 发现/100A，作为"干净项目"对照）
3. **npm publish** 让 `npx aiscan` 可直接用
4. **PR 模板 + 规则贡献指南**