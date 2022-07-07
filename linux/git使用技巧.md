# git 使用技巧

* 解决golang中私有仓库依赖https方式无法拉去问题

```
git config --global --add url."git@gitlab.33.cn:".insteadOf "https://gitlab.33.cn/"

```

[github加速器](https://github.com/dotnetcore/FastGithub)


* 设置http/https代理

```
git config --global https.proxy http://127.0.0.1:38457

git config --global https.proxy http://127.0.0.1:38457

```

* 取消代理

```
git config --global --unset http.proxy

git config --global --unset https.proxy
```

* 查看配置信息：

```
git config -l --global

```
