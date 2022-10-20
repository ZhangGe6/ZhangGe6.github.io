---
title: Latex cheat sheet
author: Zhang Ge
date: 2021-05-08 10:46:00 +0800
categories: [有趣/有用的资料, cheat sheets]
tags: [cheat sheets]
math: true
---

# 各种符号

可参考这篇[博客](https://snaildove.github.io/2017/08/20/LaTeX_symbols/)。

另外总结一些平时用到的符号：

- 组合数

  $$\tbinom{n}{r}$$形式：`\tbinom{n}{r}`

  $$C_n^r$$形式：`C_n^r`

- 分段函数

  ```latex
  f(x)= \begin{cases} 
  		0 & condition1 \\   
  		1 & condition2 
  	   \end{cases} 
  ```

  
  $$
  f(x)= \begin{cases} 0 & condition1 \\ 1 & condition2 \end{cases}
  $$

- 换行对齐

  ```
  \begin{aligend}
  	f(x) &= ax + b \\
  	g(x) &= cx + d
  \end{aligend}
  ```

  

- 范数

  $\Vert s \Vert_p$：`\Vert s \Vert_p`

# 图片

在document开始前，定义定义图片文件目录与扩展名
```latex
% 定义图片文件目录与扩展名
\graphicspath{ {figures/} }
\DeclareGraphicsExtensions{.pdf,.eps,.png,.jpg,.jpeg}

\begin{document}
...
...
\end{document}
```

## 单张图片

```latex
\begin{figure}[!htp]
    \centering
    \includegraphics[width=0.3\linewidth]{fig_path}  % [height=2cm]
    \caption{fig_caption}
    \label{fig:}
\end{figure}
```

## 两（多）图并排放置（不共用图编号）

```latex
% 两图并排放置（不共用图编号）
\begin{figure}[!htp]
    \begin{minipage}{0.48\textwidth}
      \centering
      \includegraphics[height=3cm]{fig_path}
      \caption{验电器示意}
      \label{fig:parallel1}
    \end{minipage}\hfill
    \begin{minipage}{0.48\textwidth}
      \centering
      \includegraphics[height=3cm]{fig_path}
      \caption{场景}
      \label{fig:parallel2}
\end{minipage}
\end{figure}
```

## 两图并排放置（共用图编号）

```latex
\usepackage{subcaption}  % package `subcaption` is needed

% 两图并排放置（共用图编号）
\begin{figure}[!htp]
  \centering
  \begin{subfigure}{0.3\textwidth}
    \centering
    \includegraphics[height=2cm]{sjtu-badge.pdf}
    \caption{校徽}
  \end{subfigure}
  \hspace{1cm}
  \begin{subfigure}{0.4\textwidth}
    \centering
    \includegraphics[height=1.5cm]{sjtu-name.pdf}
    \caption{校名。注意这个图略矮些，subfigure 中同一行的子图在顶端对齐。}
  \end{subfigure}
  \caption{包含子图题的范例（使用 subfigure）}
  \label{fig:subfigure}
\end{figure}
```

# 表格

[参考](https://blog.csdn.net/JueChenYi/article/details/77116011)

```latex
\begin{table}[!htp]
\centering
\caption{tab_caption}
\label{tab:tab_label}
    \begin{tabular}{|c| m{3cm}| m{5cm}| m{4cm}| }
    \hline
      &   &   & \\
    \hline
      &   &   & \\
    \hline
    \end{tabular}
\end{table}
```

# 双栏模板下，要插入跨栏图表

将

```latex
\begin{figure}
...
\end{figure}
```

替换为

```latex
\begin{figure*}
...
\end{figure*}
```

即可。表格table同理。

# 代码段

```latex
\begin{lstlisting}[language=Python]

\end{lstlisting} 
```

# 伪代码

[参考](https://blog.csdn.net/golden1314521/article/details/40923377)

# 其他

- 文字居中：`\centerline{}`

- 文字下划线：`\uline{}`

- 另起一页：`\clearpage`

- 公示中插入中文`\mbox{}`：`\sin(\mbox{狗})` $$\rightarrow$$ $$\sin(\mbox{狗})$$

- 创建新的格式（如加框文字）

  ```latex
  % 在\begin{document}之前
  \usepackage{environ}
  \usepackage{tikz}
  \NewEnviron{elaboration}{
  \par
  \begin{tikzpicture}
  \node[rectangle,minimum width=0.9\textwidth] (m) {\begin{minipage}{0.85\textwidth}\BODY\end{minipage}};
  \draw[dashed] (m.south west) rectangle (m.north east);
  \end{tikzpicture}
  }
  
  % 在正文中
  \begin{elaboration}
  what you write...
  \end{elaboration}
  ```

- 添加附录

  ```latex
  \usepackage[title]{appendix}
  
  \begin{document}
  ...
  
  \begin{appendices}
  \section{Some Notation}
  \end{appendices}
  
  \end{document}
  ```


- 手动缩小间距

  ```latex
  \vspace{-0.8cm}
  ```

- 参考文献中使用`first author et al.`格式（缩减参考文献篇幅）

  [参考](https://tex.stackexchange.com/a/123607)

  ```latex
  author={Alpher, R. and others}
  ```

- 参考文献显示问号的解决

  [参考](https://blog.csdn.net/blgpb/article/details/84885762)
