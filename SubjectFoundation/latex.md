##### 伪代码例子

```
\def\SetClass{article}
\documentclass{\SetClass}
\usepackage[linesnumbered,lined,boxed,commentsnumbered]{algorithm2e}
\begin{document}
\IncMargin{1em}
\begin{algorithm}
  \SetKwData{Left}{left}\SetKwData{This}{this}\SetKwData{Up}{up}
  \SetKwFunction{Union}{Union}\SetKwFunction{FindCompress}{FindCompress}
  \SetKwInOut{Input}{input}\SetKwInOut{Output}{output}

  \Input{An array $iocs$ of size $l1.A \  hashTable \ $ips$  \ of \  size \  $l2.iocs is the raw ioc,and ipList is the friendly list of ip.}
  \Output{An array that exlude Non-malicious ip}
  \BlankLine
  \emph{test}\;
  \For{$i\leftarrow 1$ \KwTo $l1$}{
    \emph{if ips[]}\;
    ioc=iocs[i];
    
  }
  \caption{disjoint decomposition}\label{algo_disjdecomp}
\end{algorithm}\DecMargin{1em}
\end{document}
```

```
\def\SetClass{article}
\documentclass{\SetClass}
\usepackage[lined,boxed,commentsnumbered]{algorithm2e}
\begin{document}
\begin{algorithm}[H]
  \SetAlgoLined
  \KwData{this text}
  \KwResult{how to write algorithm with \LaTeX2e }

  initialization\;
  \While{not at end of this document}{
    read current\;
    \eIf{understand}{
      go to next section\;
      current section becomes this one\;
      }{
      go back to the beginning of current section\;
      }
    }
  \caption{How to write algorithms}
\end{algorithm}
\end{document}
```

