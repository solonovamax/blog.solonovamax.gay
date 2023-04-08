---
title: "Breadcrumb LaTeX package"
description: "Improved version of the breadcrumb LaTeX package"
summary: "Improved version of the breadcrumb LaTeX package"
tags: 
- latex
- latex package
draft: false
date: 2023-02-11T11:15:22-05:00
lastmod: 2023-04-08T16:16:18-04:00
---

{{<katex>}}

{{<lead>}}
This is modified code from SeanAllred on [StackExchange](https://tex.stackexchange.com/a/122816).
Any code in this post is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/), as-per StackExchange's license.
{{</lead>}}

"Breadcrumb" is a \\(\LaTeX\\) package that adds breadcrumb navigation at the top of each page in a document.
This package was originally created by vermiculus (Sean Allred), and then modified by egreg with further modifications from solonovamax incorporating contributions Heiko Oberdiek.

I spent quite a bit of time hacking this together, and I wanted to publish it somewhere before it gets lost to the depths and never used by anyone other than me. (which it probably will be anyways, but whatever. At least it's out there now.)

To use it, first create a file called `breadcrumb.sty`. Then, inside the `breadcrumb.sty` paste the code shown below.
Finally, add `\pagestyle{breadcrumb}` to the top of your document.

The separator used between the breadcrumbs can be changed by modifying the `\DeclareRobustCommand*{\BreadSep}` line(s).
By default, the separator is a right-pointing single angle quotation mark.

By adjusting the macro `\labelize`, at the end of the package file, you can modify which document sectioning commands have the label hack applied to them.

Everything added by me, solonovamax, is annotated with `-s`.

`breadcrumb.sty`:
```latex
%! suppress = EscapeHashOutsideCommand
%! Package = breadcrumb
%! Author = vermiculus (Sean Allred), egreg, solonovamax, Heiko Oberdiek
%! Date = 2021-04-06

% Credits:
%  - vermiculus: Put things together & made shit work
%  - egreg: Contributed logic improvements
%  - solonovamax: added some minor tweaks + doc comments. All comments by solonovamax are signed with ``-s'' at the end.
%  - Heiko Oberdiek: 'stole' their code for the seperator from their StackExchange answer, since I [solonovamax] think it looks nicer.

\NeedsTeXFormat{LaTeX2e}
\ProvidesPackage{breadcrumb}[2021/04/05]

\RequirePackage{xparse}
%\RequirePackage{everypage}
\RequirePackage{atbegshi}
\RequirePackage{fancyhdr}

% If you want to change the header style, modify this command: -s
%\DeclareRobustCommand*{\BreadSep}{%
%    ~\ensuremath{\rightarrow}~%
%}
% -s
\DeclareRobustCommand*{\BreadSep}{      % This seperator is 'stolen' from Heiko Oberdiek. -s
        {\large\textbf{\guilsinglright}}\hspace{0pt}\,%
}

\let\myhook\AtBeginShipout % playing with the two to see if I can get the desired output
\myhook{\pagestyle{breadcrumb}}

% All credit to egreg for the following generalization: (#122823)
% My [vermiculus's] only additions were the booleans and related stuff.
\ExplSyntaxOn
% Defines macro that allows you to hack the document sectioning commands. -s
\NewDocumentCommand{\labelize}{mm}{\breadcrumbs_labelize:Nn#1{#2}}
\newcommand{\breadcrumbtitle}{Breadcrumb Title}

\cs_new_protected:Npn\breadcrumbs_labelize:Nn#1#2{
    % LaTeX3-ify
    \bool_new:c{g_breadcrumbs_in_#2}
    \myhook{
        \bool_gset_false:c{g_breadcrumbs_in_#2}
    }
    \cs_set_eq:cN{original_\cs_to_str:N#1}#1
    \RenewDocumentCommand#1{som}{% Put page style stuff here
        \bool_gset_true:c{g_breadcrumbs_in_#2}
        \thispagestyle{#2:style} % This is redundant now, but didn't do much before, anyways.
        %          (It only affected the \section command, since \part and \section print a new page at the start of the macro.)
        %          -s
        %\IfBooleanTF{##1}{
        \use:c{original_\cs_to_str:N#1}*{##3}
    }{
        \IfNoValueTF{##2}
        {\use:c{original_\cs_to_str:N#1}{##3}}
        {\use:c{original_\cs_to_str:N#1}[##2]{##3}}
        %
        \label{#2:\use:c{the\cs_to_str:N#1}}%
        \thispagestyle{#2:style}% Set the style of this page, because SOME things (looking at you, \chapter) reset the style. -s
        \pagestyle{#2:style}% Set the style of all future pages. -s
        %  This means:
        %  All future pages will have the right level        %  BUT this affects the entire document.
        %  -s
        %}
    }
}

% Much nicer syntax.  Thanks, egreg!
% page styles are defined in the syntax: ``breadcrumb:[style name]:style''. You can copy & paste to define any new ones. -s
% \fancypagestyle{breadcrumb:part:style}{
%     \fancyhead{} % reset header -s
%     \chead{\large\nameref{breadcrumb:part:\thepart}}
% }%
% \fancypagestyle{breadcrumb:chapter:style}{
%     \fancyhead{} % reset header -s
%     \chead{%
%         \large\nameref{breadcrumb:part:\thepart}%
%         \BreadSep\nameref{breadcrumb:chapter:\thechapter}%
%     }
% }%
\fancypagestyle{breadcrumb:section:style}{
    \fancyhead{} % reset header -s
    \chead{%
        \large\breadcrumbtitle%
        \BreadSep\nameref{breadcrumb:section:\thesection}%
    }
}
\fancypagestyle{breadcrumb:subsection:style}{
    \fancyhead{} % reset header -s
    \chead{%
        \large\breadcrumbtitle%
        \BreadSep\nameref{breadcrumb:section:\thesection}%
        \BreadSep\nameref{breadcrumb:subsection:\thesubsection}%
    }
}
\fancypagestyle{breadcrumb}{
    \fancyhead{} % reset header -s
    \chead{%
        \large\breadcrumbtitle%
        \BreadSep\nameref{breadcrumb:section:\thesection}%
        \BreadSep\nameref{breadcrumb:subsection:\thesubsection}%
    }
}

% This hacks the \part, \chapter, and \section commands. If you want more levels, define the new page styles and then labelize them.
% Uncommend these if you need \part or \chapter
% \labelize{\part}        {breadcrumb:part}
% \labelize{\chapter}     {breadcrumb:chapter}
\labelize{\section}       {breadcrumb:section}
\labelize{\subsection}    {breadcrumb:subsection}
%\labelize{\subsubsection} {breadcrumb:subsubsection}

\ExplSyntaxOff
```
