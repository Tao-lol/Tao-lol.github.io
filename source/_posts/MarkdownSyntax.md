---
title: Hexo 下的 Markdown 语法 和 MathJax 下的 LaTeX 语法
tags:
  - Markdown
categories:
  - 杂谈
date: 2019-08-30 10:42:19
# mathjax: true
katex: true
---

> Markdown：  
> https://www.ofind.cn/blog/HEXO/HEXO下的Markdown语法(GFM)写博客.html  

> LaTeX：  
> https://www.zybuluo.com/knight/note/96093  
> https://www.cnblogs.com/linxd/p/4955530.html  
> https://blog.csdn.net/ajacker/article/details/80301378  
> https://github.com/CTeX-org/lshort-zh-cn  
> https://www.mathjax.org/#demo

# Markdown
## 语法简明概述
1. 分段 `两个回车`
2. 换行 `两个空格`+`回车`
3. 标题 `#`~`######`，`#`的个数表示几级标题，即表示一级标题到六级标题
4. 强调 `**粗体**`，`__粗体__`，`*斜体*`，`_斜体_`，`***加粗斜体***`，`___加粗斜体___`，`~~删除线~~`
5. 引用 `>` 注意后面紧跟个空格，`>`的个数表示几级引用
6. 表格 `-`和`|`分割行和列，`:`控制对齐方式。表格内换行：`<br>`
7. 代码块 使用<code>\`\`\`语言</code>代码内容<code>\`\`\`</code>
8. 链接 `[文字](链接地址)`
9. 图片 `![图片说明](图片地址)`，地址可以是本地路径，也可以是网络地址
10. 无序列表 `*`，`+`，`-`，选其中之一，注意后面紧跟个空格
11. 有序列表 `1.`，`2.`，`3.`等，注意后面紧跟个空格
12. 代办事项 `- [ ] 未完成`，`- [x] 已完成`
13. 分隔线 `---`或`***`或`___`，`-`或`*`或`_`的个数三个或以上
14. 半角空格`&ensp;`或`&#8194;`
15. 全角空格`&emsp;`或`&#8195;`
16. 不断行空格`&nbsp;`或`&#160;`
<!--more-->

## 标题
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
####### 没有七级标题，但会影响生成目录，目录行多出一行空行
```


## 内容强调

### 加粗、斜体
```
字体 *斜体* 或 _斜体_ 显示
字体 **加粗** 或 __加粗__ 显示
字体 ***加粗斜体*** 或 ___加粗斜体___ 或以上任意两者组合 显示
```
> *斜体*  
> **加粗**  
> **_加粗斜体_**  

### 删除线
```
字体 ~~删除线~~ 显示
```
> ~~删除线~~  

### 高亮
```
使用<code>内容</code>或`内容`来强调内容
在code中需用\来转义符号`
```
> `强调内容`  


### 引用显示

#### 标准使用
```
每行都使用>+空格+内容
> 引用内容
```
>引用内容  

#### 省略使用
```
> 仅第一行加>号
后续内容自动变成引用内容
两个回车换行结束引用
```
> 第一行加`>`  
第二行无`>`  
第三行无`>`  

第四行无`>`  

#### 嵌套使用
```cs
> 动物
>> 水生动物
>> 陆生动物
>>> 猴子
>>> 人
>>>> 程序员
>>>> 工程师
>> 产品经理 //没有空行间隔，忽略降级引用标记
设计师 //没有空行间隔，忽略降级引用标记

>> 两栖动物
>>> 鳄鱼

```
> 动物  
>> 水生动物  
>> 陆生动物  
>>> 猴子  
>>> 人  
>>>> 程序员  
>>>> 工程师  
>> 产品经理  
设计师  

>> 两栖动物  
>>> 鳄鱼  


## 表格
```
表格语法：
 列1 | 列2 | 列3
 --- | --- | --- 
第一行|  1  | 2 
第二行|  2  | 3
```

>  列1 | 列2 | 列3  
>  --- | --- | ---   
> 第一行|  1  | 2   
> 第二行|  2  | 3  

使用冒号(:)来定义对齐方式
```
|左对齐|右对齐|居中|
|:---|---:|:-:|
|一|1|①|
|二|2|②|
```
> |左对齐|右对齐|居中|  
> |:---|---:|:-:|  
> |一|1|①|  
> |二|2|②|  

表格内换行：`<br>`

## 代码块
````
代码块使用```[可选语言]开始，```结束，如：
```cs
public class Test
{
    private string _pro;
    public string Pro
    {
        get => _pro;
        set => _pro = value;
    }
}
```
````

```cs  
public class Test
{
    private string _pro;
    public string Pro
    {
        get => _pro;
        set
        {
            _pro = value;
        }
    }
}
```

### 特别提示
如何在`代码块`中打出<code>\`\`\`</code>
只要使用4个<code>\`</code>包含3个<code>\`</code>即可，想表示更多，最外层`+1`就行。

## 链接插入
```
[文字](链接)
[首页](https://tao-lol.github.io)
```
> [首页](https://tao-lol.github.io)  


## 图片插入
```
![图片说明](图片链接)
图片链接相对路径或网络地址皆可
```
>![Life Is Strange](https://storebucket1-1258003678.cos.ap-guangzhou.myqcloud.com/Life%20Is%20Strange.jpg)  


## 列表

### 无序列表
```
*或-或+开头皆可
* 无序列表1
    * 无序列表1-1
- 无序列表2
+ 无序列表3
    + 无序列表3-1
    + 无序列表3-2
        + 无序列表3-2-1
```
> * 无序列表1
>   * 无序列表1-1 `多于一级序列一(2?)个空格`
> - 无序列表2
> + 无序列表3
>   + 无序列表3-1
>   + 无序列表3-2
>     + 无序列表3-2-1

### 有序列表
```
自动生成列表序号，最多两级
1. 有序1
1. 有序2
 1. 有序2-1
  1. 有序2-1-1
 1. 有序2-2
1. 有序3
```
> 1. 有序1
> 1. 有序2
>    1. 有序2-1 `多于一级序列一(3?)个空格`
>       1. 有序2-1-1
>    1. 有序2-2
> 1. 有序3

## 代办事项
```
- [ ] 未完成事项
- [x] 已完成事项
```
带 x 的代表已经完成的事项，空格的为还没有完成的事项
- [ ] 未完成事项
- [x] 已完成事项

## 链接自动检测
```
百度：http://www.baidu.com/
```
> 百度：http://www.baidu.com/  

## 分隔线
```
三个或以上*或-或_
***
---
___
```
> ***  
> ---  
> ___  

## 空格
```
不断行空格 &nbsp;
半角空格 &ensp;
全角空格 &emsp;
```
> 这个&nbsp;是&ensp;例&emsp;子

---

# LaTeX
## 插入公式
LaTeX 的数学公式有两种排版方式：  
* 其一是与文字混排，称为行内公式；  
* 其二是单独列为一行排版，称为行间公式。  

行内公式可以用如下两种方法表示：`\(数学公式\)`或`$数学公式$`  
行间公式可以用如下两种方法表示：`\[数学公式\]`或`$$数学公式$$`  

例子：  
行内公式：$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$
```
$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$
```

行间公式：$$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$$
```
$$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$$
```

## 字符
* 在 $\LaTeX$ 中用`^`和`_`标明上下标。注意上下标的内容（子公式）一般需要用花括号`{}`包裹，否则上下标只对后面的一个符号起作用。
$$x_{a}^{b}$$
```
$$x_{a}^{b}$$
```

* 分式使用`\frac{分子}{分母}`来书写。分式的大小在行间公式中是正常大小，而在行内被极度压缩。
$$\frac{b}{a}$$
```
$$\frac{b}{a}$$
```

* 一般的根号使用`\sqrt{...}`；表示 n 次方根时写成`\sqrt[n]{...}`。  
$$\sqrt{2} \qquad \sqrt[n]{a}$$
```
$\sqrt{2}$
$\sqrt[n]{a}$
```

---

* 函数： `\函数名`
`\sin x`$\sin x$，`\ln x`$\ln x$，`\log_n^2 5`$\log_n^2 5$，`\max(A,B,C)`$\max(A,B,C)$，`\pmod a`$\pmod a$，`\bmod a`$\bmod a$

* 矢量： `\vec{...}`
$$\vec{a} \cdot \vec{b}=0$$
```
$$\vec{a} \cdot \vec{b}=0$$
```

* 极限： `\lim`
$$\lim_{n \rightarrow +\infty} \frac{1}{n(n+1)}$$
```
$$\lim_{n \rightarrow +\infty} \frac{1}{n(n+1)}$$
```

* 求和： `\sum`
$$\sum_{i=0}^n \frac{1}{i^2}$$
```
$$\sum_{i=0}^n \frac{1}{i^2}$$
```

* 积分 `\int`$\int$，`\iint`$\iint$，`\iiint`$\iiint$，`\oint`$\oint$
$$\int_0^\infty{fxdx}$$
```
$$\int_0^\infty{fxdx}$$
```

---

矩阵：
* 需要 matrix 环境，起始标记：`\begin{matrix}`，结束标记：`\end{matrix}`
* 每一行末尾标记`\\`，行间元素之间以`&`分隔
$$\begin{matrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{matrix}$$
```
$$\begin{matrix} 1&0&0 \\ 0&1&0 \\ 0&0&1 \\ \end{matrix}$$
```

矩阵边框：
* 在起始、结束标记处用下列词替换 `matrix`

类型 | 命令 | 矩阵边框显示效果
--- | --- | ---
小括号边框 | pmatrix | $$\begin{pmatrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{pmatrix}$$
中括号边框 | bmatrix | $$\begin{bmatrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{bmatrix}$$
大括号边框 | Bmatrix | $$\begin{Bmatrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{Bmatrix}$$
单竖线边框 | vmatrix | $$\begin{vmatrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{vmatrix}$$
双竖线边框 | Vmatrix | $$\begin{Vmatrix} 1&0&0 \\\ 0&1&0 \\\ 0&0&1 \\\ \end{Vmatrix}$$

---

阵列：
* 需要 array 环境：起始、结束处以`{array}`声明
* 对齐方式：在`{array}`后以`{}`逐行统一声明
* 左对齐：`l`；居中：`c`；右对齐：`r`
* 竖直线：在声明对齐方式时，插入`|`建立竖直线
* 插入水平线：`\hline`
$$\begin{array}{c|lll} {↓}&{a}&{b}&{c} \\\ \hline {R_1}&{c}&{b}&{a} \\\ {R_2}&{b}&{c}&{c} \\\ \end{array}$$
```
$$\begin{array}{c|lll} {↓}&{a}&{b}&{c} \\ \hline {R_1}&{c}&{b}&{a} \\ {R_2}&{b}&{c}&{c} \\ \end{array}$$
```

---

方程组：
* 需要 cases 环境：起始、结束处以`{cases}`声明
$$f(n) = \begin{cases} n/2, & \text{if $n$ is even} \\\ 3n+1, & \text{if $n$ is odd} \end{cases}$$
```
$$f(n) = \begin{cases} n/2, & \text{if $n$ is even} \\ 3n+1, & \text{if $n$ is odd} \end{cases}$$
```

---

* 上划线 `\overline{...}`；下划线 `\underline{...}`。
$$0.\overline{3} = \underline{\underline{1/3}}$$
```
$$0.\overline{3} = \underline{\underline{1/3}}$$
```

* `\overbrace` 和 `\underbrace` 命令用来生成上/下括号，各自可带一个上/下标公式。
$$\underbrace{\overbrace{(a+b+c)}^6 \cdot \overbrace{(d+e+f)}^7}_\text{meaning of line} = 42$$
```
$$\underbrace{\overbrace{(a+b+c)}^6 \cdot \overbrace{(d+e+f)}^7}_\text{meaning of line} = 42$$
```

---

&emsp;&emsp;$\LaTeX$ 提供了多种括号和定界符表示公式块的边界，如小括号 ()、中括号 []、大括号 {}（`\{` `\}`）、尖括号 ⟨⟩（`\langle` `\rangle`）等。
$${a,b,c} \neq \\{a,b,c\\}$$
```
$${a,b,c} \neq \{a,b,c\}$$
```

&emsp;&emsp;使用`\left`和`\right`命令可令括号（定界符）的大小可变，在行间公式中常用。$\LaTeX$ 会自动根据括号内的公式大小决定定界符大小。`\left`和`\right`必须成对使用。需要使用单个定界符时，另一个定界符写成`\left.`或`\right.`。
\\[1 + \left(\frac{1}{1-x^{2}} \right)^3 \qquad \left.\frac{\partial f}{\partial t} \right|_{t=0}\\]
```
\[1 + \left(\frac{1}{1-x^{2}} \right)^3 \qquad \left.\frac{\partial f}{\partial t} \right|_{t=0}\]
```

&emsp;&emsp;有时我们不满意于 $\LaTeX$ 为我们自动调节的定界符大小。这时我们还可以用`\big`、`\bigg`等命令生成固定大小的定界符。更常用的形式是类似`\left`的`\bigl`、`\biggl`等，以及类似`\right`的`\bigr`、`\biggr`等（`\bigl`和`\bigr`不必成对出现）。
$$\Bigl((x+1)(x-1)\Bigr)^{2} \\\ \bigl( \Bigl( \biggl( \Biggl( \quad \bigr\\} \Bigr\\} \biggr\\} \Biggr\\} \quad \big\| \Big\| \bigg\| \Bigg\| \quad \big\Downarrow \Big\Downarrow \bigg\Downarrow \Bigg\Downarrow$$
```
$\Bigl((x+1)(x-1)\Bigr)^{2}$\\ $\bigl( \Bigl( \biggl( \Biggl( \quad \bigr\} \Bigr\} \biggr\} \Biggr\} \quad \big\| \Big\| \bigg\| \Bigg\| \quad \big\Downarrow \Big\Downarrow \bigg\Downarrow \Bigg\Downarrow$
```

&emsp;&emsp;使用`\big`和`\bigg`等命令的另外一个好处是：用`\left`和`\right`分界符包裹的公式块是不允许断行的，所以也不允许在多行公式里跨行使用，而`\big`和`\bigg`等命令不受限制。

---

&emsp;&emsp;积分号 ∫（`\int`）、求和号 ∑（`\sum`）等符号称为巨算符。巨算符在行内公式和行间公式的大 小和形状有区别。  
&emsp;&emsp;巨算符的上下标位置可由`\limits`和`\nolimits`控制，前者令巨算符类似 lim 或求和算符 ∑，上下标位于上下方；后者令巨算符类似积分号，上下标位于右上方和右下方。

---

&emsp;&emsp;数学模式中输入的空格被忽略。数学符号的间距默认由符号的性质（关系符号、运算符等）决定。需要人为引入间距时，使用`\空格`（1 格空格），`\quad`（4 格空格）和`\qquad`等命令。
&emsp;&emsp;通常来讲应当避免写出超过一行而需要折行的长公式。如果一定要折行的话，习惯上优先在等号之前折行，其次在加号、减号之前，再次在乘号、除号之前。其它位置应当避免折行。  
&emsp;&emsp;用 `\\` 折行，将公式编号放在最后一行。多行公式的首行左对齐，末行右对齐，其余行居中。

---

&emsp;&emsp;有时候可能需要一系列的公式中等号对齐，如：
$$\begin{align} \sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\\ & = \sqrt{\frac{73^2}{12^2} \cdot \frac{73^2-1}{73^2}} \\\ & = \frac{73}{12} \sqrt{1 - \frac{1}{73^2}} \\\ & \approx \frac{73}{12} \left( 1 - \frac{1}{2 \cdot 73^2} \right) \end{align}$$
```
$$\begin{align} \sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\ & = \sqrt{\frac{73^2}{12^2} \cdot \frac{73^2-1}{73^2}} \\ & = \frac{73}{12} \sqrt{1 - \frac{1}{73^2}} \\ & \approx \frac{73}{12} \left( 1 - \frac{1}{2 \cdot 73^2} \right) \end{align}$$
```

&emsp;&emsp;这时候需要使用形如`\begin{align}...\end{align}`的格式，其中需要使用`&`来指示需要对齐的位置。  
&emsp;&emsp;align 环境会给每行公式都编号。在单个式子末尾加上`\tag{...}`命令可以手动修改公式的编号，或者在每个式子末尾加上`\notag`或`\nonumber`可以去掉某行的编号。  
&emsp;&emsp;align 环境还能够对齐多组公式，除等号前的 & 之外，公式之间也用 & 分隔。

---

&emsp;&emsp;LATEX 将数学函数的名称作为一个算符排版，字体为直立字体。其中有一部分符号在上下位置可以书写一些内容作为条件，类似于后文所叙述的巨算符。  

不带上下限的算符：

不带 | 上下 | 限 | 的 | 算符
--- | --- | --- | --- | ---
\sin | \arcsin | \sinh | \exp | \dim
\cos | \arccos | \cosh | \log | \ker
\tan | \arctan | \tanh | \lg | hom
\cot | \arg | \coth | \ln | \deg
\sec | \csc

带上下限的算符：

带 | 上下 | 限 | 的 | 算符
--- | --- | --- | --- | ---
\lim | \limsup | \liminf | \sup | \inf
\min | \max | \det | \Pr | \gcd

### 字体、字号和颜色
#### 字体
示例 | 命令 | 备注
--- | --- | ---
$\mathrm{ABCDEabcde1234}$ | \mathrm{...} | 
$\mathit{ABCDEabcde1234}$ | \mathit{...} | 
$\mathbf{ABCDEabcde1234}$ | \mathbf{...} | 
$\mathsf{ABCDEabcde1234}$ | \mathsf{...} | 
$\mathtt{ABCDEabcde1234}$ | \mathtt{...} | 
$\mathcal{ABCDE}$ | \mathcal{...} | 只大写字母
$\mathscr{ABCDE}$ | \mathscr{...} | 只大写字母
$\mathfrak{ABCDEabcde1234}$ | \mathfrak{...} | 
$\mathbb{ABCDE}$ | \mathbb{...} | 只大写字母

#### 字号
示例 | 命令
--- | ---
$\tiny{ABCDEabcde1234}$ | \tiny{...}
$\scriptsize{ABCDEabcde1234}$ | \scriptsize{...}
$\small{ABCDEabcde1234}$ | \small{...}
$\normalsize{ABCDEabcde1234}$ | \normalsize{...}
$\large{ABCDEabcde1234}$ | \large{...}
$\Large{ABCDEabcde1234}$ | \Large{...}
$\LARGE{ABCDEabcde1234}$ | \LARGE{...}
$\huge{ABCDEabcde1234}$ | \huge{...}
$\Huge{ABCDEabcde1234}$ | \Huge{...}

#### 颜色
颜色 | 示例 | 命令
--- | --- | ---
black | ${\color{black} {黑色}}$ | {\color{black} {黑色}}
red | ${\color{red} {红色}}$ | {\color{red} {红色}}
green | ${\color{green} {绿色}}$ | {\color{green} {绿色}}
blue | ${\color{blue} {蓝色}}$ | {\color{blue} {蓝色}}
white | ${\color{white} {白色}}$ | {\color{white} {白色}}
cyan | ${\color{cyan} {青色}}$ | {\color{cyan} {青色}}
magenta | ${\color{magenta} {洋红色}}$ | {\color{magenta} {洋红色}}
yellow | ${\color{yellow} {黄色}}$ | {\color{yellow} {黄色}}
darkgray | ${\color{darkgray} {深灰色}}$ | {\color{darkgray} {深灰色}}
gray | ${\color{gray} {灰色}}$ | {\color{gray} {灰色}}
lightgray | ${\color{lightgray} {浅灰色}}$ | {\color{lightgray} {浅灰色}}
brown | ${\color{brown} {棕色}}$ | {\color{brown} {棕色}}
olive | ${\color{olive} {橄榄绿}}$ | {\color{olive} {橄榄绿}}
orange | ${\color{orange} {橙色}}$ | {\color{orange} {橙色}}
lime | ${\color{lime} {黄绿色}}$ | {\color{lime} {黄绿色}}
purple | ${\color{purple} {紫色}}$ | {\color{purple} {紫色}}
teal | ${\color{teal} {蓝绿色}}$ | {\color{teal} {蓝绿色}}
violet | ${\color{violet} {紫罗兰色}}$ | {\color{violet} {紫罗兰色}}
pink | ${\color{pink} {粉色}}$ | {\color{pink} {粉色}}

### LaTeX 普通符号
#### 通用符号
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\\{$ | \\{ | $\\}$ | \\} | $\\$$ | \\ $
$\\%$ | \\% | $\S$ | \S | $\dots$ | \dots

#### 希腊字母
名称 | 大写 | 命令 | 小写 | 命令 | 斜体大写 | 命令 | 斜体小写 | 命令
--- | --- | --- | --- | --- | --- | --- | --- | ---
alpha | $A$ | A | $\alpha$ | \alpha |  |  |  | 
beta | $B$ | B | $\beta$ | \beta |  |  |  | 
gamma | $\Gamma$ | \Gamma | $\gamma$ | \gamma | $\varGamma$ | \varGamma |  | 
delta | $\Delta$ | \Delta | $\delta$ | \delta | $\varDelta$ | \varDelta |  | 
epsilon | $E$ | E | $\epsilon$ | \epsilon |  |  |  | 
zeta | $Z$ | Z | $\zeta$ | \zeta |  |  |  | 
eta | $H$ | H | $\eta$ | \eta |  |  |  | 
theta | $\Theta$ | \Theta | $\theta$ | \theta | $\varTheta$ | \varTheta | $\vartheta$ | \vartheta
iota | $I$ | I | $\iota$ | \iota |  |  |  | 
kappa | $K$ | K | $\kappa$ | \kappa |  |  |  | 
lambda | $\Lambda$ | \Lambda | $\lambda$ | \lambda | $\varLambda$ | \varLambda |  | 
mu | $M$ | M | $\mu$ | \mu |  |  |  | 
nu | $N$ | N | $\nu$ | \nu |  |  |  | 
xi | $\Xi$ | \Xi | $\xi$ | \xi | $\varXi$ | \varXi |  | 
omicron | $O$ | O | $\omicron$ | \omicron 或 o |  |  |  | 
pi | $\Pi$ | \Pi | $\pi$ | \pi | $\varPi$ | \varPi | $\varpi$ | \varpi
rho | $P$ | P | $\rho$ | \rho |  |  | $\varrho$ | \varrho
sigma | $\Sigma$ | \Sigma | $\sigma$ | \sigma | $\varSigma$ | \varSigma | $\varsigma$ | \varsigma
tau | $T$ | T | $\tau$ | \tau |  |  |  | 
upsilon | $\Upsilon$ | \Upsilon | $\upsilon$ | \upsilon | $\varUpsilon$ | \varUpsilon |  | 
phi | $\Phi$ | \Phi | $\phi$ | \phi | $\varPhi$ | \varPhi | $\varphi$ | \varphi
chi | $X$ | X | $\chi$ | \chi |  |  |  | 
psi | $\Psi$ | \Psi | $\psi$ | \psi | $\varPsi$ | \varPsi |  | 
omega | $\Omega$ | \Omega | $\omega$ | \omega | $\varOmega$ | \varOmega |  | 

* \Alpha, \Beta 等希腊字母符号不存在，因为它们和拉丁字母 A, B 等一模一样；
* 小写字母里也不存在 \omicron，直接用拉丁字母 o 代替。

#### 二元关系符
示例 | 命令 | 示例 | 命令 | 示例 | 命令 
--- | --- | --- | --- | --- | --- 
$<$ | < | $>$ | > | $=$ | =
$\leq$ | \leq 或 \le | $\geq$ | \geq 或 \ge | $\equiv$ | \equiv
$\ll$ | \ll | $\gg$ | \gg | $\doteq$ | \doteq
$\prec$ | \prec | $\succ$ | \succ | $\sim$ | \sim
$\preceq$ | \preceq | $\succeq$ | \succeq | $\simeq$ | \simeq
$\subset$ | \subset | $\supset$ | \supset | $\approx$ | \approx
$\subseteq$ | \subseteq | $\supseteq$ | \supseteq | $\cong$ | \cong
$\sqsubset$ | \sqsubset | $\sqsupset$ | \sqsupset | $\Join$ | \Join
$\sqsubseteq$ | \sqsubseteq | $\sqsupseteq$ | \sqsupseteq | $\bowtie$ | \bowtie
$\in$ | \in | $\ni$ | \ni 或 \owns | $\propto$ | \propto
$\vdash$ | \vdash | $\dashv$ | \dashv | $\models$ | \models
$\mid$ | \mid | $\parallel$ | \parallel | $\perp$ | \perp
$\smile$ | \smile | $\frown$ | \frown | $\asymp$ | \asymp
$:$ | : | $\notin$ | \notin | $\neq$ | \neq 或 \ne

* 所有的二元关系符都可以加`\not`前缀得到相反意义的关系符，例如`\not=`就得到不等号（同`\ne`）。

#### 二元运算符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$+$ | + | $-$ | - |  | 
$\pm$ | \pm | $\mp$ | \mp | $\triangleleft$ | \triangleleft
$\cdot$ | \cdot | $\div$ | \div | $\triangleright$ | \triangleright
$\times$ | \times | $\setminus$ | \setminus | $\star$ | \star
$\cup$ | \cup | $\cap$ | \cap | $\ast$ | \ast
$\sqcup$ | \sqcup | $\sqcap$ | \sqcap | $\circ$ | \circ
$\vee$ | \vee 或 \lor | $\wedge$ | \wedge 或 \land | $\bullet$ | \bullet
$\oplus$ | \oplus | $\ominus$ | \ominus | $\diamond$ | \diamond
$\odot$ | \odot | $\oslash$ | \oslash | $\uplus$ | \uplus
$\otimes$ | \otimes | $\bigcirc$ | \bigcirc | $\amalg$ | \amalg
$\bigtriangleup$ | \bigtriangleup | $\bigtriangledown$ | \bigtriangledown | $\dagger$ | \dagger
$\lhd$ | \lhd | $\rhd$ | \rhd | $\ddagger$ | \ddagger
$\unlhd$ | \unlhd | $\unrhd$ | \unrhd | $\wr$ | \wr

#### 巨算符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\sum$ | \sum | $\bigcup$ | \bigcup | $\bigvee$ | \bigvee
$\prod$ | \prod | $\bigcap$ | \bigcap | $\bigwedge$ | \bigwedge
$\coprod$ | \coprod | $\bigsqcup$ | \bigsqcup | $\biguplus$ | \biguplus
$\int$ | \int | $\oint$ | \oint | $\bigodot$ | \bigodot
$\bigoplus$ | \bigoplus | $\bigotimes$ | \bigotimes |  | 
$\iint$ | \iint | $\iiint$ | \iiint | $\iiiint$ | \iiiint
$\idotsint$ | \idotsint |  |  |  | 

#### 数学重音符号
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\hat{a}$ | \hat{a} | $\check{a}$ | \check{a} | $\tilde{a}$ | \tilde{a}
$\acute{a}$ | \acute{a} | $\grave{a}$ | \grave{a} | $\breve{a}$ | \breve{a}
$\bar{a}$ | \bar{a} | $\vec{a}$ | \vec{a} | $\mathring{a}$ | \mathring{a}
$\dot{a}$ | \dot{a} | $\ddot{a}$ | \ddot{a} | $\dddot{a}$ | \dddot{a}
$\ddddot{a}$ | \ddddot{a} |  |  |  | 
$\widehat{AAA}$ | \widehat{AAA} | $\widetilde{AAA}$ | \widetilde{AAA} |  | 

#### 箭头
示例 | 命令 | 示例 | 命令
--- | --- | --- | ---
$\leftarrow$ | \leftarrow 或 \gets | $\longleftarrow$ | \longleftarrow
$\rightarrow$ | \rightarrow 或 \to | $\longrightarrow$ | \longrightarrow
$\leftrightarrow$ | \leftrightarrow | $\longleftrightarrow$ | \longleftrightarrow
$\Leftarrow$ | \Leftarrow | $\Longleftarrow$ | \Longleftarrow
$\Rightarrow$ | \Rightarrow | $\Longrightarrow$ | \Longrightarrow
$\Leftrightarrow$ | \Leftrightarrow | $\Longleftrightarrow$ | \Longleftrightarrow
$\mapsto$ | \mapsto | $\longmapsto$ | \longmapsto
$\hookleftarrow$ | \hookleftarrow | $\hookrightarrow$ | \hookrightarrow
$\leftharpoondown$ | \leftharpoondown | $\rightharpoonup$ | \rightharpoonup
$\leftharpoondown$ | \leftharpoondown | $\rightharpoondown$ | \rightharpoondown
$\rightleftharpoons$ | \rightleftharpoons | $\iff$ | \iff
$\uparrow$ | \uparrow | $\downarrow$ | \downarrow
$\updownarrow$ | \updownarrow | $\Uparrow$ | \Uparrow
$\Downarrow$ | \Downarrow | $\Updownarrow$ | \Updownarrow
$\nearrow$ | \nearrow | $\searrow$ | \searrow
$\swarrow$ | \swarrow | $\nwarrow$ | \nwarrow
$\leadsto$ | \leadsto |  | 

#### 作为重音的箭头符号
示例 | 命令 | 示例 | 命令
--- | --- | --- | ---
$\overrightarrow{AB}$ | \overrightarrow{AB} | $\underrightarrow{AB}$ | \underrightarrow{AB}
$\overleftarrow{AB}$ | \overleftarrow{AB} | $\underleftarrow{AB}$ | \underleftarrow{AB}
$\overleftrightarrow{AB}$ | \overleftrightarrow{AB} | $\underleftrightarrow{AB}$ | \underleftrightarrow{AB}

#### 定界符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$($ | ( | $)$ | ) | $\uparrow$ | \uparrow
$[$ | [ 或 \lbrack | $]$ | ] 或 \rbrack | $\downarrow$ | \downarrow
$\lbrace$ | \\{ 或 \lbrace | $\rbrace$ | \\} 或 \rbrace | $\updownarrow$ | \updownarrow
$\langle$ | \langle | $\rangle$ | \rangle | $\Uparrow$ | \Uparrow
$\vert$ | \| 或 \vert | $\Vert$ | \\\| 或 \Vert | $\Downarrow$ | \Downarrow
$/$ | / | $\backslash$ | \backslash | $\Updownarrow$ | \Updownarrow
$\lfloor$ | \lfloor | $\rfloor$ | \rfloor |  | 
$\rceil$ | \rceil | $\lceil$ | \lceil |  | 

#### 用于行间公式的大定界符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\lgroup$ | \lgroup | $\rgroup$ | \rgroup | $\lmoustache$ | \lmoustache
$\arrowvert$ | \arrowvert | $\Arrowvert$ | \Arrowvert | $\bracevert$ | \bracevert
$\rmoustache$ | \rmoustache |  |  |  | 

#### 其它符号
示例 | 命令 | 示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | --- | --- | ---
$\dots$ | \dots | $\cdots$ | \cdots | $\vdots$ | \vdots | $\ddots$ | \ddots
$\hbar$ | \hbar | $\imath$ | \imath | $\jmath$ | \jmath | $\ell$ | \ell
$\Re$ | \Re | $\Im$ | \Im | $\aleph$ | \aleph | $\wp$ | \wp
$\forall$ | \forall | $\exists$ | \exists | $\mho$ | \mho | $\partial$ | \partial
$'$ | ' | $\prime$ | \prime | $\emptyset$ | \emptyset | $\infty$ | \infty
$\nabla$ | \nabla | $\triangle$ | \triangle | $\Box$ | \Box | $\Diamond$ | \Diamond
$\bot$ | \bot | $\top$ | \top | $\angle$ | \angle | $\surd$ | \surd
$\diamondsuit$ | \diamondsuit | $\heartsuit$ | \heartsuit | $\clubsuit$ | \clubsuit | $\spadesuit$ | \spadesuit
$\neg$ | \neg 或 \lnot | $\flat$ | \flat | $\natural$ | \natural | $\sharp$ | \sharp

### AMS 符号
#### AMS 希腊字母和希伯来字母
示例 | 命令 | 示例 | 命令 | 示例 | 命令 | 示例 | 命令 | 示例 | 命令 
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- 
$\digamma$ | \digamma | $\varkappa$ | \varkappa | $\beth$ | \beth | $\gimel$ | \gimel | $\daleth$ | \daleth

#### AMS 二元关系符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\lessdot$ | \lessdot | $\gtrdot$ | \gtrdot | $\doteqdot$ | \doteqdot
$\leqslant$ | \leqslant | $\geqslant$ | \geqslant | $\risingdotseq$ | \risingdotseq
$\eqslantless$ | \eqslantless | $\eqslantgtr$ | \eqslantgtr | $\fallingdotseq$ | \fallingdotseq
$\leqq$ | \leqq | $\geqq$ | \geqq | $\eqcirc$ | \eqcirc
$\lll$ | \lll 或 \llless | $\ggg$ | \ggg | $\circeq$ | \circeq
$\lesssim$ | \lesssim | $\gtrsim$ | \gtrsim | $\triangleq$ | \triangleq
$\lessapprox$ | \lessapprox | $\gtrapprox$ | \gtrapprox | $\bumpeq$ | \bumpeq
$\lessgtr$ | \lessgtr | $\gtrless$ | \gtrless | $\Bumpeq$ | \Bumpeq
$\lesseqgtr$ | \lesseqgtr | $\gtreqless$ | \gtreqless | $\thicksim$ | \thicksim
$\lesseqqgtr$ | \lesseqqgtr | $\gtreqqless$ | \gtreqqless | $\thickapprox$ | \thickapprox
$\preccurlyeq$ | \preccurlyeq | $\succcurlyeq$ | \succcurlyeq | $\approxeq$ | \approxeq
$\curlyeqprec$ | \curlyeqprec | $\curlyeqsucc$ | \curlyeqsucc | $\backsim$ | \backsim
$\precsim$ | \precsim | $\succsim$ | \succsim | $\backsimeq$ | \backsimeq
$\precapprox$ | \precapprox | $\succapprox$ | \succapprox | $\vDash$ | \vDash
$\subseteqq$ | \subseteqq | $\supseteqq$ | \supseteqq | $\Vdash$ | \Vdash
$\shortparallel$ | \shortparallel | $\Supset$ | \Supset | $\Vvdash$ | \Vvdash
$\blacktriangleleft$ | \blacktriangleleft | $\sqsupset$ | \sqsupset | $\backepsilon$ | \backepsilon
$\vartriangleright$ | \vartriangleright | $\because$ | \because | $\varpropto$ | \varpropto
$\blacktriangleright$ | \blacktriangleright | $\Subset$ | \Subset | $\between$ | \between
$\trianglerighteq$ | \trianglerighteq | $\smallfrown$ | \smallfrown | $\pitchfork$ | \pitchfork
$\vartriangleleft$ | \vartriangleleft | $\shortmid$ | \shortmid | $\smallsmile$ | \smallsmile
$\trianglelefteq$ | \trianglelefteq | $\therefore$ | \therefore | $\sqsubset$ | \sqsubset

#### AMS 二元运算符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\dotplus$ | \dotplus | $\centerdot$ | \centerdot |  | 
$\ltimes$ | \ltimes | $\rtimes$ | \rtimes | $\divideontimes$ | \divideontimes
$\doublecup$ | \doublecup | $\doublecap$ | \doublecap | $\smallsetminus$ | \smallsetminus
$\veebar$ | \veebar | $\barwedge$ | \barwedge | $\doublebarwedge$ | \doublebarwedge
$\boxplus$ | \boxplus | $\boxminus$ | \boxminus | $\circleddash$ | \circleddash
$\boxtimes$ | \boxtimes | $\boxdot$ | \boxdot | $\circledcirc$ | \circledcirc
$\intercal$ | \intercal | $\circledast$ | \circledast | $\rightthreetimes$ | \rightthreetimes
$\curlyvee$ | \curlyvee | $\curlywedge$ | \curlywedge | $\leftthreetimes$ | \leftthreetimes

#### AMS 箭头
示例 | 命令 | 示例 | 命令
--- | --- | --- | ---
$\dashleftarrow$ | \dashleftarrow | $\dashrightarrow$ | \dashrightarrow
$\leftleftarrows$ | \leftleftarrows | $\rightrightarrows$ | \rightrightarrows
$\leftrightarrows$ | \leftrightarrows | $\rightleftarrows$ | \rightleftarrows
$\Lleftarrow$ | \Lleftarrow | $\Rrightarrow$ | \Rrightarrow
$\twoheadleftarrow$ | \twoheadleftarrow | $\twoheadrightarrow$ | \twoheadrightarrow
$\leftarrowtail$ | \leftarrowtail | $\rightarrowtail$ | \rightarrowtail
$\leftrightharpoons$ | \leftrightharpoons | $\rightleftharpoons$ | \rightleftharpoons
$\Lsh$ | \Lsh | $\Rsh$ | \Rsh
$\looparrowleft$ | \looparrowleft | $\looparrowright$ | \looparrowright
$\curvearrowleft$ | \curvearrowleft | $\curvearrowright$ | \curvearrowright
$\circlearrowleft$ | \circlearrowleft | $\circlearrowright$ | \circlearrowright
$\multimap$ | \multimap | $\upuparrows$ | \upuparrows
$\downdownarrows$ | \downdownarrows | $\upharpoonleft$ | \upharpoonleft
$\upharpoonright$ | \upharpoonright | $\downharpoonright$ | \downharpoonright
$\rightsquigarrow$ | \rightsquigarrow | $\leftrightsquigarrow$ | \leftrightsquigarrow

#### AMS 反义二元关系符和箭头
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\nless$ | \nless | $\ngtr$ | \ngtr | $\varsubsetneqq$ | \varsubsetneqq
$\lneq$ | \lneq | $\gneq$ | \gneq | $\varsupsetneqq$ | \varsupsetneqq
$\nleq$ | \nleq | $\ngeq$ | \ngeq | $\nsubseteqq$ | \nsubseteqq
$\nleqslant$ | \nleqslant | $\ngeqslant$ | \ngeqslant | $\nsupseteqq$ | \nsupseteqq
$\lneqq$ | \lneqq | $\gneqq$ | \gneqq | $\nmid$ | \nmid
$\lvertneqq$ | \lvertneqq | $\gvertneqq$ | \gvertneqq | $\nparallel$ | \nparallel
$\nleqq$ | \nleqq | $\ngeqq$ | \ngeqq | $\nshortmid$ | \nshortmid
$\lnsim$ | \lnsim | $\gnsim$ | \gnsim | $\nshortparallel$ | \nshortparallel
$\lnapprox$ | \lnapprox | $\gnapprox$ | \gnapprox | $\nsim$ | \nsim
$\nprec$ | \nprec | $\nsucc$ | \nsucc | $\ncong$ | \ncong
$\npreceq$ | \npreceq | $\nsucceq$ | \nsucceq | $\nvdash$ | \nvdash
$\precneqq$ | \precneqq | $\succneqq$ | \succneqq | $\nvDash$ | \nvDash
$\precnsim$ | \precnsim | $\succnsim$ | \succnsim | $\nVdash$ | \nVdash
$\precnapprox$ | \precnapprox | $\succnapprox$ | \succnapprox | $\nVDash$ | \nVDash
$\subsetneq$ | \subsetneq | $\supsetneq$ | \supsetneq | $\ntriangleleft$ | \ntriangleleft
$\varsubsetneq$ | \varsubsetneq | $\varsupsetneq$ | \varsupsetneq | $\ntriangleright$ | \ntriangleright
$\nsubseteq$ | \nsubseteq | $\nsupseteq$ | \nsupseteq | $\ntrianglelefteq$ | \ntrianglelefteq
$\subsetneqq$ | \subsetneqq | $\supsetneqq$ | \supsetneqq | $\ntrianglerighteq$ | \ntrianglerighteq
$\nleftarrow$ | \nleftarrow | $\nrightarrow$ | \nrightarrow | $\nleftrightarrow$ | \nleftrightarrow
$\nLeftarrow$ | \nLeftarrow | $\nRightarrow$ | \nRightarrow | $\nLeftrightarrow$ | \nLeftrightarrow

#### AMS 定界符
示例 | 命令 | 示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | --- | --- | ---
$\ulcorner$ | \ulcorner | $\urcorner$ | \urcorner | $\llcorner$ | \llcorner | $\lrcorner$ | \lrcorner
$\lvert$ | \lvert | $\rvert$ | \rvert | $\lVert$ | \lVert | $\rVert$ | \rVert

#### AMS 其它字符
示例 | 命令 | 示例 | 命令 | 示例 | 命令
--- | --- | --- | --- | --- | ---
$\hbar$ | \hbar | $\hslash$ | \hslash | $\Bbbk$ | \Bbbk
$\square$ | \square | $\blacksquare$ | \blacksquare | $\circledS$ | \circledS
$\vartriangle$ | \vartriangle | $\blacktriangle$ | \blacktriangle | $\complement$ | \complement
$\triangledown$ | \triangledown | $\blacktriangledown$ | \blacktriangledown | $\Game$ | \Game
$\lozenge$ | \lozenge | $\blacklozenge$ | \blacklozenge | $\bigstar$ | \bigstar
$\angle$ | \angle | $\measuredangle$ | \measuredangle |  | 
$\diagup$ | \diagup | $\diagdown$ | \diagdown | $\backprime$ | \backprime
$\nexists$ | \nexists | $\Finv$ | \Finv | $\varnothing$ | \varnothing
$\eth$ | \eth | $\sphericalangle$ | \sphericalangle | $\mho$ | \mho
