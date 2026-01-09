# Git 敏感信息检测配置说明

## ✅ 已完成配置

### 1. 安装工具
- ✅ gitleaks v8.28.0 已安装

### 2. Git Hooks 配置
- ✅ pre-commit hook：在每次 commit 前检测敏感信息
- ✅ pre-push hook：在每次 push 前检测敏感信息

### 3. 自定义检测规则
已创建 `.gitleaks.toml` 配置文件，包含以下检测规则：
- 通用密码模式
- API密钥
- GitHub/AWS/阿里云/腾讯云等平台密钥
- JWT Token
- 数据库连接字符串
- Docker登录命令
- 私钥
- JFrog Token
- Slack Token
- 可疑的密钥赋值

## 🔍 检测结果

扫描发现 **34 处敏感信息**，详细信息见 `.gitleaks-report.json`

主要问题文件：
1. `飞书文档迁移/03-知识记录/04-软件开发/06-服务器知识/docker安装mysql.md`
   - MySQL密码
   - Docker登录凭证

2. `飞书文档迁移/03-知识记录/04-软件开发/08-MYSQL数据迁移/（全量）Mysql-xtrabackup-demo.md`
   - MySQL密码

3. 其他日报和文档中的密码信息

## 🛠️ 如何使用

### 正常工作流程
```bash
# 1. 正常添加文件
git add .

# 2. 提交时会自动检测
git commit -m "your message"
# 如果检测到敏感信息，会阻止提交并显示详情

# 3. push时会再次检测
git push
# 如果检测到敏感信息，会阻止push
```

### 手动扫描
```bash
# 扫描所有文件
gitleaks detect --config=.gitleaks.toml --verbose

# 扫描并生成报告
gitleaks detect --config=.gitleaks.toml --report-path=report.json --report-format=json

# 扫描特定文件
gitleaks detect --config=.gitleaks.toml --verbose --source=文件路径
```

## 📋 处理现有敏感信息

### 方案1：移除敏感内容（推荐）
```bash
# 1. 编辑文件，将真实密码替换为占位符
# 例如：将真实密码改为 YOUR_PASSWORD_HERE 或 xxx

# 2. 使用 git filter-repo 清理历史（需先安装）
# brew install git-filter-repo

# 3. 清理特定文件的历史
git filter-repo --path 文件路径 --invert-paths
```

### 方案2：修改历史记录
```bash
# 1. 找到包含敏感信息的commit
git log --all --full-history -- "文件路径"

# 2. 交互式rebase修改历史
git rebase -i commit_id^

# 3. 在编辑器中将要修改的commit标记为 edit
# 4. 修改文件内容
# 5. 继续rebase
git add .
git rebase --continue

# 6. 强制推送（谨慎！）
git push -f origin branch-name
```

### 方案3：使用 BFG Repo-Cleaner（快速）
```bash
# 1. 安装BFG
brew install bfg

# 2. 创建密码文件
echo "敏感密码1" > passwords.txt
echo "敏感密码2" >> passwords.txt

# 3. 清理密码
bfg --replace-text passwords.txt

# 4. 清理reflog
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 5. 强制推送
git push -f origin --all
```

## ⚠️ 重要注意事项

1. **已泄露的密码必须更换**
   - 如果密码已经push到GitHub，即使删除历史，也应该认为已泄露
   - 建议立即更换所有检测到的真实密码

2. **团队协作**
   - 在修改历史前，通知团队成员
   - 强制推送会影响其他人的工作

3. **备份**
   - 在执行历史修改前，建议先备份仓库

4. **.gitignore**
   - 将检测报告添加到 .gitignore
   ```bash
   echo ".gitleaks-report.json" >> .gitignore
   ```

## 🔧 自定义配置

如需修改检测规则，编辑 `.gitleaks.toml` 文件：

```toml
# 添加忽略规则
[allowlist]
paths = [
  '''path/to/ignore''',
]

# 添加停止词（不触发检测的字符串）
stopwords = [
  "example",
  "test",
]

# 添加自定义规则
[[rules]]
id = "my-custom-rule"
description = "自定义检测规则"
regex = '''your-regex-pattern'''
tags = ["custom"]
```

## 📞 获取帮助

- gitleaks文档：https://github.com/gitleaks/gitleaks
- 如有问题，检查 `.gitleaks-report.json` 获取详细信息
