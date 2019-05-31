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

## 2.3 Tables
You can create tables by assembling a list of words and dividing them with hyphens - (for the first row), and then separating each column with a pipe |:

First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column

## 2.4 SHA references
Any reference to a commit’s SHA-1 hash will be automatically converted into a link to that commit on GitHub.

16c999e8c71134401a78d4d46435517b2271d6ac
mojombo@16c999e8c71134401a78d4d46435517b2271d6ac
mojombo/github-flavored-markdown@16c999e8c71134401a78d4d46435517b2271d6ac

## 2.5 Issue references within a repository
Any number that refers to an Issue or Pull Request will be automatically converted into a link.

#1
mojombo#1
mojombo/github-flavored-markdown#1

## 2.6 Username @mentions
Typing an @ symbol, followed by a username, will notify that person to come and view the comment. This is called an “@mention”, because you’re mentioning the individual. You can also @mention teams within an organization.

## 2.7 Automatic linking for URLs
Any URL (like http://www.github.com/) will be automatically converted into a clickable link.

## 2.8 Strikethrough
Any word wrapped with two tildes (like ~~this~~) will appear crossed out.

## 2.9 Emoji
GitHub supports emoji!

To see a list of every image we support, check out the Emoji Cheat Sheet.



