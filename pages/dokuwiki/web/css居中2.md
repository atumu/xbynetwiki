title: css居中2 

#  CSS居中完整版 
原版https://css-tricks.com/centering-css-complete-guide/
中文https://github.com/Erichain/css-center-complete
##  横向居中 
###  行内元素水平居中 
**是行内元素或者行内块级元素？(inline 或者 inline-block)**
你可以将行内元素居中在块级元素中，就像这样：
```

.center-children {
    text-align: center;
}

```
这个方法对于 display 属性为 inline, inline-block, inline-table, inline-flex 等的元素都有作用。

###  块级元素水平居中 
对于块级元素，可以通过设置他的 margin 属性为 auto 来达到居中的效果**。前提是要设置一个宽度**。如果不设置宽度的话，默认为100%，就用不着居中了。就像这样：
```

.center-me {
    margin: 0 auto;
}

```

###  多个块级元素水平居中 
如果需要在一行中居中两个及以上的块级元素，**最好给他们设置 display 属性为 inline-block** 。
除非你是想多个块级元素都在各自的顶部，如果是这样的话，那么使用 mrgin: 0 auto； 也可以；


##  垂直居中 
###  行内元素垂直居中 
行内元素或者行内块级元素？(inline 或者 inline-block)
单独一行
有的时候行内元素很明显可以垂直居中。` 只需要设置它们的上下 padding 值相等 `：
```

.link {
    padding-top: 30px;
    padding-bottom: 30px;
}

```
**如果设置 padding 不行，而且你想居中的是文本的话，那么，可以设置文本的 line-height 与元素的 height 相等。**
```

.center-text-trick {
    height: 100px;
    line-height: 100px;
    white-space: nowrap;
}

```
**多行**
**1、相等的 padding 对多行的情况也适用。如果不起作用的话，那么这个元素或者文本的 display 属性可能是 table-cell 。这种情况下， vertical-align 就有作用了。**与其它情况不同，这个是用来处理一行内的元素居中的。
###  flexbox垂直居中 
2、如果类表格元素的居中不起作用，那么是否考虑使用flexbox？**在flexbox的父元素中居中flexbox子元素就太简单了**。
```

.flex-center-vertically {
    display: flex;
    justify-content: center;
    flex-direction: column;
    height: 400px;
}

```
**记住只有父级元素有固定的高度，这样写才有意义。**

###  ghost element 方法垂直居中 
3、如果前面两种方法都不起作用，可以使用 ghost element 方法。在包含块里放置一个高度为100%的伪元素，这样，文本就居中了。
```

.ghost-center {
    position: relative;
}
.ghost-center::before {
    content: " ";
    display: inline-block;
    height: 100%;
    width: 1%;
    vertical-align: middle;
}
.ghost-center p {
    display: inline-block;
    vertical-align: middle;
}

```

###  块级元素垂直居中 
####  知道元素的高度 

不知道网页布局的高度简直是太习以为常的事情了。各种情况都会出现：
宽度改变，文字重排，高度会改变
不同的文字样式的高度也不一样
不同文本的数量的高度也不一样
固定比例的元素，比如图片啥的，在改变尺寸的时候也会改变高度等等
**但是如果你知道元素的高度就好办了：**
```

.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    height: 100px;
    margin-top: -50px; 
    /* 如果没有使用border-box的话就只需要关心padding和border了 */
}

```
####  不确定元素的高度 
不知道元素高度的情况下，通过先将他往下移动50%，然后再向上移动他的高度的一半来居中也还是有可能的。
```

.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}

```
####  考虑flexbox 
别太惊讶，使用flexbox就太简单了
```

.parent {
    display: flex;
    flex-direction: column;
    justify-content: center;
}

```

##  横竖都居中 
你完全可以用各种方式将上面的技术结合起来达到完美居中的效果。但我觉得可以把这些情况分为下面三种：
###  元素是否是固定的宽高 
在使用绝对定位分别设置上下50％和左右50％之后，使用分别等于宽高一半的负边距就能够跨浏览器实现完全居中了：
```

.parent {
    position: relative;
}
.child {
    width: 300px;
    height: 100px;
    padding: 20px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin: -70px 0 0 -170px;
}

```
###  不确定元素的宽高 
如果不知道元素的宽高，那么可以使用 transform 属性在两个不同的方向上设置 -50% (基于当前元素的宽高)来居中：
```

.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

```
###  flexbox 
要在flexbox中居中，需要用到两个居中属性：
```

.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}

```
总结
经过上面这些方法，我们完全可以使用CSS来达到完美的居中。