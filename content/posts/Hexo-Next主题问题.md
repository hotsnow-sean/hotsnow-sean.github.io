---
title: Hexo-Next主题 可能遇到的问题解决方案（持续更新）
slug: hexo-next-theme-issue
tags: [博客, Hexo, next-theme, Helper]
categories: 云的旅途
date: 2020-07-10T09:25:22+08:00
lastmod: 2020-07-14T20:29:31+08:00
---

![](https://cdn.jsdelivr.net/gh/hotsnow-sean/image-host/img/20200710100226.png)

<!--more-->

{{< note type="danger" >}}
注意本文更新时间，部分内容可能已经不适用
{{< /note >}}

## Next 主题版本问题

&emsp;&emsp;现在百度到的 next 官网写的比较旧，而且截至今日，这个主题已经在 GitHub 上开辟了新项目，原来的项目已经很久不更新了，新旧网址也比较像，这里推荐新网址：https://github.com/next-theme/hexo-theme-next ，根据其更新信息来获取最稳的 Next 主题以及配置方法。

## 主题更新导致重新配置问题 🔧

&emsp;&emsp;这个问题的基本解决办法见[hello word 这篇文章的分离主题配置部分]({{< relref "/posts/hello-world" >}})，这里对其中提到的 Next 主题的 `custom_file_path` 配置项中的部分内容作如下说明:

| 子项目名称  | 文件路径（相对根目录，可改）    | 作用                                                  |
| ----------- | ------------------------------- | ----------------------------------------------------- |
| postBodyEnd | source/\_data/post-body-end.njk | 会被添加到文章结束尾部（[用例](#eg2-end-flag)）       |
| bodyEnd     | source/\_data/body-end.njk      | 整个页面的底部                                        |
| variable    | source/\_data/variables.styl    | 主题中 \_variables 下的所有相关样式都可以在此覆盖修改 |
| style       | source/\_data/styles.styl       | 用于一般样式的修改添加（[用例](#eg1)）                |

&emsp;&emsp;除了配置中这些，`_data`中还可以加入 `next.yml` 用于主题配置，这样主题配置就可以和主题文件本身分离了。需要用到哪个文件就创建并取消注释，用好这种方式基本可以实现大部分想要的功能。

## 数学公式渲染问题 📕

&emsp;&emsp;经过尝试，当前版本（2020-7-10）可以不更换渲染器，只需把主题配置文件中的 `mathjax` 选项置为 `true`，并且在需要渲染公式的页面头部信息中写入 `mathjax: true` 即可，只要注意 markdown 文档的写法规范，一般都没什么问题。  
&emsp;&emsp;至于网上推荐的各种渲染器或多或少都有可能引发莫名其妙的问题，而且好几个渲染器已经基本弃坑，要注意甄别。目前 Next 针对数学渲染建议的渲染器是 `hexo-renderer-pandoc`，不过这个需要提前安装好 `pandoc`，而且我尝试的时候报错了，因此放弃。

## emoji 表情渲染问题 😀

&emsp;&emsp;首先当然可以选择常用的办法，更换可以使用 emoji 渲染插件的渲染器，例如：`hexo-renderer-markdown-it`，不过更换渲染器可能会引起文章其它部分的渲染问题，不推荐。
&emsp;&emsp;其次，经过尝试确认可以摒弃在 markdown 中打入表情代码的方式，而是直接打入表情本身，例如 win10 自带输入法中的表情，基本都可以渲染。  
&emsp;&emsp;重点，emoji 表情中有一系列蓝底白字数字的表情，这部分表情在部分浏览器尤其是手机端会呈现出一种乱码的状态，这个问题无法解决，或者说它可能本身就不算个问题，因为我们通过百度搜 [emoji 表情大全](https://apps.timwhitlock.info/emoji/tables/unicode)，可以发现这些表情的 `Native` 版本本身就长成这个鬼样子。。。所以，尽量不要使用它们！

## 手机端长文后面的内容看不见

&emsp;&emsp;这个问题经过测试，只会在手机端或者手机模拟器出现，文章内容仿佛突然被截断，但是你依旧可以选中文字进行复制等操作。这表明内容实际上是存在的，只是我们看不见，经过探索发现是动画导致的。  
&emsp;&emsp;Next 主题默认开启了页面各部分的动画，位于主题配置文件的如下内容：

```yml
motion:
  enable: true
  async: false
  transition:
    # All available transition variants: https://theme-next.js.org/animate/
    post_block: fadeIn
    post_header: fadeInDown
    post_body: fadeInDown
    coll_header: fadeInLeft
    # Only for Pisces | Gemini.
    sidebar: fadeInUp
```

&emsp;&emsp;既然只有长文会出现这个问题，再联系 `animate动画库` [官方](https://animate.style/)的说明有这样一句话：`Don't animate large elements`，可以大胆猜测是由于元素过高或者元素高度加载不及时导致的动画执行不完全问题。
💡 解决方案一：改变或者去掉上面配置中 `post_body` 的动画，直接留空即可
<a name="eg1">💡 解决方案二</a>：通过媒体查询，将手机端竖屏状态下的动画留空或改动。这种办法可以使得一般情况下动画效果依旧不变，只针对有问题的情况做出调整。如下：

```styl
// 写到 'source/_data/styles.styl' 中即可
@media only screen and (orientation: portrait) {
  .post-body {
    animation: fadeIn !important;
  }
}
```

&emsp;&emsp;之所以匹配竖屏是因为在测试中手机横屏不会出现这个问题，这里的查询条件可根据需要调整，要改变成的动画也随意调整到没问题即可，实在不行就置为 `none` 关闭动画。

## 在文章末尾添加结束标识 🔚 {#eg2-end-flag}

&emsp;&emsp;网上有很多解答，但都大同小异而且太旧，用的方法里还都是 `.swig` 文件，而 Next 早就不用这种格式了。。这里可以采用之前提到的 `custom_file_path` 的方式进行添加，例如：

```njk
{### 写到 'source/_data/post-body-end.njk' 中，并且取消相应主题配置中的注释 ###}
{### 样式内容随便改，只要在这个if里就可以 ###}
<div>
    {%- if not is_index -%}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束 <i class="fa fa-paw"></i> 感谢您的阅读-------------</div>
    {%- endif -%}
</div>
```

效果见本文末尾，哈哈

## Logo 分离

&emsp;&emsp;由于已经采用了主题文件与其配置分离的方法，当然希望图片等资源也不要放在主题文件夹下，经过测试直接在项目根目录的 `source` 文件夹里创建 `images` 文件夹，然后把 logo 丢进去，这样配置路径本身就和默认一样即可，图片名改成自己的就好。

## 流程图、序列图等渲染

&emsp;&emsp;Next 本身支持 markdown 图表展示，依赖于 mermaid，因此需要安装相关插件，推荐安装 [hexo-filter-mermaid-diagrams](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)，运行 `npm i hexo-filter-mermaid-diagrams --save`即可。安装完成之后在 Next 主题的配置文件中将 mermaid 开启就可以显示图表了。

&emsp;&emsp;**_问题：_** 这个问题会在同时启用了 pjax 时存在，由于 pjax 的 js 加载特性，导致页面只有在第一次加载或者刷新时图表才能正确显示。具体原理还未探究，但是目前版本（NexT version 8.0.0-rc.4）可用的解决办法如下：

- 打开 `themes\next\layout\_third-party\tags\mermaid.njk` 文件，在 `mermaid.initialize()` 语句之后加一句 `mermaid.contentLoaded()` 便可以解决这个问题。

_待续......_
