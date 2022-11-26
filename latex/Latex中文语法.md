

# Latex中文语法

## 1.Latex行文结构

```
\documentclass[options]{class}
\begin{document}
正文内容
\end{document}
```

> options : 文档类的属性，不同的选项之间用逗号隔开
>
> class: 指定文档类型
>
> > * article   - for articles in scientific journals, presentations, short reports, program documentation, invitations, …（科学期刊、 演示文档、 短报告、 程序文档、 邀请函等）
> > * book   - for real books（书籍排版）
> > * letter   - for writing letters.（信件书写）
> > * report   - for longer reports containing several chapters, small books, thesis, …（多章节长报告、 短篇书籍、 博士论文等）
> > * proc   - a class for proceedings based on the article class.（基于 article 的会议文集类）
> > * slides   - for slides. The class uses big sans serif letters.（幻灯片。 该文档类使用大号 sans serif 字体。）
> > * minimal   - is as small as it can get. It only sets a page size and a base font. It is mainly used for debugging 
> > * purposes.（非常小的文档类。 只设置了页面尺寸和基本字体。 主要用来查错。）
> > * beamer   - for writing presentations.（演示文稿编写）

```
内容层次：
part   –   篇，实对章节的归类
chapter   –   章
section   –   一级标题，也就是节
subsection   –   二级标题
subsubsection   –   三级标题
paragraph   –   段，是一段文字的说明
subparagraph   –   子段，是以列举的形式对前段文字的说明
```

## 数学公式：

```
数学公式：
行内(inline)模式：即在正文中插入数学内容。行间公式用 $ … $     $ f(x) = a+b $
行间公式 :占行 $$ f(x) = a+b $$
手动编号: $$ f(x) = a - b \tag{1.1} $$
独立(display)模式：独立成行，可以有或没有编号。无编号用\ [ … \ ]
有编号
\begin{equation}
1+2+3+\dots+(n-1)+n = \frac{n(n+1)}{2}
\end{equation}

符号：
1.\cdot表示乘法的圆点，命令\neq表示不等号，命令\equiv表示恒等于，命令\bmod表示取模
		$$ 0 \neq 1 \quad x \equiv x \quad 1 = 9 \bmod 2 $$
2.语法_表示下标、^表示上标，但上下标内容不止一个字符时，需用大括号括起来。单引号'表示求导
		$$ a_{ij}^{2} + b^3_{2}=x^{t} + y' + x''_{12} $$
3.\sqrt表示平方根，\sqrt[n]表示n次方根，\frac表示分式
		$$\sqrt{x} + \sqrt{x^{2}+\sqrt{y}} = \sqrt[3]{k_{i}} - \frac{x}{m}$$
4.\overline, \underline 分别在表达式上、下方画出水平线
		$$\overline{x+y} \qquad \underline{a+b}$$
5.\vec表示向量，\overrightarrow表示箭头向右的向量，\overleftarrow表示箭头向左的向量
		$$\vec{a} + \overrightarrow{AB} + \overleftarrow{DE}$$
6.\int表示积分，\lim表示极限， \sum表示求和，\prod表示乘积，^、_表示上、下限
		$$  \lim_{x \to \infty} x^2_{22} - \int_{1}^{5}x\mathrm{d}x + \sum_{n=1}^{20} n^{2} = \prod_{j=1}^{3} y_{j}  + \lim_{x \to -2} \frac{x-2}{x} $$
7.\ldots点位于基线上，\cdots点设置为居中，\vdots使其垂直，\ddots对角线排列
		$$ x_{1},x_{2},\ldots,x_{5}  \quad x_{1} + x_{2} + \cdots + x_{n} $$
```

![image-20221002141626361](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20221002141626361.png)

## 图片：

```
图片：
\usepackage{graphicx}  %插入图片
...
\includegraphics[scale=1]{yanzhenqing.png} %插入图片
scale缩放率
```

## 插入代码

```
插入代码
\usepackage{listings}
...
\begin{lstlisting}
% 代码段
\end{lstlisting}

配置：
\lstset{
 columns=fixed,       
 numbers=left,                                        % 在左侧显示行号
 numberstyle=\tiny\color{gray},                       % 设定行号格式
 frame=none,                                          % 不显示背景边框
 backgroundcolor=\color[RGB]{245,245,244},            % 设定背景颜色
 keywordstyle=\color[RGB]{40,40,255},                 % 设定关键字颜色
 numberstyle=\footnotesize\color{darkgray},           
 commentstyle=\it\color[RGB]{0,96,96},                % 设置代码注释的格式
 stringstyle=\rmfamily\slshape\color[RGB]{128,0,0},   % 设置字符串格式
 showstringspaces=false,                              % 不显示字符串中的空格
 language=c++,                                        % 设置语言
}

```

## 注释

- 单行 %开头

- 多行 (需引入多行注释的包  \usepackage{verbatim})

  ```
  /begin{comment}
  
  注释内容
  
  /end{comment}
  ```

  

- 脚注 \footnote{脚注内容}

## 换行、分段、分页

> **换行**
>
> - \\\  换行
> - \newline   与\\相同
> - 注意，这里的换行，都是在段内换行
>
> **分段**
>
> - \par   添加在段落末尾或另起一行进行分段
> - 在段落后连续两个回车，也可以实现分段效果（推荐的方式）
>
> **分页**
>
> - \newpage   添加在段落末尾或另起一行进行分页

## 文字样式

>**粗体**
>
>\textbf{文字}
>**斜体**
>
>\emph{文字}，（ 备注，中文文档里面斜体配置没有成功，以后有时间再改进）
>**颜色**
>
> 3种方式可选
>
>```
>直接使用定义好的颜色
>
>1. \usepackage{color}
>
>2. \textcolor{red/blue/green/black/white/cyan/magenta/yellow}{text}
>
>3. % 其中textcolor{…}中包含的是系统定义好的颜色
>
>   * 组合red、green和blue的值合成我们想要的颜色
>
>     1. \usepackage{color}
>
>     2. \textcolor[RGB]{R,G,B}{text}
>     3. % 其中{R,G,B}代表red、green和blue三种颜色的组合，取值范围为[0-255]
>        * 定义一种颜色，直接调用
>          1. \usepackage{color}
>          2. \definecolor{ColorName}{RGB}{R,G,B} % 这时R/G/B的定义域就在[0-255]
>          3. \textcolor{ColorName}{text}
>          4. 这里为颜色定义了名称ColorName，下面可以直接调用这个颜色方案
>```
>
>**大小**
>
>```
>全局模式
>
>1. \documentclass[12pt]{article}
>   - 局部模式
>     1. 根据既有命令设置
>        命令：\tiny、\scriptsize、\footnotesize、\small、\normalsize、\large、\Large、\LARGE、\huge、\Huge
>        示例，\tiny{Latex}，注意命令跟文字之间有空格
>     2. 自定义修改字体大小和尺寸
>        \fontsize{字体尺寸}{行距}
>        示例， \fontsize{20pt}{24pt} 中国，注意命令跟文字之间有空格
>```

## 下划线、双下划线、波浪线、删除线、斜删除线

```
\usepackage{ulem}
…
\uline{} %下划线
\uuline{} %双下划线
\uwave{} %波浪线
\sout{} %删除线
\xout{} %斜删除线
```

## 使用BibTeX生成参考文献列表

```
%! Tex program = xelatex
\documentclass{article}
\usepackage[UTF8]{ctex}
\begin{document}
123456\cite{quanxue}。
\bibliography{bibFormat}{}
\bibliographystyle{plain}
\end{document}
“~\cite{name}” ，这里的 name 用于在 bib 文件中检索。
```

```
创建一个BibTeX参考文献库文件，文件的命名跟上述语句保持一致 \bibliography{bibFormat}{} ，也就是创建名为 “bibFormat” 的bib文件，即 bibFormat.bib。录入如下内容，每一个参考文献，写一个 “@misc{ …… }”。
@misc{ quanxue,
        author = "",
        title = "",
        year = ""}
```

