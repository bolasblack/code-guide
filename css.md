# CSS 编码规范指南

* 两个空格缩进

* [使用 `* { box-sizing: border-box; }`](http://paulirish.com/2012/box-sizing-border-box-ftw/)

* 类名里使用连字符 `-` ，不要使用驼峰式或者下划线 `_`

* 尽量摒弃原子类，使用 CSS 预编译器的 extend 和 mixin 来替代
    * [Meta CSS —— 一个Anti Pattern的典型](http://hax.iteye.com/blog/497338)
    * [关于样式类（Style Class）](http://hax.iteye.com/blog/500015)
    * [再谈某些所谓CSS最佳实践](http://hax.iteye.com/blog/849826)

* 使用十六进制的色值（类似 `#fff` ）或者 RGBA 色值（类似 `rgba(100, 100, 100, .5)` ），替代颜色的名称（类似 `red` ）
    * [colors - Are there any good reasons for using hex over decimal for RGB colour values in CSS? - Stack Overflow](http://stackoverflow.com/questions/3230851/are-there-any-cons-to-using-color-names-in-place-of-color-codes-in-css)
    * [Are there any cons to using color names in place of color codes in CSS? - Stack Overflow](http://stackoverflow.com/questions/3230851/are-there-any-cons-to-using-color-names-in-place-of-color-codes-in-css)

* CSS 属性的名称和值能够缩写的时候就缩写
    * margin
    * padding
    * font
    * border
    * background
    * ...

* 使用如下的属性编写顺序

    ```css
    .item {
      /* 预编译工具的 mixin extend 变量 */

      /* 定位 */
      position: static;
      z-index: 0;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;

      /* 可视 */
      display: block;
      visibility: hidden;
      float: none;
      clear: none;
      overflow: hidden;
      clip: rect(0 0 0 0);

      /* 盒模型 */
      box-sizing: content-box;
      margin: 0;
      padding: 0;
      width: auto;
      min-width: 0;
      max-width: 0;
      height: auto;
      min-height: 0;
      max-height: 0;

      /* 布局 */
      table-layout: fixed;
      empty-cells: show;
      border-spacing: 0;
      border-collapse: collapse;
      list-style: none;

      /* 字体 */
      font: 1em sans-serif;
      font-family: Arial, sans-serif;
      font-size: 1em;
      font-weight: normal;
      font-style: normal;
      font-variant: normal;

      /* 排版 */
      text-align: left;
      vertical-align: top;
      line-height: 1;
      white-space: normal;
      text-decoration: none;
      text-indent: 1;
      text-transform: uppercase;
      letter-spacing: 1;
      word-spacing: normal;

      /* 修饰 */
      content: "";
      cursor: default;
      opacity: 1;
      color: #d00;
      text-shadow: 5px 5px 5px #d59;
      border: 1px solid #d00;
      border-radius: 15px;
      box-shadow: inset 1px 0 0 #fff;
      background: #fff url("../i/bg.png") no-repeat 0 0;
    }
    ```

