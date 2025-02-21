
# Git 和 GitHub 常用操作

## Git 基本操作

### 1. 初始化一个 Git 仓库
在项目目录下，使用以下命令初始化一个新的 Git 仓库：
```bash
git init
```

### 2. 配置 Git 用户信息
在开始使用 Git 前，需要配置用户名和邮箱：
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### 3. 查看 Git 配置信息
查看当前的 Git 配置信息：
```bash
git config --list
```

### 4. 克隆远程仓库
将远程 Git 仓库克隆到本地：
```bash
git clone https://github.com/username/repository.git
```

### 5. 查看文件状态
查看当前工作目录中文件的状态（比如是否有未跟踪的文件、是否有更改）：
```bash
git status
```

### 6. 添加文件到暂存区
将文件添加到 Git 暂存区：
```bash
git add filename
```
或者添加所有文件：
```bash
git add .
```

### 7. 提交更改
将暂存区的更改提交到本地仓库：
```bash
git commit -m "Your commit message"
```

### 8. 查看提交历史
查看项目的提交历史：
```bash
git log
```

### 9. 推送本地更改到远程仓库
将本地的更改推送到远程仓库：
```bash
git push origin main
```

### 10. 拉取远程更改到本地
将远程仓库的更改拉取到本地仓库：
```bash
git pull origin main
```

### 11. 创建分支
创建一个新的分支：
```bash
git branch new-branch-name
```

### 12. 切换分支
切换到指定的分支：
```bash
git checkout branch-name
```

### 13. 合并分支
将指定分支的更改合并到当前分支：
```bash
git merge branch-name
```

### 14. 删除分支
删除本地分支：
```bash
git branch -d branch-name
```

## [GitHub](https://github.com/) 常用操作

### 1. 创建一个新的 GitHub 仓库
1. 登录到 [GitHub](https://github.com/)。
2. 点击右上角的 **+** 图标，选择 **New repository**。
3. 填写仓库名称、描述等信息，然后点击 **Create repository**。

### 2. 将本地仓库推送到 GitHub
1. 在本地仓库中初始化 Git 并进行一些提交（如 `git init` 和 `git add .`）。
2. 将远程仓库添加为 Git 远程仓库：
   ```bash
   git remote add origin https://github.com/username/repository.git
   ```
3. 推送本地更改到 GitHub：
   ```bash
   git push -u origin main
   ```

### 3. 创建 Pull Request (PR)
1. 在 GitHub 仓库页面，点击 **Pull requests** 标签。
2. 点击 **New pull request**，选择源分支和目标分支。
3. 输入 PR 的描述并点击 **Create pull request**。

### 4. Fork 和 Pull Request 工作流程
1. Fork 其他人的 GitHub 仓库。
2. 克隆 Fork 后的仓库到本地并创建一个新分支。
3. 做出更改并提交，然后推送到自己的 GitHub 仓库。
4. 在原仓库的 **Pull requests** 标签中发起 PR，提交更改。

### 5. 查看远程仓库的状态
查看当前远程仓库的详细信息：
```bash
git remote -v
```

## 常用 Git 命令

| 命令                         | 描述                               |
|------------------------------|------------------------------------|
| `git init`                    | 初始化 Git 仓库                   |
| `git clone <repository>`       | 克隆远程仓库到本地                 |
| `git add <file>`              | 将文件添加到暂存区                 |
| `git commit -m "message"`     | 提交更改到本地仓库                 |
| `git status`                  | 查看当前工作目录中文件的状态       |
| `git log`                     | 查看提交历史                       |
| `git push`                    | 将本地更改推送到远程仓库           |
| `git pull`                    | 从远程仓库拉取更改到本地           |
| `git branch`                  | 查看本地分支                       |
| `git checkout <branch>`       | 切换分支                           |
| `git merge <branch>`          | 合并分支                           |
| `git remote add origin <url>` | 添加远程仓库                       |


<学习网站>[https://learngitbranching.js.org/?locale=zh_CN]