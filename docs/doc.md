title: Pizicai's Tech Blog
description: 个人技术博客
theme: jekyll-theme-minimal
markdown: kramdown
plugins:
  - jekyll-feed
  - jekyll-seo-tag
```

2. 创建默认布局文件:

````html
// filepath: d:\Person\Projects\pizicaiman.github.io\_layouts\default.html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ page.title }} - {{ site.title }}</title>
    <link rel="stylesheet" href="/assets/css/style.css">
</head>
<body>
    <header>
        <h1><a href="/">{{ site.title }}</a></h1>
        <nav>
            <a href="/">首页</a>
            <a href="/blog">博客</a>
            <a href="/about">关于</a>
        </nav>
    </header>

    <main>
        {{ content }}
    </main>

    <footer>
        <p>&copy; {{ site.time | date: '%Y' }} {{ site.title }}</p>
    </footer>
</body>
</html>