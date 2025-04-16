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

## 分支管理

```bash
# 创建特性分支（基于主分支）
git checkout -b feat/user-profile main

# 定期合并主分支更新
git checkout feat/user-profile
git merge main

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
