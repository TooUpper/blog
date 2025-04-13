# blog
## 克隆blog到本地

~~~shell
git clone git@github.com:TooUpper/blog.git
~~~

## 下载依赖包

项目根目录下执行：

~~~she
npm install hexo-cli -g // Linux 下需要以root权限运行
npm install
~~~

## 运行

~~~shell
hexo clean
hexo s
~~~

## 发布

```shell
hexo clean
hexo generate
hexo server //如果本地到localhost:4000预览博客效果没问题可省略
hexo deploy
===========================================================
// 目前改为通过 GitHub Actions 部署
// 通过在项目根目录创建 .github/workflows/deploy.yml 文件使其自动部署
// 每次 git push 推送后会自动进行部署
```

## 如果每次hexo d都自定义域名失效，则

在根目录下创建`source/CNAME`文件内容为

```shell
www.TooUpper.com // 自定义的域名
```

然后重新发布即可。

## 错误

**1.powershell 提示在此系统上禁止运行脚本**

**系统** -> **开发者选项** -> **PowerShell** -> 打开*更改执行策略，以允许本地PowerShell脚本在未签名的情况下运行*。远程脚本需要签名”。

**2.本地搜索报错`redirect error search/undefined`**

此问题原因是所用主题中的“search.js”文件获取 URL 时出现问题。

将所用主题的`search.js`文件中的`url: $("link", this).attr("href")`替换为`url: $( "url" , this).text()`即可。

**3.git clone git@github.com:TooUpper/blog.git 时克隆不了，即使网络是正常且挂了梯子**

这种情况一般出现在新配置 SSH 的设备中，注意看下你的 git Bash 是不是弹窗提示你验证当前账号，一般验证即可；
