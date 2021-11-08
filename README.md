# Ethereum-Academy
Ethereum Academy is a technical seminar.

## 1. 怎么分享讲义？

**1.1.** 克隆这个仓库到本地：

```bash
$ git clone https://github.com/MonteCarloClub/Ethereum-Academy.git
```

**1.2.** 新建分支，在新分支新增讲义并提交。建议新分支上只有单一提交；另外，讲义最好是 *.md 或 *.pdf 格式。

```bash
$ cd Ethereum-Academy
$ git checkout -b <NEW_BRANCH>
$ # 新增讲义
$ git add .
$ git commit -m "write a message here"
```

**1.3.** 推送提交到远程仓库。

```bash
$ git push -u origin <NEW_BRANCH>
```

**1.4.** [新建 PR](https://github.com/MonteCarloClub/Ethereum-Academy/compare)，请求把新分支的提交合并到主分支，@KofClubs 将 review 和 approve PR。

**注意事项**

* 记得在必要时更新 [.gitignore](https://github.com/MonteCarloClub/Ethereum-Academy/blob/main/.gitignore)，不要把无关文件推送到远程仓库。请参考 [.gitignore 文档](https://git-scm.com/docs/gitignore)。
* 避免琐碎、非线性的提交，在必要时做 `rebase` 操作。请参考 [git rebase 文档](https://git-scm.com/docs/git-rebase)。建议新分支只领先主分支1个提交。
* 禁止直接向主分支推送更改！