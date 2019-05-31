[markdown github基础语法](https://guides.github.com/features/mastering-markdown/)

[Basic writing and formatting syntax](https://help.github.com/en/articles/basic-writing-and-formatting-syntax)

# 一、Syntax guide
## 1.1.Headers
# This is an <h1> tag
## This is an <h2> tag
###### This is an <h6> tag

## 1.2.Emphasis
*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

_You **can** combine them_

## 1.3.Lists
### 1.3.1 Unordered
* Item 1
* Item 2
  * Item 2a
  * Item 2b
### 1.3.2 Ordered
1. Item 1
1. Item 2
1. Item 3
   1. Item 3a
   1. Item 3b
## 1.4.Images
![GitHub Logo](/images/logo.png)
Format: ![Alt Text](www.baidu.com)

## 1.5.Links
http://github.com - automatic!
[GitHub](http://github.com)

## 1.6.Blockquotes
As Kanye West said:

> We're living the future so
> the present is our past.

## 1.7.Inline code
I think you should use an
`<addr>` element here instead.

# 二、GitHub Flavored Markdown

## 2.1.Syntax highlighting

```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```
## 2.2 Task Lists
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item

