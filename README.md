# [Jow Blog](https://wofead.top "Jow's Blog")

## 致谢
站在巨人的肩膀上，所以十分感谢的模板的提供者 [Hux](https://github.com/Huxpro/huxpro.github.io)
还有就是博客的完善还参照了[qiubaiying](https://github.com/qiubaiying/qiubaiying.github.io)
在说明的开头表示对前辈们的感谢
还有感谢 Jekyll、Github Pages 和 Bootstrap!


### [查看我的博客](https://wofead.top)
关于博客的搭建可以参照[qiubaiying](https://github.com/qiubaiying/qiubaiying.github.io)的[这篇文章](http://www.jianshu.com/p/e68fba58f75c '博客的搭建')，这里就不再赘述了

### 环境

如果你安装了 [jekyll](http://jekyllcn.com/)，那你只需要在命令行输入`jekyll serve` 或 `jekyll s`就能在本地浏览器中输入`http://127.0.0.1:4000/`预览主题，对主题的修改也能实时展示（需要强刷浏览器）。

### 开始
你可以通用修改 `_config.yml`文件来轻松的开始搭建自己的博客。
Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址在这里：[Jekyll - Official Site](http://jekyllrb.com/) 中文版的在这里：[Jekyll中文](http://jekyllcn.com/).

### 撰写博文

要发表的文章一般以 **Markdown** 的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml 头文件长这样:

```
---
layout:     post
title:      第一篇博客
subtitle:   万事开头难
date:       2019-3-13
author:     BY 
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true #是否开启catalog
tags:
    - Life
    - All
---

```
## Attention

> Note：
> 可以使用 `jekyll -s` 命令在本地实时配置博客，提高效率。详见 [Jekyll.com](http://jekyllcn.com/)

参考文档：[using jekyll with pages](https://help.github.com/articles/using-jekyll-with-pages/) & [Upgrading from 2.x to 3.x](http://jekyllrb.com/docs/upgrading/2-to-3/)

> 上传的图片最好先压缩，这里推荐 imageOptim 图片压缩软件，让你的博客起飞。

### SEO Title

我的博客标题是 **“Cute Blog”** 但是我想要在搜索的时候显示 **“Jow的博客 | BY Blog”** ，这个就需要 SEO Title 来定义了。

其实这个 SEO Title 就是定义了<head><title>标题</title></head>这个里面的东西和多说分享的标题，你可以自行修改的。


## License

遵循 MIT 许可证。有关详细,请参阅 [LICENSE](https://github.com/qiubaiying/qiubaiying.github.io/blob/master/LICENSE)。