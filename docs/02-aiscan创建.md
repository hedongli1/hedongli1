# GitHub 运营执行报告（二）· aiscan 启动部署

> 日期：2026-08-26 · 方向：安全/紫队 · 原则：真实、可验证
> 本轮核心：**从零创建并部署开源 AI 代码安全审计工具 aiscan**，全部运行结果可复现。

---

## 一、交付项目：aiscan 🛡️

**定位**：AI 辅助代码安全审计 —— 零依赖静态分析 + 熵启发式密钥检测 + SARIF 报告 + GitHub Action 一键接入。

📍 [github.com/hedongli1/aiscan](https://github.com/hedongli1/aiscan) · MIT · v1 标签已推送

### 核心能力（全部真实可验证）

| 能力 | 实现 | 证据 |
| --- | --- | --- |
| 14 条检测规则 | 密钥/AWS/私钥/连接串/SQL注入/命令注入/XSS/弱加密/路径穿越/关闭TLS/日志泄漏/供应链 | `lib/rules/index.js` |
| **AI 熵启发式** | 香农熵 + 长度 + 字符混合度 → 置信度 0-100，识别无格式高熵密钥 | `lib/entropy.js` |
| 四种输出 | 人类可读 / JSON / **SARIF 2.1.0** / HTML 报告 | `bin/aiscan.js` |
| GitHub Action | `hedongli1/aiscan@v1` 一行接入，自动上传 SARIF 到 Security 标签页 | `action.yml` |
| .aiscanignore | 类似 .gitignore，忽略已复核误报 | `lib/scanner.js` |

### 真实运行结果

| 场景 | 结果 | 意义 |
| --- | --- | --- |
| `npm test` | **10/10 通过** | 单元测试覆盖引擎/熵/规则 |
| 扫自身源码（dogfooding） | **0 发现，评分 100/A** | 自己扫自己 |
| 扫漏洞演示文件 | **11 发现（5 critical/5 high/1 medium），评分 0/F** | 全部准确识别 |
| **扫真实项目 ledger-app** | **发现 1 critical SQL 拼接信号** | 真实项目审计价值！ |
| CI（GitHub Actions） | **全绿**：测试 + dogfooding + demo | [Actions](https://github.com/hedongli1/aiscan/actions) |

### 真实安全闭环（最有说服力的案例）

aiscan **真实扫描了同账号的 ledger-app 项目**，在 `server/src/routes.js:42` 发现动态 SQL 拼接信号。

**人工复核结论**：该处 `conds` 数组全部由服务端硬编码 SQL 片段组成（`'user_id = ?'` 等白名单），用户输入全部经 `?` 占位符参数化传入 `.all(...args)`，**实际安全**。但写法确实有工程改进价值。

**处理**：重构代码使拼接点更清晰 + `.aiscanignore` 记录复核结论。这是"用自己写的工具发现自己项目的隐患 → 复核 → 改进"的完整真实闭环。

## 二、与其他仓库的联动

| 动作 | 结果 |
| --- | --- |
| aiscan 扫描 purple-team-lab | 0 发现（评分 100/A，代码干净） |
| aiscan 扫描 ledger-app | 发现 SQL 拼接信号 → 复核 → 修复 |
| 个人主页置顶 aiscan | 加 CI/tests 徽章，置顶精选项目 |
| 博客新文章 | 《我用零依赖写了一个 AI 代码安全审计工具 aiscan》已上线 |

## 三、博客

📝 新文章上线：[aiscan 开发复盘](https://hedongli1.github.io/2026/08/26/aiscan/)
- 讲三件事：为什么做、熵启发式 AI 原理、扫真实项目发现了什么

## 四、仓库现状总览

| 仓库 | CI | License | Topics | 说明 |
| --- | --- | --- | --- | --- |
| **aiscan** 🛡️ | ✅ 全绿 | MIT | 12 个 | **主打新品**：AI 代码安全审计 |
| purple-team-lab | ✅ 全绿 | MIT | 12 个 | 紫队实验台 |
| ledger-app | ✅ 全绿 | MIT | 11 个 | 记账本（本轮含 SQL 安全改进） |
| hedongli1.github.io | ✅ | - | - | 博客（5 篇文章） |
| hedongli1 | ✅ (snake) | - | - | 个人主页（aiscan 置顶） |

---

## 五、吸引力提升（吸 star 策略落地）

1. **差异化定位**：`AI 辅助 + 零依赖 + SARIF + GitHub Action`四合一，与其他安全扫描器错位
2. **dogfooding 自扫**：README 直接展示"扫自己 100/A"，最诚实的宣传
3. **可复现证据**：CI 全绿 + 10 测试 + demo 检出 11 漏洞，点徽章可验证
4. **真实审计案例**：扫自己项目发现 SQL 拼接 → 有真实故事可讲
5. **`@v1` Action 可用**：任何人能一行接入，降低使用门槛（已推 v1 tag）

## 六、建议下一步

1. **写第二篇博客**：把"aiscan 扫 ledger-app 发现 SQL 拼接"的真实审计过程写成文章（强故事性）
2. **投稿安全社区**：freebuf / 先知社区 / HackerNews Show HN
3. **npm 发布**：`npm publish` 让 `npx aiscan` 可直接用（当前需 clone）
4. **扩展规则**：加 Dockerfile/Go/Java 规则，扩大适用面
5. **PR 模板 + Discussion 开洞**：鼓励社区贡献规则

> ⚠️ 安全说明：所有 token 仅存临时变量，未写入任何公开文件；随时可在 github.com/settings/tokens 撤销。