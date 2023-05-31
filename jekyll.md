# Jekyll

Jekyll 的工作方式是，服务器端（对于 github pages，就是 github 的服务器）将特定文件夹下（根目录，_posts）的文件编译成静态文件，我们只需要在该文件夹下写 md 文件就好了，无需生成 html。

# 本地预览

1. 确保Ruby已经安装 [Mac安装Ruby](https://developer.aliyun.com/article/459167)
2. 确保bundler已经安装：sudo gem install bundler
3. 确保Jekyll已经安装，在项目目录下bundle
4. 在项目目录下执行 bundle exec jekyll serve
5. 在浏览器中可通过URL [http://localhost:4000/](http://localhost:4000/) 看到你的网站


# 目录结构
一个基本的 Jekyll 站点目录结构：

|  文件（包括目录）   | 用途   |
|  ----  | ----  |
| _config.yml  | 保存配置信息 |
| _drafts  | 用于存放未发表博文的目录，其中博文文件名形如标题.后缀（不加日期），可以用bundle exec jekyll serve --drafts预览 |
| _includes | 用于存放可重用组件（如页眉、页脚、边栏、作者介绍）的目录，其中名如文件名的文件可以在布局或博文中引用 |
| _layouts | 用于存放布局模板的目录，如果一个博文或布局模板的导言指定了layout: 模板，则会通过把内容代入这目录下的模板.html中的{{ content }}来生成页面 | 
| _posts | 用来存放你的博文的目录，其中博文文件名形如年-月-日-标题.后缀 | 
| _data | 用来存放站点的结构化数据的目录，可以是名如数据.yml、数据.yaml、数据.json或数据.csv的文件，这些数据可通过site.data.数据访问 | 
| _sass | 用来存放后缀为scss的CSS文件片段，它们将被用（通过在用户样式表中@import "{{ site.theme }}";之类引用）于生成最终的样式表 | 
| _site | 用来存放本地预览生成的目录，应该在.gitignore文件忽略它。 | 
| .jekyll-metadata | 用来帮助本地预览跟踪文件修改，应该在.gitignore文件忽略它。 | 
| 其它后缀为.html、.markdown、.md或.textile的文件 | 有YAML导言的话会被Jekyll处理并在生成站点的对应路径生成.html文件 | 	
| 其它 |	会被原样复制到生成的站点，如样式表、图像、站点图标favicon.ico |

## 使用data
在_data文件夹下创建一个member.yml
```
- name: Zhang 3
- name: Li 4
- name: Wang 5
```
在页面中使用
```html
{% for member in site.data.member %}
<ul>
  <li>{{ member.name }}</li>
</ul>
{% endfor %}
```




- - -
# jekyll使用技巧

### 数学公式

要支持LaTeX数学公式，可以加入使用MathJax：
```html
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$']]}});
</script>
<script type="text/javascript" async src="https://cdn.jsdelivr.net/npm/mathjax@2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```

### Jekyll插件

#### 安装方法

- 在Gemfile中加入一行类似gem 'jekyll-插件'
- bundle install
- 在_config.yml中在plugins列表中加入jekyll-插件

#### 常用插件

* jekyll-sitemap可自动生成sitemap.xml和robots.txt，有利于帮助搜索引擎发现你的页面。不想被收录的页面可以把sitemap变量设为false。
* jekyll-seo-tag提供标签{% seo %}以便生成SEO需要的标签
* jekyll-paginate用于分页
  
</br>

> [更多jekyll使用技巧](https://crispgm.com/page/48-tips-for-jekyll-you-should-know.html)