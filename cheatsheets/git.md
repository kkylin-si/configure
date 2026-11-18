# Git

## 通过命令在指定的repo 里面指定提交者和作者信息

**完全可以。** 和 `core.sshCommand` 一样，只要不加 `--global`，在仓库目录里直接运行 `git config`，配置就会写入当前仓库的 `.git/config`，**只影响这一个仓库**，而且优先级高于 `~/.gitconfig` 和 `includeIf`。

---

## 一、对已有仓库：进入目录直接设置

### 情况 1：作者和提交者是同一人（最常见）
```bash
cd /path/to/repo

git config user.name "你的名字"
git config user.email "你的邮箱"
git config core.sshCommand "ssh -i ~/.ssh/你的私钥 -o IdentitiesOnly=yes"
```

### 情况 2：需要把“作者”和“提交者”设为不同的人（Git 2.22+）
```bash
cd /path/to/repo

git config author.name "作者名字"
git config author.email "author@example.com"
git config committer.name "提交者名字"
git config committer.email "committer@example.com"
```
*(如果同时配置了 `user.*` 和 `author.*`/`committer.*`，Git 会优先使用 `author.*` 和 `committer.*`。)*

---

## 二、对新仓库：`git clone -c` 一步到位

克隆新项目时，可以直接用 `git clone -c`（`--config`）：它会在本地初始化完仓库、**连接远程仓库拉取代码之前**就把配置写入新仓库的 `.git/config`。[1]

也就是说，**既能解决首次 `clone` 选密钥的问题，又能把身份顺手固化到新仓库里**：

```bash
git clone \
  -c core.sshCommand="ssh -i ~/.ssh/你的私钥 -o IdentitiesOnly=yes" \
  -c user.name="你的名字" \
  -c user.email="你的邮箱" \
  git@github.com:org/repo.git
```

如果是公司/个人拆开身份，也可以把这些敲成一个 Shell 别名（写进 `~/.zshrc`），以后克隆直接用：

```bash
# 加到 ~/.zshrc 后执行 source ~/.zshrc
alias git-clone-work='git clone -c core.sshCommand="ssh -i ~/.ssh/id_ed25519_work -o IdentitiesOnly=yes" -c user.name="WorkName" -c user.email="work@company.com"'
```
以后克隆工作仓库只需要：
```bash
git-clone-work git@github.com:company/repo.git
```

---

## 三、验证是否已经绑定成功

进到仓库目录后运行：

```bash
# 查看当前仓库生效的本地配置
git config --local --list

# 或者查看完整来源
git config --show-origin --get-regexp "^(user|author|committer|core\.sshCommand)"
```

提交一次后（或者用 `git commit --allow-empty -m "test"` 测试后回退），可以用以下命令确认记录里的 Author 和 Committer 是否正确：
```bash
git log -1 --format=fuller
```
