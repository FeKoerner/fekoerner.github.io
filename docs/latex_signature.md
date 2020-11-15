# Latex Signature

!!! summary
    This article explains how to digitaly sign a latex document using a linux system.

For this you will need a picture of your signature.
Our picture of the signature is called `signature.jpg`.

* convert jpg to bmp  
`mogrify -format bmp signature.jpg`
* trace signature.bmp with potrace  
`potrace signature.bmp -b svg -o signature.svg`
* conver signature.svg to png  
`mogrify -format png signature.svg`

## latex test document:

```latex
\documentclass{article}
\usepackage{graphicx}% http://ctan.org/pkg/graphicx
\setlength{\parindent}{0pt}% Remove paragraph indent
\begin{document}
\hspace*{0.5\linewidth}
\begin{minipage}{0.4\linewidth}
Random Institute \par
Random City 1000 \par
Randomia \par
\today
\end{minipage}

To whom it may concern: \par \bigskip

Hire me, it'll be worth your while. \par \bigskip

Sincerely, \par \medskip

\includegraphics[height=1.5\baselineskip]{signature.png} \par
Random Randofsky \par
Randomville
\end{document}
```