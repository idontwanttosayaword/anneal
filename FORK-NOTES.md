# Fork 二开规范

本仓库是 [mosonlab/anneal](https://github.com/mosonlab/anneal) 的 fork，用于二次开发。基线：v0.8.0（tag，commit 44e9b326）。

## remote 布局

- `origin` → idontwanttosayaword/anneal（自己的二开仓库，可写）
- `upstream` → mosonlab/anneal（官方，只读，永不 push）

## 二开三原则

1. **新增优于修改**。新代码放 `packages/` 下的新目录（workspaces 是 `apps/*`、`packages/*` glob，新目录自动纳入，不用改 `package.json`）。每修改一个上游文件，就多一个永久冲突点。
2. **每个新文件登记进 `public-snapshot.json`**。include 列表按路径字母序插入 `{ "glob": ..., "purpose": ... }` 条目，否则 merge-gate 和 `snapshot:scan` 按 blocker 默认处置拒绝。升级合并时这份清单必然与上游冲突，解决方式是把两边的条目都保留。
3. **避开重灾区**：
   - `packages/db`：上游每个 release 都新增迁移，且迁移集带 release attestation 校验，二开不碰它；
   - `package-lock.json`：冲突时直接取上游版本，然后 `npm install` 把二开的依赖重新锁进去，不做手工合并。

## 升级流程（每个 release tag 执行一次）

```sh
git fetch upstream --tags
git merge v0.x.0
npm ci
npm run build
scripts/merge-gate.sh --expect-head "$(git rev-parse HEAD)"
git push origin main
```

- 只跟 release tag，不追 `upstream/main`：基线时它与 v0.8.0 之间已领先 1185 个提交。
- preview 之间没有数据升级路径，升级后按官方流程重建数据库（`npm run db:migrate:release -- --fresh`）。
- 上游不接受外部 PR（见 CONTRIBUTING.md），二开的修复无法回馈，差异只增不减，所以改动面控制是第一原则。

## 本仓库的二开改动登记

| 文件/目录 | 内容 |
| --- | --- |
| `FORK-NOTES.md` | 本文件，fork 规范 |