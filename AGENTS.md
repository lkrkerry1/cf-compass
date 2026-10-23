# AGENTS.md

> 给在本仓库工作的 AI 编程代理（Codex / Claude / Copilot / 其他自动化代理）的作业说明。
> 人类贡献者请先读 [CONTRIBUTING.md](./CONTRIBUTING.md)；用户文档见 [README.md](./README.md)。

## 0. 一句话项目

CF Compass 是一个**本地优先**的 Codeforces / OI / ICPC 训练桌面应用：题库筛选、今日训练、间隔复习、赛事复盘、虚拟赛估分、算法模板和本地笔记，数据全部保存在用户本机。

## 1. 仓库现状（重要）

- 当前版本：**v4.3.5**。
- 本仓库是原项目 `Binah-Dev/cf-compass` 的社区延续版。原仓库及原作者 GitHub 账号已于 2026 年 10 月被删除，代码基于公开 fork 保留的 v4.3.4。
- 维护分支：`main`（稳定/发布）与 `dev`（集成）；远程：
  - `origin` → `lkrkerry1/cf-compass`（**唯一可写**，推送目标）
  - `upstream` → `lorgarbr/cf-compass`（历史来源，只读参考，**不要推送**）
  - `archive` → `Binah-Dev/cf-compass`（已失效，仅留档）
- 分支与合并流程：
  - `main`：只接受来自 `dev` 的合并和 tag 发布，不直接开发。
  - `dev`：集成分支，接受 `feat-<topic>` / `fix-<topic>` / `chore-<topic>` 分支的 PR。
  - 流程：`main` → 创建 `dev` → 从 `dev` 开功能分支 → PR 合入 `dev` → PR 合入 `main` → 在 `main` 打 tag 发布 → 发布后把 `main` 同步回 `dev`。
  - PR 必须通过 CI `verify` 才能合并；合并后删除功能分支。
- 保留原作者署名与 MIT 许可；不要删除 README 里的来源说明、`CREDITS.md` 和 `LICENSE`。
- `docs/releases/*`、`docs/appimage-acceptance.md` 等属于**历史记录**，其中指向原仓库/旧 issue/旧 CI 的链接保持原样，不要批量"修复"。

## 2. 技术栈与目录

| 位置 | 作用 |
| --- | --- |
| `electron/main.cjs` | Electron 主进程、窗口、IPC、更新器入口 |
| `electron/preload.cjs` | `contextBridge` 暴露的白名单 API |
| `electron/services/*.cjs` | 领域服务：Codeforces 同步、赛事/虚拟赛、训练、复习、模板、素材、更新、原子 JSON 等 |
| `electron/shared/*` | 主进程与测试共用的纯逻辑（如 `codeforces-transport.mjs`） |
| `src/` | React 渲染层（Vite 构建），`src/lib/*` 放可单测的纯逻辑（ESM） |
| `src/i18n/` | 中英双语文案（`index.jsx` + `en-US.js`） |
| `scripts/` | `node:test` 单测与 Playwright 验收（QA）脚本 |
| `docs/` | 用户/维护文档与历史发布记录 |
| `build/` | 图标、NSIS 脚本、AppStream 元数据 |

运行时约束：无后端、无账号密码、无遥测；只调用 Codeforces 公开 API。网络失败必须优雅降级，不得导致页面白屏或数据丢失。

## 3. 本地开发

```bash
# 依赖（Node 20+，CI 用 22；pnpm 版本由 packageManager 锁定）
pnpm install --frozen-lockfile

pnpm dev        # 仅 Vite 渲染层
pnpm desktop    # Vite + Electron 外壳
pnpm build      # 生产构建（= pnpm verify）
```

- Electron 相关代码是 **CommonJS**（`.cjs`），渲染层与 `src/lib` 是 **ESM**。
- 新增依赖前先确认能否用现有依赖或内置能力实现；不要引入遥测、字体 CDN 或远程脚本。
- 不要提交 `node_modules/`、`dist/`、`release*/`、`output/`、`.qa-output/`、日志、备份、绝对路径或凭据（见 `.gitignore`）。

## 4. 测试与验证

提交前至少运行与改动相关的测试，界面类改动再加对应 QA：

```bash
pnpm verify                                  # 生产构建，必须通过
node --test scripts/test-virtual-score.cjs   # 虚拟赛估分（时间戳容差相关）
pnpm test:virtual-rating                     # 估计 Rating
pnpm test:app-update                         # 更新服务
pnpm test:codeforces-transport               # CF 同源传输/重定向
pnpm test:custom-training
pnpm test:study-timer
pnpm test:analytics
pnpm qa:dual-rating                          # 需要构建产物 + Playwright
pnpm qa:study-timer
pnpm qa:app-update
pnpm qa:i18n
```

- 测试优先用 `node:test` + 纯函数；涉及窗口/IPC 的用现有 QA 脚本与隔离 profile。
- QA 报告与截图不得包含个人数据、真实 handle、绝对路径或本地备份。
- CI（`.github/workflows/build.yml`）在 push/PR 时只跑 `verify`；`package` 仅在下列情况运行：tag `v*`、`workflow_dispatch`、或提交信息包含 `[package-test]`。

## 5. 发布流程

1. 更新 `package.json` 的 `version`（语义化）。
2. 写 `CHANGELOG.md` 条目和 `docs/releases/vX.Y.Z.md`（中文 + English summary）。
3. 运行 `pnpm verify` 与相关测试；需要验证打包时可提交带 `[package-test]` 的 commit 触发 `package`（不会发布）。
4. 合并到 `main`，创建并推送 tag：

   ```bash
   git tag vX.Y.Z
   git push origin main
   git push origin vX.Y.Z
   ```

5. tag 必须与 `package.json` 版本一致（脚本会校验）。tag 推送后 CI 构建六个平台包、发布 GitHub Release，并回验下载与安装。
6. **禁止**强推 `main`、移动/重打已发布 tag、或在无 `package-test` 的情况下随意触发打包。

发布相关的已知边界：

- 本仓库的 tag push 可能不触发 Actions；未触发时可改用 `gh workflow run build.yml --ref vX.Y.Z`（ref 为 tag 时 release job 仍会执行）。
- 仓库内还没有任何已发布稳定版时，上一版本升级验证会明确跳过并在日志中记录；之后的稳定版继续执行该验证。
- Windows 自动更新仅支持安装版（NSIS），源固定为本仓库 GitHub 稳定发布；v4.3.4 及更早版本的旧更新源已失效，需手动迁移。
- AppImage 使用外部工具（AppImageUpdate 等）的 `.zsync`，应用内不检查更新。
- macOS 为 ad-hoc 签名，未公证。

## 6. 硬性约束（不可违反）

1. **隐私**：不提交用户数据、备份、日志、绝对路径、真实 Codeforces handle、令牌；不新增上传/遥测/统计。
2. **安全**：所有窗口保持 `contextIsolation: true`、`nodeIntegration: false`、`sandbox: true`；IPC 走既有 trusted 包装；不放宽 CSP；不执行远程代码。
3. **素材版权**：不提交第三方角色图、游戏素材、壁纸或提取资源；只支持用户本地导入（见 `docs/OPTIONAL_CHARACTER_ASSETS.md`）。
4. **数据安全**：写本地 JSON 必须使用 `electron/services/atomic-json.cjs`；结构变更需兼容旧数据，不得静默丢弃用户记录。
5. **双语**：所有用户可见文案同时更新中文与英文（`src/i18n/`）。
6. **主题**：开源版主题固定为天空蓝、薄荷绿、珊瑚粉；新增/删除主题需先讨论。
7. **署名与许可**：MIT；保留 `LICENSE`、`CREDITS.md`、原作者与 fork 来源说明。

## 7. 代码约定

- 提交信息用英文 conventional 前缀：`feat:` / `fix:` / `docs:` / `chore:` / `refactor:` / `test:`。
- 领域逻辑尽量放进 `electron/services/*.cjs` 或 `src/lib/*` 的纯函数，便于单测；React 组件只做展示与交互。
- Codeforces Rating 颜色逻辑集中在 `src/components/RatingDisplay.jsx` 与 `src/lib/stats.js`。
- Codeforces 网络访问统一走 `electron/shared/codeforces-transport.mjs`；不要新增裸 `fetch`/`https` 直连。
- 同步/缓存失败要有可恢复路径：保留旧缓存、给出明确文案、避免整页不可用。
- 新增设置项要有默认值、向后兼容和对应文案。

## 8. AI 代理作业规则

- 动手前先读 `package.json` scripts、相关 `docs/` 与本文件；不确定的接口以代码与测试为准。
- 改动范围保持最小；大重构先给计划，不要顺手改无关文件。
- 不要使用 `git commit --no-verify`、不要强推、不要改历史；不要向 `upstream`/`archive` 推送。
- 不要为了让测试通过而删除/放宽既有回归用例；测试反映真实约束（例如虚拟赛时间戳容差、沙箱标志、更新源固定）。
- 涉及 UI 的改动要实际渲染检查（Playwright QA 或 `pnpm desktop`），不能只看代码。
- 外部服务（Codeforces API）不可用时不要猜测数据；用本地缓存并明确提示。
- 完成改动后：运行最小相关测试 + `pnpm verify`，更新必要文档；如果改变了开发/发布/架构约定，**同步更新本文件**。

## 9. 常见任务速查

| 任务 | 先看 | 验证 |
| --- | --- | --- |
| 虚拟赛估分/时间戳 | `electron/services/virtual-score.cjs`、`scripts/test-virtual-score.cjs` | `node --test scripts/test-virtual-score.cjs`、`pnpm test:virtual-rating` |
| 赛事复盘/赛事中心 | `electron/services/contest-*.cjs`、`src/components/Contest*.jsx` | `pnpm test:contest-sessions`、`pnpm qa:contest-sessions` |
| 今日训练/复习 | `src/lib/planning.js`、`review.js`、`stats.js` | `pnpm test:analytics`、`pnpm qa:layout` |
| 模板库 | `src/lib/templates.js`、`template-summary-format.js` | `node scripts/test-template-summary-format.mjs`、`pnpm qa:workbench` |
| 外观/壁纸 | `src/lib/appearance.js`、`electron/services/wallpaper-service.cjs` | `pnpm test:materials`、`pnpm qa:wallpaper`、`pnpm qa:materials` |
| 应用更新 | `electron/services/app-update-service.cjs`、`scripts/test-app-update-service.cjs` | `pnpm test:app-update`、`pnpm qa:app-update` |
| 打包/发布 | `package.json` build 段、`.github/workflows/build.yml`、`docs/*appimage*`、`docs/windows-auto-update.md` | `pnpm desktop:build` / tag 后看 CI |

## 10. 已知缺口

- 新仓库尚未发布任何 GitHub Release；安装包需要先构建。
- 依赖未随仓库提供，首次开发先 `pnpm install`。
- 历史文档中的原仓库链接已失效（保留为历史记录）。
- Codeforces 接口或页面结构变化时，同步/复盘可能暂时不可用；修复需保留缓存与降级提示，并补回归测试。
