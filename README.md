# blog
## 克隆blog到本地

~~~shell
git clone git@github.com:it-kay/blog.git
~~~

## 下载依赖包

项目根目录下执行：

~~~she
npm install hexo-cli -g
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
