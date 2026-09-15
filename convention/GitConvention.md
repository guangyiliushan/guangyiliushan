# Git/GitHub 最佳实践范式

## 分支模型选型（按项目规模与团队人数）

| 模型 | 适用场景 | 长期分支 | 短期分支 | 团队规模建议 |
|------|---------|---------|---------|-------------|
| **GitHub Flow** | 小项目、持续部署、Web 应用 | `main` | `feat/*` | 1-10 人 |
| **GitLab Flow** | 中型项目、多环境部署（staging/production） | `main` + 环境分支 | `feat/*` | 5-30 人 |
| **Git Flow** | 大型项目、版本化发布、多版本并行维护 | `main` + `dev` | `feat/*` `release/*` `hotfix/*` | 10+ 人 |
| **Trunk-Based Dev** | 强 CI/CD、特性开关成熟、资深团队 | `main`（trunk） | 存活 ≤ 2 天的短分支 | 任意（需工程成熟度） |

选型原则：

- 项目越简单、发布越频繁，选越轻量的模型。
- 需要同时维护多个生产版本（如移动端 App 多版本共存），选 Git Flow。
- 有独立 staging / production 环境部署流程，选 GitLab Flow 环境分支。
- 团队具备完善自动化测试和特性开关能力，选 Trunk-Based Development。

---

## 提交规范（Conventional Commits 1.0.0）

格式：`<type>(<scope>): <subject>`

```bash
# 类型示例（前缀小写，冒号后空格，scope 可选）
git commit -m "feat(auth): 新增用户登录功能"
git commit -m "fix(ui): 修复首页样式错位问题"
git commit -m "docs(api): 更新接口文档"
git commit -m "style: 调整代码缩进格式"
git commit -m "refactor(user): 重构数据层"
git commit -m "perf(image): 优化图片懒加载性能"
git commit -m "test(cart): 添加购物车单元测试"
git commit -m "build: 升级 webpack 至 v5.0"
git commit -m "ci: 配置 GitHub Actions 流水线"
git commit -m "chore(deps): 更新项目依赖版本"
git commit -m "feat!: 不兼容的 API 变更"
git commit -m "revert: 回滚 feat(auth): 新增登录"
```

类型与 SemVer 对应关系：

- `feat` → MINOR 版本号递增
- `fix` → PATCH 版本号递增
- `!` 后缀或 footer 中 `BREAKING CHANGE:` → MAJOR 版本号递增

---

## 分支命名规范

```text
main                # 生产环境代码，受保护
dev                 # 集成分支（仅 Git Flow 使用）
feat/<简短描述>      # 新功能
fix/<简短描述>       # Bug 修复
release/<版本号>     # 发布准备
hotfix/<版本号>      # 紧急修复
```

命名规则：使用 `/` 分隔前缀与名称；名称用 kebab-case；保持简短且有意义。

---

## Git Flow 工作流（大项目 / 大团队）

### 分支职责

| 分支 | 来源 | 合并目标 | 生命周期 |
|------|------|---------|---------|
| `main` | - | - | 永久，生产就绪，每次合并打 tag |
| `dev` | `main` | - | 永久，集成最新开发成果 |
| `feat/*` | `dev` | `dev` | 短期，功能完成即删 |
| `release/*` | `dev` | `main` + `dev` | 短期，发布测试期 |
| `hotfix/*` | `main` | `main` + `dev` | 短期，修复后即删 |

### 初始化

```bash
git branch dev
git push -u origin dev
```

### 功能开发

```bash
# 从 dev 创建功能分支
git checkout -b feat/user-profile dev
git push -u origin feat/user-profile

# 开发并提交
git status
git add .
git commit -m "feat(user): 新增用户资料模块"

# 完成后合并回 dev
git checkout dev
git pull origin dev
git merge --no-ff feat/user-profile
git push origin dev
git branch -d feat/user-profile
git push origin --delete feat/user-profile
```

### 发布流程

```bash
# 从 dev 创建发布分支
git checkout -b release/1.0.0 dev

# ... 测试、修复发布分支上的 Bug ...

# 发布：合并到 main 并打标签
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

# 同步回 dev（关键步骤，不可省略）
git checkout dev
git merge --no-ff release/1.0.0
git push origin dev

# 清理
git branch -d release/1.0.0
git push origin --delete release/1.0.0
```

### 紧急修复

```bash
# 从 main 创建 hotfix 分支
git checkout -b hotfix/1.0.1 main

# ... 修复、测试 ...

# 合并到 main 并打标签
git checkout main
git merge --no-ff hotfix/1.0.1
git tag -a v1.0.1 -m "Hotfix version 1.0.1"
git push origin main --tags

# 同步回 dev（关键步骤，不可省略）
git checkout dev
git merge --no-ff hotfix/1.0.1
git push origin dev

# 清理
git branch -d hotfix/1.0.1
git push origin --delete hotfix/1.0.1
```

---

## GitHub Flow 工作流（小项目 / 小团队）

```bash
# 1. 从 main 创建功能分支
git checkout -b feat/user-profile main

# 2. 开发并提交
git add .
git commit -m "feat(user): 新增用户资料模块"

# 3. 推送并创建 Pull Request
git push -u origin feat/user-profile
gh pr create --base main --head feat/user-profile \
  --title "feat(user): 新增用户资料模块"

# 4. Code Review 通过后 Squash 合并（在 GitHub UI 或 CLI 上操作）
gh pr merge --squash

# 5. 清理本地分支
git checkout main
git pull origin main
git branch -d feat/user-profile
```

---

## GitLab Flow 工作流（多环境部署）

```text
main  -->  staging  -->  production
```

- `main`：开发主线，自动部署到开发/测试环境。
- `staging`：预发环境分支，`main` 代码经 MR/PR 合入。
- `production`：生产环境分支，`staging` 验证通过后合入。
- 代码只能由上游分支向下游分支单向流动，不允许反向合并。

---

## 合并策略与 Rebase 规则

### Merge 的三种模式

| 模式 | 命令 / 操作 | 历史效果 | 适用场景 |
|------|-----------|---------|---------|
| Fast-forward | `git merge`（默认） | 线性，无 merge commit | 私有分支间更新，无分叉时最简洁 |
| `--no-ff` | `git merge --no-ff` | 保留分支拓扑 + merge node | Git Flow：feat 合入 dev/main |
| Squash | `git merge --squash` 或平台按钮 | 线性，多个 commit 压缩为一个 | GitHub Flow / Trunk-Based：PR 合入 main |

`--no-ff` 的核心价值：

> The --no-ff flag causes the merge to always create a new commit object, even if the merge could be performed with a fast-forward. This avoids losing information about the historical existence of a feature branch.
>
> -- Vincent Driessen, nvie.com

实际好处：`git log --graph` 清晰展示 feature 起止；`git bisect --first-parent` 可将 feature 作为原子单元调试；revert 一个 merge commit 即可撤销整个 feature。

Squash 的取舍：`main` 历史干净可读，但 feature 内部的 commit 粒度信息完全丢失，无法逐步 bisect feature 内的变化。

### Rebase 规则

**黄金法则：永远不要 rebase 已推送到远程且他人可能基于其工作的公共分支（`main`、`dev`、`release/*` 等）。**

Git 官方文档原文（git-scm.com Git Book）：

> Do not rebase commits that exist outside your repository and that people may have based work on.

```bash
# 允许：rebase 自己的私有功能分支，保持提交历史整洁
git checkout feat/user-profile
git rebase dev

# 允许：合并前交互式整理自己的提交
git rebase -i HEAD~3

# 禁止：rebase main / dev 等共享分支
# 禁止：对他人协作中的分支执行 rebase + force push

# 合并策略选择
git merge --no-ff feat/user-profile   # Git Flow：保留分支拓扑
                                      # GitHub Flow 推荐使用 Squash Merge
```

### PR 合并策略选择（GitHub / GitLab 平台）

| 合并按钮 | main 历史效果 | 保留 commit 粒度 | 推荐模型 |
|---------|-------------|-----------------|---------|
| **Merge commit** | 保留分支拓扑 + merge node | 是（含中间 WIP） | Git Flow |
| **Squash and merge** | 线性，一个 PR 一个 commit | 否（压缩为一个） | GitHub Flow / Trunk-Based |
| **Rebase and merge** | 线性，保留分支内每个 commit | 是（线性排列） | GitHub Flow（commit 有意义时） |

选择原则：

- feature 分支上有大量 WIP/调试 commit → 选 Squash，保持 `main` 干净。
- feature 分支上每个 commit 都是原子且有意义的 → 选 Rebase Merge，保留粒度但不要 merge node。
- 需要在 `main` 上看到 feature 分支的起止边界 → 选 Merge Commit。

### 按分支模型推荐合并策略

| 模型 | feat → dev/main | PR 合并策略 | 理由 |
|------|----------------|-----------|------|
| **Git Flow** | `--no-ff` merge | Merge commit | 保留拓扑，`bisect --first-parent` 可定位到 feature |
| **GitHub Flow** | - | Squash and merge | `main` 线性干净，一个 PR 对应一个功能 |
| **GitLab Flow** | 上游 → 下游 | Merge commit 或 Rebase | 环境分支间需保留可追溯性 |
| **Trunk-Based** | 短分支 → main | Squash（大多数场景） | 频繁合并，历史必须极简 |

---

## 版本标签（SemVer 2.0.0）

格式：`v<MAJOR>.<MINOR>.<PATCH>`，可选预发布标识：`v1.2.3-beta.1`

```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

版本号递增规则：

- `MAJOR`：不兼容的 API 变更（`feat!` 或 `BREAKING CHANGE`）
- `MINOR`：向后兼容的新功能（`feat`）
- `PATCH`：向后兼容的 Bug 修复（`fix`）

---

## 分支保护策略

对 `main` 和 `dev` 均应启用以下保护规则：

| 规则 | 说明 |
|------|------|
| Require pull request reviews | 至少 1 人审批（大团队建议 2 人） |
| Require status checks | CI/CD 流水线必须通过 |
| Require conversation resolution | 所有 PR 讨论已解决 |
| Require signed commits | 提交必须签名验证 |
| Require linear history | 禁止 merge commit（GitHub Flow 推荐启用） |
| Block force pushes | 禁止强制推送 |
| Restrict deletions | 禁止删除受保护分支 |
| Enforce for administrators | 管理员同样受约束 |

### GitHub CLI 配置示例（REST API）

```bash
gh api -X PUT repos/:owner/:repo/branches/main/protection \
  -H "Accept: application/vnd.github+json" \
  -F "required_status_checks[strict]=true" \
  -F "required_status_checks[contexts][]=ci/cd" \
  -F "enforce_admins=true" \
  -F "required_pull_request_reviews[dismiss_stale_reviews]=true" \
  -F "required_pull_request_reviews[require_code_owner_reviews]=true" \
  -F "required_pull_request_reviews[required_approving_review_count]=1" \
  -F "allow_force_pushes=false" \
  -F "allow_deletions=false" \
  -F "required_linear_history=true"
```

### 协作合并控制（谁执行合并）

| 团队规模 | 合并方式 | 说明 |
|---------|---------|------|
| ≤ 5 人 | 个人直接 merge + push 或轻量 PR | 信任度高，审查可通过 pair programming 完成 |
| 5-30 人 | 必须通过 PR/MR 审批后由平台合并 | 分支保护规则强制执行 |
| 30+ 人 | PR 审批 + 指定 Release Manager / Tech Lead 合并 | 合并权限限制到特定角色 |

核心原则：

- 所有合入共享分支（`main` / `dev` / `release/*`）的操作必须通过 PR/MR 审批。
- 个人不得直接 push 到受保护分支。
- 功能分支上的 rebase 由分支所有者在本机执行；合并到共享分支由平台完成。
- Trunk-Based 模式下，代码审查通过后由 CI 自动合并（pair programming 可替代 PR 审批）。

---

## 日常分支管理

```bash
# 查看本地/远程分支
git branch -a

# 定期同步上游变更（避免大冲突）
git checkout feat/user-profile
git rebase dev

# 删除已合并分支（本地）
git branch -d feat/user-profile

# 强制删除未合并分支（慎用，仅确认不需要时）
git branch -D feat/abandoned-idea

# 清理已删除远程分支的本地引用
git fetch --prune
```

---

## 参考

- Git 官方文档（Rebasing）：https://git-scm.com/book/en/v2/Git-Branching-Rebasing
- Git 官方文档（git-merge）：https://git-scm.com/docs/git-merge
- GitHub 官方文档（Pull request merges）：https://docs.github.com/en/pull-requests/reference/pull-request-merges
- GitLab 官方文档（Merge methods）：https://docs.gitlab.com/user/project/merge_requests/methods/
- GitLab 官方文档（Squash and merge）：https://docs.gitlab.com/user/project/merge_requests/squash_and_merge/
- Git Flow 原始文章（nvie.com）：https://nvie.com/posts/a-successful-git-branching-model/
- Atlassian（Merging vs. Rebasing）：https://www.atlassian.com/git/tutorials/merging-vs-rebasing
- Atlassian（Git rebase）：https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase
- Paul Hammant（Google & Facebook Trunk-Based Dev）：https://paulhammant.com/2014/01/08/googles-vs-facebooks-trunk-based-development/index.html
