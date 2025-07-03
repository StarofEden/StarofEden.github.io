---
title: butterfly主题博客美化记录
date: 2025.6.30
updated: 2025.7.2
description: 魔改butterfly时遇到的一些问题
swiper_index: 3 #置顶轮播图顺序，非负整数，数字越大越靠前
sticky: 1
categories: blog  
tags: 博客美化
---
> 前言🌟
>
> 美化教程主要参考：[Fomalhaut博客魔改教程](https://www.fomal.cc/posts/4aa2d85f.html)
>
# 文章页Post Front-matter
```markdown
---
title: 文章标题
date: 创建日期
updated: 更新日期
cover: 文章封面
description: 文章描述
swiper_index: 1 #置顶轮播图顺序，非负整数，数字越大越靠前
sticky: 1 #置顶
categories: blog  #分类
tags: 文章标签
---
```

# 更换字体未成功
下图划横线的代表应用的字体：

![image-20250630102717322](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202506301027409.png)

字体文件路径未配置正确：

![image-20250630103515926](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202506301035945.png)
custom.css:
![image-20250630103442769](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202506301034788.png)

# 黑夜霓虹灯与导航栏发光未成功配置：

```css
/* 深色模式下的导航栏文字阴影 */
[data-theme="dark"] {
  .menus_item a.site-page {
    text-shadow: 0 0 6px rgba(126, 165, 255, 0.7) !important; /* 主题色光晕 */
    transition: text-shadow 0.3s ease;
  }

  #page-name {
    text-shadow: 0 0 8px rgba(126, 165, 255, 0.7) !important;  /* 主题色霓虹光晕 */
    transition: text-shadow 0.3s ease;
  }
}
```

# 没有评论



# 右下角按钮不能改变颜色



![image-20250701095831024](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202507010958098.png)

blog\node_modules\hexo-theme-butterfly\source\css\_global

此文件可以设置各种全局颜色

通过f12找到要改变颜色的参数名



# 黑夜模式主题颜色

  C:\blog\node_modules\hexo-theme-butterfly\source\css\_mode\darkmode.styl

  添加：

  ```stylus
  if hexo-config('darkmode.enable') || hexo-config('display_mode') == 'dark'
    [data-theme='dark']
  +    --theme-color: #e98ca1
      --global-bg: darken(#121212, 2)
      --font-color: alpha(#FFFFFF, .7)
  ```

  

# 导航栏中间标题显示不居中

  custom.css：

  ```css
  /* 标题增强 */
  #site-name::before {
    opacity: 0;
    background-color: #e98ca1 !important;
    border-radius: 8px;
    -webkit-border-radius: 8px;
    -moz-border-radius: 8px;
    -ms-border-radius: 8px;
    -o-border-radius: 8px;
    transition: 0.3s;
    -webkit-transition: 0.3s;
    -moz-transition: 0.3s;
    -ms-transition: 0.3s;
    -o-transition: 0.3s;
    position: absolute;
    top: 0 !important;
    right: 0 !important;
    width: 100%;
    height: 100%;
    content: "\f015";
    box-shadow: 0 0 5px #e98ca1;
    font-family: "Font Awesome 6 Free";
    text-align: center;
    color: white;
    line-height: 30px; /*如果有溢出或者垂直不居中的现象微调一下这个参数*/
    font-size: 18px; /*根据个人喜好*/
  }
  ```

  将上面的`line-height: 30px;`适当调小一点

  同时：

  修改`top: 7px !important;`可以f12寻找最合适的数值

  ```css
  /* 修复滚动显示标题居中 */
  center#name-container {
    position: absolute !important;
    width: fit-content !important;
    left: 43% !important;   //调整左右
    top: 7px !important;    //调整高度
  }
  ```

# 公告设置
# 自定义图标


# 修改主题颜色--theme-color
## 方法一：
_config.butterfly.yml：
```YML
theme_color:
  enable: true
  main: "#e98ca1"   
  paginator: "#e98ca1"  # 主色调
  button_hover: "#e98ca1" #分页按钮颜色
  text_selection: "#86a4ff"  #按钮悬停色
  link_color: "#99a9bf"    #超链接颜色
  meta_color: "#858585"      #元信息文字色 作者、日期等元信息颜色）
  hr_color: "#A4D8FA"        #水平线颜色
  code_foreground: "#F47466"      #代码文字色
  code_background: "rgba(27, 31, 35, .05)"
  toc_color: "#e98ca1"        # 目录文字色
  blockquote_padding_color: "#49b1f5" # 引用框边框色
  blockquote_background_color: "#49b1f5"  # 引用框背景
  scrollbar_color: "#e98ca1" # 滚动条颜色
  meta_theme_color_light: "ffffff"  # 浅色模式主题色
  meta_theme_color_dark: "#7ec1f8"  # 深色模式主题色
```

## 方法二：

blog\node_modules\hexo-theme-butterfly\source\css\_global\index.styl

```stylus
--theme-color: #e98ca1 !important;
```

在后面加一个`！important；`能够覆盖其它设置

*其它颜色参数同上，基本都能在这个文件里找到，如果找不到就随便在一个styl文件里自己添加*

# 文章目录调整

* custom.css:

  ```css
  /* 调整目录栏宽度 */
  #aside-content #card-toc .toc-content {
    width: 320px !important;  /* 默认250px */
    max-width: 110% !important;
  
    /* 正文行高（默认1.5，建议1.2-1.4） */
    line-height: 1.6 !important;
  
    /* 标题行高（可选调整） */
    .toc-link {
      line-height: 1.6 !important;
      margin: 5px 0 !important;  /* 同时调小上下间距 */
    }
  }
  ```
  
  
  

# 小风车右边#井号没去除

在 `blog\node_modules\hexo-theme-butterfly\source\css\index.styl` 中添加：

```css
.markdownIt-Anchor {
  display: none !important;
}
```



# 黑夜模式遮罩问题

`C:\blog\node_modules\hexo-theme-butterfly\source\css\_layout\loading.styl`

```css
 /* 深色模式背景覆盖 */
    [data-theme="dark"] & {
      background:rgb(21, 21, 21) !important;  // 纯黑背景
      opacity: 0.5;
    }
```

