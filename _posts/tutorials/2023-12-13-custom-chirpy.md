---
title: Chirpy主题的个性化自定义旅程
categories: [Blogging, Blog Tutorial]
tags: [tutorial]

toc: true

pin: false

render_with_liquid: false
---

> 这篇博客将记录我在改造基于Chirpy的个人博客时遇到的一些个性化相关问题的解决思路和方法，以及一些参考链接。👇
{:.prompt-info}

<br />


## 自定义SideBar标签

1. 检查`_tabs`文件内的`.md`文件信息，定位到`icon`属性：
    ```markdown
    ---
    layout: archives
    icon: fas fa-archive
    order: 3
    ---
    ```
2. 前往[fontawesome](https://fontawesome.com/)找一个心仪的icon复制即可，例如我找了一个“书架”相关的icon作为我新的SideBar元素的icon：<i class="fa-solid fa-book"></i>

<br />


## 除去「最近更新」和「热门标签」侧边栏
为了美观（折腾），我选择去除了侧边栏中的「最近更新」和「热门标签」两个属性，需要去除的内容很简单，找到`_layouts/default.html`中的`panel`部分：
```html
<!-- panel -->
<aside aria-label="Panel" id="panel-wrapper" class="col-xl-3 ps-2 mb-5 text-muted">
<div class="access">
    {% include_cached update-list.html lang=lang %}
    {% include_cached trending-tags.html lang=lang %}
</div>
```
修改为：
```html
<!-- panel -->
<aside aria-label="Panel" id="panel-wrapper" class="col-xl-3 ps-2 mb-5 text-muted">
<div class="access"></div>
```
由于去掉这个`div`会影响页面布局，所以我的做法是暂时只去掉了内容，保留了占位的`div`。

<br />


## 书架页的设计和流程

为了更好地进行赛博碎碎念，我加上了[书架](/bookshelf)和[游戏](/games)两个tab，其中游戏tab就是一个`markdown`文件，而书架页则多了Ruby的内容。由于我不会Ruby，所以只是朴素地照猫画虎罢了。

1. 创建tab页：
    ```markdown
    ---
    icon: fa-solid fa-book
    order: 4
    ---

    > **Hello!**🎉 <br>
    >  
    > 📕 This is my bookshelf, which contains my reading list and notes.
    {: .prompt-tip}

    ## Reading List

    |       | Title | Author | Tags  | Description |
    | :---: | :---: | :----: | :---: | :---------: |
    |       |  xxx  |  xxx   |  xxx  |     xxx     |


    ## Reading Notes

    - `Suspense`（悬疑）👻
    - `Whodunit`（推理）🕵️‍
    - `Unknown`（待定）❓

    {% assign sorted = site.notes | sort: 'title' %}           
    {% for note in sorted %}{% unless note.omit == true %}| [《{{ note.title }}》 - {{note.subtitle}}]({{ note.url | relative_url }})  | {{ note.author }} | {{note.tag}} | 
    {% endunless %}{% endfor %}
    ```
    {: file="_tabs/bookshelf.md"}
    上半部分是很正常的`markdown`，下半部分是从`_layouts/home.html`里抄过来的（虽然我这里只有改名以后的`_layouts/blogs.html`了），表示读出`notes`中的所有帖子，然后按照排序展示，中间的`|`能画出一个表格。

2. 但是只做这些是没法生效的，因为`site.notes` 是个空对象，编译的时候就会报`null object`，但似乎只有触发`sort: 'title'`的时候才会，`for-loop`并不会引发报错？神奇的Ruby，个中原理后面再看吧。话头回到生效上，看了一下[Natalie](https://github.com/some-natalie)这位大佬[博客](https://some-natalie.dev/recipes/)某一页的源码，选择在`_config.yml`下进行一番操作：
    ```yml
    collections:
        ...
        notes:
            output: true
            sort_by: name

    defaults:
    ...
    # reading-notes
    - scope:
        path: "_notes"
        type: notes
        values:
        layout: reading-note
        permalink: /reading-notes/:title/
    ```
    {: file="_config.yml"}
    添加了`collections.notes`使得`site.notes`里非空，`default.scope`感觉是让对应`markdown`能够展示，即md文件被编译成`index.html`。

3. 这样其实就搞定了，基本功能就有了。

<br />


## 简历页面

[简历页面](/resume)是我花时间最多的一个页面了，为了好看最后几乎是用纯html写的，现在看上去好多了，纯markdown毕竟有一点限制在。也懒得写了，直接看`_tabs/resume.md`去。

### 更新1
我重新做了一个`contact`的样式，上一版本长这样：<br />

<i class="fa-solid fa-location-dot" style="font-size: 95%; margin-left: 0.1rem; margin-right: 0.59rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">Wuxi - Jiangsu - China</span>

<i class="fa-solid fa-envelope" style="font-size: 95%; margin-right: 0.5rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">giannischen@nuaa.edu.cn</span>

<i class="fa-brands fa-github" style="font-size: 95%; margin-left: 0.05rem; margin-right: 0.47rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">https://github.com/GiannisChen</span>
<br />
源代码长这样：
```html
<i class="fa-solid fa-location-dot" style="font-size: 95%; margin-left: 0.1rem; margin-right: 0.59rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">Wuxi - Jiangsu - China</span>

<i class="fa-solid fa-envelope" style="font-size: 95%; margin-right: 0.5rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">giannischen@nuaa.edu.cn</span>

<i class="fa-brands fa-github" style="font-size: 95%; margin-left: 0.05rem; margin-right: 0.47rem;"></i>  <span style="font-size: 90%; letter-spacing: 0.2px;">https://github.com/GiannisChen</span>
```
但是总感觉怪怪的，于是乎有了新版，现在长这样：
<ul class="fa-ul">
<li><span class="fa-li"><i class="fa-solid fa-location-dot"></i></span>Wuxi - Jiangsu - China</li>
<li><span class="fa-li"><i class="fa-solid fa-envelope"></i></span>giannischen@nuaa.edu.cn</li>
<li><span class="fa-li"><i class="fa-brands fa-github"></i></span>https://github.com/GiannisChen</li>
</ul>

源代码长这样：
```html
<ul class="fa-ul">
<li><span class="fa-li"><i class="fa-solid fa-location-dot"></i></span>Wuxi - Jiangsu - China</li>
<li><span class="fa-li"><i class="fa-solid fa-envelope"></i></span>giannischen@nuaa.edu.cn</li>
<li><span class="fa-li"><i class="fa-brands fa-github"></i></span>https://github.com/GiannisChen</li>
</ul>
```
完全依赖于[Font Awesome](https://fontawesome.com/docs/web/style/lists)提供的能力，至少方便很多，完全不用手动去做上下对齐了（显然有更加优雅的办法，但是不想弄了💤）。

<br />


## 自定义首页

自定义首页踩的坑和参考内容值得另开一个博客写了，于是，这部分的内容我放在了这篇博客里，想看的请移步：[Chirpy主题的个性化自定义之自定义首页](../replace-chirpy-home-page-with-own)。🧐

<br />


## 首页`Contact`部分的构造

看到Font Awesome提供的[Icons in a List](https://fontawesome.com/docs/web/style/lists)功能，就想着我能不能做一个***SVG in a List***？正好有这么一个栏可以来尝试一下这项技术，参考了这个[stack-overflow的问答](https://stackoverflow.com/questions/13354578/custom-li-list-style-with-font-awesome-icon)和[How to add SVG icon as list item bullets](https://brickslabs.com/how-to-add-svg-icon-as-list-item-bullets/)这篇教程，效果展示在了[Home - Where to find me ?](/#where-to-find-me-)里。
```html
<style>
    ul.font-gc {
        list-style-type: none;
        margin-left: 10px;
    }
    ul.font-gc li {
        margin-bottom: 12px;
        margin-left: -10px;
        display: flex;
        align-items: center;
    }
    ul.font-gc li::before {
        color: transparent;
        font-size: 1px;
        content: " ";
        margin-left: -1.3em;
        margin-right: 15px;
        padding: 10px;
        background-repeat: no-repeat;
        background-image: url("/assets/img/icons/icon-default.svg");
    }

    ul.font-gc li.leetcode::before {
        background-image: url("/assets/img/icons/leetcode.svg");
    }

    ul.font-gc li.github::before {
        background-image: url("/assets/img/icons/github.svg");
    }

    ul.font-gc li.bilibili::before {
        background-image: url("/assets/img/icons/bilibili.svg");
    }

    ul.font-gc li.steam::before {
        background-image: url("/assets/img/icons/steam.svg");
    }
</style>

<ul class="font-gc">
    <li class="leetcode"><a href="https://leetcode.cn/u/yuan-mou-zhi-li-ren/">GiannisChen</a></li>
    <li class="github"><a href="https://github.com/GiannisChen">GiannisChen</a></li>
    <li class="bilibili"><a href="https://space.bilibili.com/23284023">源谋直立人</a></li>
    <li class="steam"><a href="https://steamcommunity.com/profiles/76561198313603277/">源某直立人</a></li>
</ul>
```
{: file="_tabs/home.md"}

1. 首先定义出`ul`的样式，为了不污染其他的`ul`，我们给他一个`class`，称之为`font-gc`，意味着GiannisChen's kind of fonts（有点臭不要脸😛）。
2. 接着定义出其他的子样式，比如`ul.font-gc li`和`ul.font-gc li::before`，`ul.font-gc li::before`内定义默认的样式，这样其他`li`引入的`class`就不用加上这些属性描述了，只要修改`background-image`就行。
3. 从网站上扒一点svg文件下来，这里推荐国内的开源图标库[iconfont-阿里巴巴矢量图标库](https://www.iconfont.cn/)，除了潜在的侵权风险外，比Font Awesome感觉好用多了。
4. 自定义图标对应的`class`，就像这样：
    ```css
    ul.font-gc li.steam::before {
        background-image: url("/assets/img/icons/steam.svg");
    }
    ```
5. 最后，去预览效果吧！

> ***坑来了！***<br />
> iconfont上下载的svg默认是有长宽限制的，比如这样：
> ```html
> <svg width="200" height="200">
> ```
> 这个就会影响展示，图太大了真实展示的就只有左上角了。要**解决这种问题**也很简单，直接把`width`属性和`height`属性删掉就行。🎉
{: .prompt-warning}

<br />


## BUGFIX

### 简历工作经历部分显示重叠问题

[简历部分](/resume)出现了靠左地文字和靠右地文字重叠在了一起。
<br />

#### 问题解决

究其原因是直接用了写死的布局，当手机等窄屏显示的时候，无法自动浮动布局，所以解决方法就是直接使用flex布局。
```html
<style>
.flex-container {
    display: flex;
    flex-ditrction: column;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: baseline;
}

.flex-container > div {
	margin: 10px;
	padding: 0px;
}
</style>

<div class="flex-container"><!-- .element: style="display: flex; flex-direction: row;" -->
    <div style="align-self: center">
        <svg>...</svg>
    </div> <!-- .element: style="margin: 10px; padding: 20px;"-->
    <div style="flex-grow: 1;">
        <p style="text-align: left;">
            <b><i><font size=4>Idlefish - Alibaba Inc.</font> </i></b> <br />
            <i >Hangzhou China</i> 
        </p>
    </div>
    <div>
        <i style="float: right;">Java Software Engineer Intern</i> <br />
        <i style="float: right;">May. 2023 - Sept. 2023</i> <br />
    </div>
</div>
```
最后可以实现经典的三栏布局。其中，中间栏的`style="flex-grow: 1;"`能够让中间栏自适应宽度。

#### 参考资料
- [Flex 布局教程：语法篇](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [三栏布局](https://www.jianshu.com/p/d485f2a26a17)
