# GIT

- 强制删除文件

```shell
git filter-branch --force --index-filter 'git rm --cached -r --ignore-unmatch path-to-your-remove-file' --prune-empty --tag-name-filter cat -- --all

git push origin master -f
```

https://blog.51cto.com/phpervip/2497305

https://help.github.com/articles/remove-sensitive-data