---
title:  HTML - learning notes
categories:
  - Front end
tags:
  - HTML
  - Learning notes
---
Content

{% include toc %}

# Getting started with html

```html
<!--What specification to use -->
<!DOCTYPE html>

<html lang="en">

<!--head represents the header of the webpage-->
<head>
<!-- meta descriptive tag, used to describe some information of the website-->
<meta charset="UTF-8">
<meta name="keywords" content="Front-end learning">
<meta name="description" content="Build a website yourself">
<!--title represents the title of the webpage-->
<title>My first webpage</title>
</head>
<body>
My first webpage
</body>
</html>
```

# Basic tags

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Basic tag</title>
</head>
<body>

<!--Title tag-->
<h1>First level tag</h1>
<h2>Second level</h2>
<h3>Third level</h3>

<!--Paragraph tag-->
<p>Two tigers</p>
<p>paodekuai</p>
<p>uiozhineiadfhjio</p>

<!--Horizontal line tag-->
<hr>
<!--Line break tag-->
liangzhi<br/>
asdfhaoisfh<br/>
sdafdsafwterewyrte<br/>

<!--Bold, italic-->
<h1>Font tag style</h1>
Bold: <strong>i love you</strong><br/>
Italic: <em>i love you</em><br/>

<!--Special symbols-->
Space: empty&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<br/>
Greater than: &gt;
<br/>
Less than: &lt;
<br/>
Copyright: &copy;Copyright jay
<br/>
&gamma;

</body>

</html>
```

# Image tag

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Image tag learning</title>
</head>
<body>
<!-- src: Image path, can be relative path or absolute path-->
<img src="../resource/image/img.png" alt="Minions" title="Hover text" width="300" height="300" >

</body>
</html>
```

# Link tag

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Link tag</title>
</head>
<body>

<a name="top">Top</a>

<!-- a tag
href: required, indicating which page to jump to
target: indicates where the window is opened, _blank opens in a new tab, _self opens in this tab
-->

<a href="1.初识html.html" target="_blank">Click me to jump to the page-</a>

<a href="http://www.baidu.com">Click me to jump to the page-</a>

<br>
<!--Click the image to jump to the webpage-->
<a href="1.初识html.html">
<img src="../resource/image/img.png" alt="Minions" title="Hover text" width="1019" height="1024" >
</a>
<br>
<!--Anchor link
1. An anchor tag is required
2. Jump to the tag, for example, jump to the top of the tag
#
-->
<a href="#top">Back to the top</a>
<br>
<!--Functional link
Mail link: mailto
QQ link, you can go to the "QQ Promotion" official website to generate a statement
-->
<a href="mailto:chenjienudt@163.com">Click to contact me</a>
<br>

<a target="_blank" href="http://wpa.qq.com/msgrd?v=3&uin=862039735&site=qq&menu=yes"><img border="0" src="http://wpa.qq.com/pa?p=2:862039735:53" alt="Click here to send me a message" title="Click here to send me a message"/></a>
<br>
</body>
</html>
```

# List

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>List Learning</title>
</head>
<body>

<!--Ordered List-->
<ol>
<li>Java</li>
<li>python</li>
<li>C++</li>
</ol>

<hr>

<!--Unordered List-->
<ul>
<li>Java</li>
<li>python</li>
<li>C++</li>
</ul>

<hr>

<!--Custom List
dl: Tags
dt: list name
dd: list content

Usually used at the bottom of company websites
-->
<dl>
<dt>language</dt>

<dd>Java</dd>
<dd>python</dd>
<dd>C++</dd>
</dl>

</body>
</html>
```

# Table

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Table tag</title>
</head>
<body>
<!--Table
Row tr
Column td
-->

<table border="1px">
<tr>
<!-- Cross-column colspan-->
<td colspan="3">1-1</td>
</tr>
<tr>
<!-- Cross-row rowspan-->
<td rowspan="2">2-1</td>
 <td>2-2</td>
 <td>2-3</td>
 </tr>
 <tr>
 <td>3-2</td>
 <td>3-3</td>
 </tr>
</table>

</body>
</html>
```
