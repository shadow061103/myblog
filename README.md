# hugo blog
### 套用樣式
https://themes.gohugo.io/themes/gokarna/

```
git init
git submodule add https://github.com/526avijitgupta/gokarna.git themes/gokarna
echo theme = \"gokrana\" >> config.toml
```

### theme usage doc
- [basic](https://gokarna-hugo.netlify.app/posts/theme-documentation-basics/#installation)
- [advance](https://gokarna-hugo.netlify.app/posts/theme-documentation-advanced/)

- 首頁icon
會去找`static/icons`資料夾下的圖片對應config的socialIcons配置
icon需要用svg格式 可在[這裡](https://www.svgrepo.com/vectors/blog/1)下載

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
可以在 http://localhost:1313/ 查看
### 部屬到github page
- `hugo` 在public生成網站檔案
- 在github上建立一個新的repository，名稱使用`<account>.github.io`
- 一般版控流程
```
cd public
git init 
git add .
git commit -m '123'
git remote add origin https://github.com/<account>/<account>.github.io.git
git push -u origin master
```

