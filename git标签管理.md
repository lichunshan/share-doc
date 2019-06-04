# git标签管理

**标签操作**

git tag   查看所有标签

git tag name   创建标签

git tag -a name -m "comment"   创建标签同时追加提交信息

git tag -d name    删除标签

git push origin name 将标签发布到远程

git push origin :refs/tags/name  删除远程标签

***

**通过标签回滚**

git show name  查看某个标签的详情

commit  fb479960c0cec5549463ae123d70bdd72ccf6be7

通过commit id 回退

git reset --hard  fb479960c0cec5549463ae123d70bdd72ccf6be7

