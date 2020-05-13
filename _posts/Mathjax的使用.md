# Mathjax的使用

## 常用符号的记录

常用符号的查询：https://www.maixj.net/wz/mathjax-14939



MathJax采用的是LaTex语法，在应用的过程中，我们会经常使用一些希腊字母和希伯来字母，在这里记录一些常用的字符用来查询。
$$
\alpha \\
\beta \\
\gamma \\
\delta \\ 
\epsilon \\
\zeta \\
\eta \\
\theta \\
\iota \\
\kappa \\
\lambda \\
\mu \\
\xi \\
\pi \\
\rho \\
\sigma \\
\varsigma \\
\upsilon \\
\phi \\
\chi \\
\psi \\
\omega \\
$$
还有一些常用的逻辑运算法：
$$
\because \\
\therefore \\
\forall \\
\exists \\
\not= \\
\not> \\
\not\subset
$$
集合运算符：
$$
\emptyset \\
\in \\
\notin \\
\subset \\
\supset \\
\subseteq \\
\supseteq \\
\bigcap \\
\bigcup \\
\bigvee \\
\bigwedge \\
\biguplus \\
\bigsqcup
$$
关系运算符：
$$
\mid \\
\nmid \\
\cdot \\
\leq \\
\geq \\
\neq \\
\approx \\
\equiv \\
\circ \\
\ast \\
\bigodot \\
\bigotimes \\
\bigoplus \\
\pm \\
\times \\
\div \\
\sum \\
\prod \\
\coprod 
$$

## 常用的操作

1. 行内的公式使用 **$...$**包围起来，单独的一下使用两个$符号包围起来。
2. 希腊字母小写字母转义的时候第一个字母小写，大写的话，第一个字母大写。
3. 上标和下标分别使用^和_来标记。
4. 组：上标、下标或者其他的运算，仅仅是对接下来的组有效，一个组可以是一个单独的符号，也可以是任意被一对大括号包围起来的公式。
5. 括号：小括号和中括号直接表示，而大括号使用斜杠转义，或者英文转义`\lbrace和\rbrace`,而使用英文转义的括号会自动调整尺寸`\left \right`
6. 求和和积分：`\sum \int`,再加上上标和下标和组成你想要的的公式。
7. 分式：`frac{1}{2} a+1 over b+1`
8. 根号：`\sqrt \sqrt[3]{frac xy}`
9. 特殊函数`lim sim max ln`,例如：`\sin x  \lim_{x \to 0}`
10. 间隔：换行`\\`,窄间隔：`\,`
11. 重音符号和变音符号：`\hat \bar \vec \overline`

$$
\begin{align*}
y = y(x,t) &= A e^{i\theta} \\
&= A (\cos \theta + i \sin \theta) \\
&= A (\cos(kx - \omega t) + i \sin(kx - \omega t)) \\
&= A\cos(kx - \omega t) + i A\sin(kx - \omega t)  \\
&= A\cos \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big) + i A\sin \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big)  \\
&= A\cos \frac{2\pi}{\lambda} (x - v t) + i A\sin \frac{2\pi}{\lambda} (x - v t)
\end{align*}
$$

