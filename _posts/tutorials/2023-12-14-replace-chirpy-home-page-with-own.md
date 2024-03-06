---
title: Chirpy主题的个性化自定义之自定义首页
categories: [Blogging, Blog Tutorial]
tags: [tutorial]

toc: true

pin: false

img_path: "/assets/img/posts/replace-chirpy-home-page-with-own/"
render_with_liquid: false
---

## Motivation
做自定义首页的动机挺直白的，我不太喜欢首页直接贴博客的flow，而更希望是对这个个人网站的引导，所以我选择尝试自己修改一些样式来完成这个工作。<br />
最主要的参考来自[jekyll-theme-chirpy issue 711](https://github.com/cotes2020/jekyll-theme-chirpy/issues/711)和[jekyll-theme-chirpy issue 855](https://github.com/cotes2020/jekyll-theme-chirpy/issues/855)以及博客[Jinchao Li - Exchange Homepage and About](https://jekyll-theme-chirpy-taupe.vercel.app/blog/exchange-homepage-and-about/)，大体上一致，有些细节我优化了一下，有些细节我省略了。

<br />


## 支持新的tab
在替换之前，我们先使用新的tab来指向所有的Blog文章和详细列表页，我们称之为“博客”。由于[踩坑记录](#no0-jekyll的paginator在md文件中不生效)里所记，我们必须要在`index.html`里做文章：

1. 在根目录下新建`blogs`文件夹，并放一个`index.html`文件，这个文件影响生成的`_site/blogs/index.html`，而后者就是静态网站访问的对象：
    ```html
    ---
    layout: blogs
    title: Blogs
    # Index page
    ---
    ```
    {: file="_site/blogs/index.html"}
    其中，`layout`属性影响布局，相当于引用`_layouts/blogs.html`文件；`title`是主标题，和`_tabs`文件夹下的`title`是一个东西，通过文件`_data/locales/xx-xx.yml`来实现全局国际化：
    ```yml
    # The tabs of sidebar
    tabs:
    # format: <filename_without_extension>: <value>
        home: 首页
        blogs: 博客
    ```
    {: file="_data/locales/zh-CN.yml"}
2. 在`_config.xml`文件内指定新的可分页的路径，我们这用的是`/blogs/`，所以在`paginate`属性下添加：
    ```yml
    paginate: 5
    paginate_path: "/blogs/:num/"
    ```
    {: file="_config.yml"}
    `:num`表示具体页数。
3. 往sidebar里强制塞一个tab，在`_includes/sidebar.html`里增加点东西，让它看起来像这样：
    ```html
    <!-- home -->
    <li class="nav-item{% if page.layout == 'home' %}{{ " active" }}{% endif %}">
    <a href="{{ '/' | relative_url }}" class="nav-link">
        <i class="fa-fw fas fa-home"></i>
        <span>{{ site.data.locales[include.lang].tabs.home | upcase }}</span>
    </a>
    </li>

    <!-- blogs -->
    <li class="nav-item{% if page.url == '/blogs/' | relative_url %}{{ " active" }}{% endif %}">
    <a href="{{ '/blogs/' | relative_url }}" class="nav-link">
        <i class="fa-fw fa-solid fa-blog"></i>
        <span>{{ site.data.locales[include.lang].tabs.blogs | upcase }}</span>
    </a>
    </li>
    ```
    {: file="_includes/sidebar.html"}
    重点是blogs块里的`url`和`href`都要对上，`<span>`里用上了写死的国际化方法，这里会去读对应的文件，如果读不到就用`upcase`，这部分也很直白。
4. 为了topbar的美观，修改topbar相关的文件：
    ```html
    <nav id="breadcrumb" aria-label="Breadcrumb">
      {% assign paths = page.url | split: '/' %}
      <!-- {% if paths.size == 0 or page.layout == 'home' %} -->
      {% if paths.size == 0 %}
        <!-- home page -->
        <span>{{ site.data.locales[include.lang].tabs.home | capitalize }}</span>

      {% else %}
        {% for item in paths %}
          {% if forloop.first %}
            <span>
              <a href="{{ '/' | relative_url }}">
                {{ site.data.locales[include.lang].tabs.home | capitalize }}
              </a>
            </span>

          {% elsif forloop.last %}
            <!-- {% if page.collection == 'tabs' %} -->
            {% if page.collection == 'tabs' or item == 'blogs' %}
              <span>{{ site.data.locales[include.lang].tabs[item] | default: page.title }}</span>
            {% else %}
              <span>{{ page.title }}</span>
            {% endif %}
            <!-- {% elsif page.layout == 'category' or page.layout == 'tag' %} -->
            {% elsif page.layout == 'category' or page.layout == 'tag' or page.layout == 'home' %}
            <span>
              <a href="{{ item | relative_url }}">
                {{ site.data.locales[include.lang].tabs[item] | default: page.title }}
              </a>
            </span>
          {% endif %}
        {% endfor %}
      {% endif %}
    </nav>
    <!-- endof #breadcrumb -->
    ```
    {: file="_includes/topbar.html"}
    效果如下：<br />
    ![topbar](topbar.png)
5. 修改`_layouts/page.html`、`_layouts/head.html`、`_layouts/sidebar.html`和`_layouts/topbar.html`中的`page.layout == 'home'`替换成`page.url == '/' or page.url == site.baseurl`，这一般是样式突然变得奇怪的解法。

<br />


## 首页替换
为了新的首页，我们删掉了根目录下的`index.html`，为此，需要一个新的文件来顶替首页的渲染。我们根据教程采用了`permalink`+`markdown`的思路。
1. 在`_tabs`目录下新建一个`home.md`文件：
    ```markdown
    ---
    title: Hi there 👋
    order: 0

    permalink: /
    ---
    ```
    {: file="_tabs/home.md"}
    然后刷新就会发现sidebar上多出了一个，显然不是我们想要的效果：<br />
    ![bug-sidebar](bug-sidebar.png)
2. 修改`_includes/sidebar.html`以解决sidebar出现多余内容的问题：
    ```html
    <!-- the real tabs -->
    {% for tab in site.tabs %}
    {% if tab.url == '/' | relative_url %}{% continue %}{% endif %}
    <li class="nav-item{% if tab.url == page.url %}{{ " active" }}{% endif %}">
      <a href="{{ tab.url | relative_url }}" class="nav-link">
        <i class="fa-fw {{ tab.icon }}"></i>
        {% capture tab_name %}{{ tab.url | split: '/' }}{% endcapture %}

        <span>{{ site.data.locales[include.lang].tabs.[tab_name] | default: tab.title | upcase }}</span>
      </a>
    </li>
    <!-- .nav-item -->
    {% endfor %}
    ```
    {: file="_includes/sidebar.html"}
    现在正常了。😀

<br />


## 踩坑
这里记录了过程中的一些踩坑和细节，防止你我再次掉进坑里。🤕

### No.0 Jekyll的Paginator在md文件中不生效
> 这算是前人有总结的一个问题了，但是作死的我还是义无反顾地走了上去，然后华丽地踩坑了。😶
{: .prompt-warning}


#### 问题描述
Jekyll中有辅助翻页的`paginator`对象，内部存储了一些`page`、`per_page`等一些属性。一开始打算直接在`_tabs`目录下新建md文件然后引用`_layouts`里的布局直接解决的。但是使用md后发现一个问题，`paginator`变成空对象了，我翻页和填充元素还要用到它呢!😱
```html
<!-- Get pinned posts -->

{% assign offset = paginator.page | minus: 1 | times: paginator.per_page %}
{% assign pinned_num = pinned.size | minus: offset %}

{% if pinned_num > 0 %}
  {% for i in (offset..pinned.size) limit: pinned_num %}
    {% assign posts = posts | push: pinned[i] %}
  {% endfor %}
{% else %}
  {% assign pinned_num = 0 %}
{% endif %}

<!-- Get default posts -->

{% assign default_beg = offset | minus: pinned.size %}

{% if default_beg < 0 %}
  {% assign default_beg = 0 %}
{% endif %}

{% assign default_num = paginator.posts | size | minus: pinned_num %}
{% assign default_end = default_beg | plus: default_num | minus: 1 %}

{% if default_num > 0 %}
  {% for i in (default_beg..default_end) %}
    {% assign posts = posts | push: default[i] %}
  {% endfor %}
{% endif %}
```
{: file="_layouts/blogs.html // renamed from _layouts/home.html"}

#### 解决过程
没有解决过程，直接鸵鸟，因为我看到了Jekyll官网的教程[Pagination](https://jekyllrb.com/docs/pagination/)：
> **Pagination only works within HTML files** <br />
> Pagination does not work from within Markdown files from your Jekyll site. Pagination works when called from within the HTML file, named `index.html`, which optionally may reside in and produce pagination from within a subdirectory, via the `paginate_path` configuration value.
{: .prompt-info}
好了，直接跳过自我摧残环节，后面就直接参照[Jinchao Li - Exchange Homepage and About](https://jekyll-theme-chirpy-taupe.vercel.app/blog/exchange-homepage-and-about/)了。

<br />


### No.1 错误的底部导航栏链接

#### 问题描述
问题很简单，当我将原本的Home页下到第二个tab时，底部的导航栏静态链接出了问题（绿框框出来的部分）：<br />
![bug-wrong-page-one-nav](bug-wrong-page-one-nav.png)

#### 解决过程

显然是导航栏直接写死了`page_num`为1时的链接，那么我们只要修改一下对应的`href`就行，找一下对应的文件和代码：
```html
{% if paginator.total_pages > 1 %}
  {% include post-paginator.html %}
{% endif %}
```
{: file="_layouts/blogs.html // renamed from _layouts/home.html"}

```html
<!-- show number -->
<li class="page-item {% if i == paginator.page %} active{% endif %}">
    <a
    class="page-link"
    href="{% if i > 1 %}{{ site.paginate_path | replace: ':num', i | relative_url }}{% else %}{{ '/' | relative_url }}{% endif %}"
    >
    {{- i -}}
    </a>
</li>
```
{: file="_layouts/blogs.html"}
定位到显示部分发现，当`i > 1`时，去替换`paginate_path`里`:num`占位符，否则，也就是`i == 1`的情况，直接用`'/' | relative_url`的拼接来生成链接。所以，这里只要改成`'/blogs' | relative_url`就行了。

> 修改文件时一帆风顺，到了写文档的时候出了点问题，我的Jekyll Liquid怎么没办法在文档中展示了？直到我看到了这个报错：
> ```shell
>   Liquid Exception: Liquid syntax error (line 55): 'for' tag was never closed in /home/gc/repo/GiannisChen.github.io/_posts/HowTo/2023-12-14-replace-chirpy-home-page-with-own.md
             Error: Liquid syntax error (line 55): 'for' tag was never closed
             Error: Run jekyll build --trace for more information.
> ```
> 突然想起了新版[write-a-new-post](https://chirpy.cotes.page/posts/write-a-new-post/)中的一句话：<br />
> **Or adding `render_with_liquid: false` (Requires Jekyll 4.0 or higher) to the post’s YAML block.** <br />
> 那就没事了，恰好我有高版本的Jekyll。🥵
{: .prompt-danger}

最后的效果：<br />
![bugfix-wrong-page-one-nav](bugfix-wrong-page-one-nav.png)