# hugo blog
### 套用樣式
https://themes.gohugo.io/themes/gokarna/

```
git init
git submodule add https://github.com/526avijitgupta/gokarna.git themes/gokarna
echo theme = \"gokrana\" >> config.toml
```

### theme basic usage
https://gokarna-hugo.netlify.app/posts/theme-documentation-basics/#installation

### 新增文章
`hugo new posts/first_page.md`
新增完的文章會有基本格式
```
---
title: "First_post"
date: 2022-10-18T11:14:55+08:00
draft: true #草稿 要改成false才會deploy
---
```

### 在本地端開啟網頁
`hugo server -D`

### deploy
`hugo`


