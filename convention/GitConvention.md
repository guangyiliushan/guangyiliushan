# Git/GitHub 最佳实践范式

## 提交规范

```bash
# 类型示例（前缀小写，冒号后空格）
git commit -m "feat: 新增用户登录功能"
git commit -m "fix: 修复首页样式错位问题"
git commit -m "docs: 更新API接口文档"
git commit -m "style: 调整代码缩进格式"
git commit -m "refactor: 重构用户模块数据层"
git commit -m "perf: 优化图片懒加载性能"
git commit -m "test: 添加购物车单元测试"
git commit -m "build: 升级webpack至v5.0"
git commit -m "ci: 配置GitHub Actions流水线"
git commit -m "chore: 更新项目依赖版本"
git commit -m "revert: 回滚错误的数据库迁移"
```

## 分支管理（Gitflow Workflow）

##  Git 常用分支介绍

###  `dev` 分支

- 此为主要开发分支，涵盖所有要发布到下一个版本的代码。其他分支（如 `feat`、`release`、`hotfix`）的代码最终都会合并到该分支。
- `dev` 分支基于 `main` 分支创建。

###  `feat-<feature-name>` 分支

- 用于开发新功能。`<feature-name>` 要简洁明了，能体现该功能的核心内容。功能开发完成后，会将此分支代码合并回 `dev` 分支。
- 功能开发完毕后，需将 `feat-<feature-name>` 分支合并回 `dev` 分支。合并完成后，可删除该分支。

###  `release-<version>` 分支

- 在准备发布新版本时，基于 `dev` 分支创建。`<version>` 遵循版本号规则，像 `1.0.0`。在此分支可进行测试和修复 Bug。
- 发布完成后，会将代码合并到 `main` 和 `dev` 分支。基于 `dev` 分支创建 `release-<version>` 分支。创建后，可在此分支进行测试和修复 Bug。
- 同时，其他开发人员可基于 `dev` 分支创建新的 `feat` 分支。发布时，将 `release-<version>` 分支合并到 `main` 和 `dev` 分支，在 `main` 分支打标签记录版本号，之后删除 `release-<version>` 分支。

###  `hotfix-<version>` 分支

- 当生产环境出现紧急 Bug 时，基于 `main` 分支创建 `hotfix-<version>` 分支。`<version>` 为修复后的版本号。修复完成后，同时在 `main` 分支打标签。

###  `main` 分支

- 代表生产环境的代码，存放最近发布到生产环境的版本。此分支只能从其他分支（如 `release`、`hotfix`）合并，不能直接修改。`main` 分支上的每个提交都应打标签，以记录版本号。通常 `main` 分支不会有直接提交。

### Git Flow 命令示例

####  `dev` 相关

```bash
git branch dev
git push -u origin dev
```

#### feat 相关

开始新功能开发从 dev 分支创建新的 feat 分支：

```bash
git checkout -b feat-new-feature dev
git push -u origin feat-new-feature
```

修改代码

```bash
git status
git add .
git commit -m "完成新功能开发"
```

完成功能开发

```bash
git pull origin dev
git checkout dev
git merge --no-ff feat-new-feature
git push origin dev
git branch -d feat-new-feature
git push origin --delete feat-new-feature
```
#### release 相关

开始发布流程

```bash
git checkout -b release-1.0.0 dev
```

完成发布流程

```bash
git checkout main
git merge --no-ff release-1.0.0
git push
git checkout dev
git merge --no-ff release-1.0.0
git push
git branch -d release-1.0.0
git push origin --delete release-1.0.0
git tag -a v1.0.0 main
git push --tags
```

#### hotfix 相关

开始紧急修复

```bash
git checkout -b hotfix-1.0.1 main
```
完成紧急修复

```bash
git checkout main
git merge --no-ff hotfix-1.0.1
git push
git checkout dev
git merge --no-ff hotfix-1.0.1
git push
git branch -d hotfix-1.0.1
git push origin --delete hotfix-1.0.1
git tag -a v1.0.1 main
git push --tags
```

## 本地分支管理命令

```bash
# 创建特性分支（基于主分支）
git checkout -b feat/user-profile main

# 定期合并主分支/更新
git checkout feat/user-profile
git merge dev

# 删除已合并分支（本地）
git branch -d feat/user-profile
# 强制删除未合并分支（慎用）
git branch -D hotfix/login-error
```

## 远程仓库操作

```bash
# 推送特性分支至远程
git push origin feat/user-profile

# 拉取最新代码（避免冲突）
git pull --rebase origin main

# 创建Pull Request（GitHub CLI示例）
gh pr create --base main --head feat/user-profile --title "feat: 新增用户资料模块"

# 合并前Squash Commits（交互式变基）
git rebase -i HEAD~3
# 在编辑界面将pick改为squash/fixup后保存
```

## 保护性策略

GitHub Branch Protection Rules

```bash
# 主分支保护（示例：GitHub设置）
gh api -X PUT repos/:owner/:repo/branches/main/protection \

  -H "Accept: application/vnd.github.v3+json" \
  # 状态检查增强：要求通过 CI/CD 流水线检查
  -F "required_status_checks={\"strict\":true,\"contexts\":[\"ci/cd\"]}" \ 
  # 强制管理员遵守规则
  -F "enforce_admins=true" \ 
  # 代码审查强化：要求代码所有者参与审查，至少 1 人通过
  -F "required_pull_request_reviews={\"dismiss_stale_reviews\":true,\"require_code_owner_reviews\":true,
  \"required_approving_review_count\":1}" \ 
  # 操作限制：禁用强制推送
  -F "allow_force_pushes=false" \ 
   # 操作限制：防止分支被误删
  -F "allow_deletions=false" \
  # 权限精细化：限制特定用户/团队拥有推送权限
  -F "restrictions={\"users\":[\"user1\"],\"teams\":[\"team1\"],\"apps\":[]}" 
```
