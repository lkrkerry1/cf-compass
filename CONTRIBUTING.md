# Contributing

感谢你帮助改进 CF Compass。

## 开始之前

1. 分支模型：`main` 为稳定/发布分支，`dev` 为集成分支；功能开发从 `dev` 创建 `feat-<topic>`、`fix-<topic>` 或 `chore-<topic>` 分支。
2. 开发完成后向 `dev` 提 PR，CI `verify` 通过后合并；发布时再向 `main` 提 PR，合并后在 `main` 打 tag。
3. 使用 Node.js 20+ 与 pnpm。
4. 运行 `pnpm install --frozen-lockfile` 安装依赖。
5. 提交前运行 `pnpm verify`。

## 开源版边界

- 主题色固定为天空蓝、薄荷绿、珊瑚粉；新增或删除主题请先开 Issue 讨论。
- 不要提交角色图片、游戏素材、第三方壁纸或从其他软件提取的素材。
- 不要提交本地用户数据、备份、日志、绝对路径或个人凭据。
- Codeforces Rating 颜色逻辑集中在 `src/components/RatingDisplay.jsx` 与 `src/lib/stats.js`。

## Pull Request

请说明改动目的、验证方法和用户可见影响。涉及界面时附上不含个人信息的截图；涉及数据结构时说明兼容和迁移方式。
