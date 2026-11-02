# UI 和 Agent 重构交接

> 更新日期：2026-11-02
> 工作分支：`codex/ui-agent-refactor`
> 业务代码基线：`master`，提交 `ceef938a4ab8d50f864f690fa7d768b4294fcf77`

本轮先整理分支、历史资料和重构边界。新的 UI 设计与 Agent 技术方案尚未确定，业务代码、依赖、数据库迁移和测试保持基线版本。

## 分支职责

| 分支 | 职责 | 使用方式 |
| --- | --- | --- |
| `master` | 已有产品基线，包含 7 月界面重构 | 保留，作为行为比较与回退依据 |
| `codex/ui-agent-refactor` | 重构准备与后续开发 | 从此分支继续；当前仅有文档整理 |

旧 `codex/ui-redesign-prep` 的成果已通过 `6a400744a440da27cf8ce88f9d8a66b421efd629` 于 2026-07-20 合入主分支，不需要重建旧分支。

2026-10-04 的 `4e7bf2225d900dd4227a107efebf350ef8ce2e89` 只修改三份准备文档，没有推送新 UI 或 Agent 实现。原机器的未提交文件、未推送分支和 worktree 不会随克隆恢复；取回原机器后，应先检查这些位置，再决定是否接续其中的工作。

## 依赖分支清理记录

以下 13 个分支只修改 `package.json` 或 `package-lock.json`，没有独有业务代码。对应 PR 关闭且不合并，分支删除前已保存完整 Git bundle。依赖升级仍是待办，清理分支不代表修复漏洞或完成兼容性验证。

分支均以 `dependabot/npm_and_yarn/` 开头。

| 分支后缀 | PR | 原提交 | 后续处理 |
| --- | --- | --- | --- |
| `body-parser-2.4.0` | [43](https://github.com/ziweiknows/ziwei-chat/pull/43) | `b440ee5529c7c5b416aee20283edef7d350dedad` | 随上层依赖评估 |
| `fast-uri-3.1.9` | [39](https://github.com/ziweiknows/ziwei-chat/pull/39) | `16986e1c0edcd4c5555e36dbe8be87511503e561` | 随上层依赖评估 |
| `hono-4.13.13` | [31](https://github.com/ziweiknows/ziwei-chat/pull/31) | `270ccb882689d96f31b02cf226c40f91d5c56099` | 随上层依赖评估 |
| `ip-address-10.7.5` | [42](https://github.com/ziweiknows/ziwei-chat/pull/42) | `1b31d7b393bc68f0fee0491efbb19eceb2761f8c` | 随上层依赖评估 |
| `modelcontextprotocol/sdk-1.32.1` | [32](https://github.com/ziweiknows/ziwei-chat/pull/32) | `bd2aa7cd16fcb05cb9a62156bb54439ca9609d86` | 先确认上层工具链需求，不因包名直接引入 MCP 架构 |
| `multi-05c505ab13` | [28](https://github.com/ziweiknows/ziwei-chat/pull/28) | `9224487831e603627ed977ffa31b17fd8058fb10` | 处理 `brace-expansion` 依赖链 |
| `multi-456d42d8d4` | [38](https://github.com/ziweiknows/ziwei-chat/pull/38) | `752e8cf82f06e74ebada13edec019bf980ed85d9` | Vitest 与 `@vitest/mocker` 一起验证 |
| `next-16.4.1` | [41](https://github.com/ziweiknows/ziwei-chat/pull/41) | `e3f80e0b9dd597712b011d0346acc46b7973e6d7` | 独立验证 Next.js、React 和 ESLint 配套兼容性 |
| `postcss-selector-parser-7.1.6` | [36](https://github.com/ziweiknows/ziwei-chat/pull/36) | `b1f564ff7db761b5eb3968541cc265903df9bab0` | 随上层依赖评估 |
| `proxy-addr-2.0.8` | [34](https://github.com/ziweiknows/ziwei-chat/pull/34) | `de5cab9ea7145644625663dd41340edbc0e117be` | 随上层依赖评估 |
| `source-map-js-1.2.2` | [35](https://github.com/ziweiknows/ziwei-chat/pull/35) | `040dd2778e5cebca09684bd394fd352b3ff1be43` | 随上层依赖评估 |
| `undici-7.30.1` | [40](https://github.com/ziweiknows/ziwei-chat/pull/40) | `a8f7814f4aa0d75097a9fb91be20d0b97cf89e8a` | 随上层依赖评估 |
| `vite-8.3.5` | [37](https://github.com/ziweiknows/ziwei-chat/pull/37) | `260f88111bbfc257a0b595e1e4f32a0fe9d1ce24` | 与 Vitest、esbuild 锁文件一起验证 |

后续按实际审计和兼容性选择版本，不必机械采用这些分支名里的版本。部分分支除升级目标包外还删除了 esbuild 锁文件条目；例如 `fast-uri-3.1.9`，不能仅根据升级标题认定变更可直接合并。

本次没有停用 Dependabot；GitHub 仍可能创建新的更新 PR。关闭这批 PR 后，相关版本升级需要在独立依赖维护工作中主动跟进。

2026-11-02 的 GitHub 推送提示默认分支存在 126 条依赖漏洞告警（11 critical、48 high、58 moderate、9 low）。这是平台告警计数，不等于受影响包数或已确认可利用的问题；后续依赖维护需重新审计，不能沿用 10 月的旧审计数字。

## 备份与恢复

当前机器的备份位于 `D:/Code/new_CHAT/.git/refactor-cleanup-20261102/`：

- `before-cleanup.bundle`：清理前的完整 Git 历史与分支引用，已通过 `git bundle verify`。
- `branch-manifest.json`：分支名、完整提交、PR 编号及变更文件清单。
- `open-pulls.json`：清理前的开放 PR 快照。

Bundle 的 SHA-256：`AB226A83B59EA14A35E2B22678DBDB61365C88EEFB409805A454F35A63128AFF`。

备份文件仅在当前机器，不随 Git 推送。搬迁或删除本地仓库前须另存。2026-10-04 记录的 `.git/refactor-cleanup-20261004/` 属于原机器，当前克隆没有该备份。

例如从当前备份恢复 `fast-uri` 分支到本地，以便检查：

```powershell
git bundle verify .git/refactor-cleanup-20261102/before-cleanup.bundle
git fetch .git/refactor-cleanup-20261102/before-cleanup.bundle refs/remotes/origin/dependabot/npm_and_yarn/fast-uri-3.1.9:refs/heads/recovered/fast-uri-3.1.9
```

若确认需要重新发布该分支，再执行：

```powershell
git push origin refs/heads/recovered/fast-uri-3.1.9:refs/heads/dependabot/npm_and_yarn/fast-uri-3.1.9
```

恢复分支不会自动重开 PR。其他分支的原引用可由上表和备份清单确定。

## 重构资产与边界

| 资产 | 位置 | 重构处理 |
| --- | --- | --- |
| 排盘与事实提取 | `src/lib/chart/`、`src/lib/domain/` | 优先复用，保留 iztro 的确定性计算边界 |
| 六类分析流程与知识 | `content/skills/`、`content/knowledge/` | 保留内容、来源和许可证；导入材料与精选知识继续分层 |
| 持久化与数据删除 | `src/lib/db/`、`drizzle/` | 保留数据兼容性、匿名归属与删除后防止旧请求重新写入的保护 |
| 回答检查与证据 | `src/lib/agent/critic.ts`、`evidence-events.ts` | 保留事实和可追溯性约束，具体接入随新运行时调整 |
| 浏览器业务状态 | `src/lib/ui/` | 按调用关系复用命盘恢复、会话、重试、隔离与删除逻辑 |
| 行为回归与评估 | `tests/`、`src/lib/evaluation/` | 保留领域行为断言，更新与实现结构绑定的断言 |
| 现有视觉与动画 | `src/components/`、`src/app/globals.css` | 等新设计明确后逐项替换，不能整体删除业务组件 |
| 旧视觉与迁移测试 | `tests/ui/reference-visual-contract.test.ts`、`migration-cleanup.test.ts` | 新设计落地时更新；保留其中真实业务约束 |
| 独立图片接口诊断 | `scripts/test-image-provider.mjs` | 非聊天主流程必需；待诊断需求确认后再决定移除 |

本轮没有删除业务文件、测试或知识内容。单纯缺少直接调用不是删除依据，还需核对类型引用、动态加载及测试用途。

## 文档阅读顺序

1. 本交接与 [Project Status](project-status.md)：当前入口、分支职责与历史界限。
2. [PRD](../product/prd.md)、[回应协议](../prompts/response-protocol.md)：产品边界与人格。
3. [Agent 架构](../architecture/agent-architecture.md)、[工具契约](../architecture/tool-contracts.md)、[数据模型](../architecture/data-model.md)：现有设计；新方案确定后同步修订。
4. [知识规范](../knowledge/knowledge-and-skills-spec.md)、[内容缺口](agent-content-gaps.md)：保留的领域资产与扩展空间。
5. [验收标准](../evaluation/acceptance-criteria.md)、[验收缺口](final-v1-plus-gap-register.md)、[真实模型验收](real-provider-acceptance.md)：继承未完成的验证事项。

[旧实施计划](implementation-plan.md) 和 Project Status 中的 July 2026 记录用于历史追溯，不作为新一轮任务顺序。旧测试数字及依赖审计数字均不代表本轮重新验证结果。

## 后续工作顺序

1. **恢复开发基线。** 使用 CI 对齐的 Node.js 22，安装依赖并运行 lint、typecheck、tests、Agent evaluation 和 build；独立验证 Postgres 和真实模型场景，记录跳过项。
2. **处理依赖维护。** 重新审计依赖链，分别处理框架、测试工具和其他依赖；修复锁文件后验证干净安装，不与业务重构混为一次变更。
3. **确定新方案。** 明确 UI 页面和交互、Agent 运行时、模型接入、工具、检索、记忆及持久化边界。当前依赖声明包含 Vercel AI SDK，但实际模型调用使用自写 HTTP/流适配器；需统一实现与文档。
4. **拆清编排职责。** `src/app/api/chat/route.ts` 在业务基线中有 1,404 行，应将编排从 HTTP 路由中分离，同时保持接口与错误行为兼容。
5. **逐步替换 UI 和 Agent。** 保留确定性命盘、知识来源、数据归属、可删除性及证据机制。记忆/摘要工具目前不能视为已接通的持久主聊天记忆功能。
6. **完成验收。** 继承真实模型 Insights、键盘与无障碍、缓存失效、完整删除及视觉验收场景；通过相应检查后再合回主分支。

本轮验证范围是分支、PR、备份及文档一致性；未安装依赖或运行应用测试。
