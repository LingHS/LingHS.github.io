---
title: FFC之margin
date: 2020-05-17 12:15:47
tags:
categories:
---

# FFC

Flex Formatting Context 自适应格式化上下文。尽管很多浏览器已经支持 flex，但目前仍是 CR 阶段。

Flex 布局在表面上类似于块布局。它缺少许多可用于块布局的更复杂的以文本或文档为中心的属性，例如 float 和 column。作为回报，它获得了简单而强大的工具，可以按照 Web 应用程序和复杂网页经常需要的方式来分配空间和对齐内容。
可以在任何流动方向上进行布局（向左，向右，向下，甚至向上！）

可以在样式层颠倒或重新排列它们的显示顺序（即，视觉顺序可以独立于源和语音顺序）

可以沿单个（主轴）轴线性布置，也可以沿辅助（交叉）轴缠绕成多行

可以“调整”其大小以响应可用空间

可以相对于其容器对齐，也可以在次级容器上对齐（十字）

可以沿主轴动态折叠或展开，同时保持容器的十字尺寸。

<!--- more --->

# Flex

关于 flex 的使用 https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html 阮一峰的这篇文章已经讲的很好了，当然类似由于时间久远有一些类似 space-evenly 的属性没有涉及，这里可看 https://developer.mozilla.org/zh-CN/docs/Web/CSS/justify-content

# Margin

我们主要说的是 auto 这个神奇的属性

## BFC

在 BFC 中 auto 主要用来使元素水平居中

```css
div {
  width: 100px;
  height: 100px;
  margin: 0 auto;
}
```

> If both margin-left and margin-right are auto, their used values are equal, causing horizontal centring.  
> If margin-top, or margin-bottom are auto, their used value is 0.

auto 只能用来水平居中而不能垂直居中，因为在块格式化上下文中，如果 margin-left 和 margin-right 都是 auto，则它们的表达值相等，从而导致元素的水平居中。( 这里的计算值为元素剩余可用剩余空间的一半)

而如果 margin-top 和 margin-bottom 都是 auto，则他们的值都为 0，当然也就无法造成垂直方向上的居中。

## FFC

> Prior to alignment via justify-content and align-self, any positive free space is distributed to auto margins in that dimension.

在 FFC 中，设置了 margin: auto 的元素，在通过 justify-content 和 align-self 进行对齐之前，任何正处于空闲的空间都会分配到该方向的自动 margin 中去

这里，很重要的一点是，margin auto 的生效不仅是水平方向，垂直方向也会自动去分配这个剩余空间。

**例：**

<div style="width:100%;background:#367b66;height:50px;display:flex;align-items:center">
    <div style="padding:0 10px">栏目1 </div>
    <div style="padding:0 10px">栏目2</div>
    <div style="padding:0 10px">栏目3</div>
    <div style="padding:0 10px; margin-left:auto">栏目4</div>
</div>

在父级设置 dislay：flex，在子级的最后一个元素设置 margin-left：auto，这样剩余的空间会全部分配到 auto 上，从而实现靠右的布局。
