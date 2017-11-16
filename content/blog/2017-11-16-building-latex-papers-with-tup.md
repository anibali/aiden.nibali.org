+++
description = """
  Building complex LaTeX documents can be quite painful. Thankfully,
  a little program called Tup can automate the build process for you."""
categories = ["misc"]
showpagemeta = true
slug = ""
date = "2017-11-16T08:09:03+11:00"
tags = ["tools", "latex", "tup", "git"]
comments = false
draft = false
title = "Building LaTeX papers with Tup"
+++

When writing a research paper, there are often two options for manuscript
creation: Microsoft Word and LaTeX.

In general I prefer to use LaTeX, because the idea of writing what you _mean_
and allowing a powerful typesetting engine produce a beautiful document
appeals to me. Furthermore, it is really useful being able to keep track
of the manuscript with version control. However, being over 30 years old at this
point, LaTeX is not without its warts.

One of the nastiest aspects of LaTeX is the build process. In Microsoft Word,
there really isn't a build step at all---what you see is really what you get.
If you need to submit the document somewhere, you can do so and be reasonably
confident that the recipient will see the same thing that you do.

LaTeX, in my experience, is a bit more complicated. Which LaTeX compiler
command should be used? Which figures need to be built before compiling the main
document? What are the dependencies between parts of the document?

In this post we will look at setting up automated builds for LaTeX documents.

## Automatic LaTeX builds with Tup

Let's say that you have a  project directory called `my_paper/` with a simple
LaTeX document called `main.tex` inside:

```tex
\documentclass{article}
\begin{document}
My document is so awesome!
\end{document}
```

We can compile this document into a PDF by running the following command:

```sh
$ latexmk -pdf main.tex
```

### Introducing Tup

[Tup](http://gittup.org/tup/) is a build system that fills a similar role
to [GNU Make](https://www.gnu.org/software/make/), but has some nifty
additional features.

To get started with Tup, create an empty file called `Tupfile.ini` file beside
`main.tex` so that Tup knows where the root of the project is, then run
`tup init`.

```sh
$ touch Tupfile.ini
$ tup init
```

Tup build rules are defined in Tupfiles. Make a `build/` directory, and create
a file called `Tupfile` within it.

```tupfile
: ../main.tex |> latexmk -pdf %f |> %B.pdf
```

This Tupfile has a single rule which says "take main.tex, run latexmk on it,
and we will get main.pdf". We can trigger the build by running Tup without any
arguments.

```sh
$ tup
```

In this case Tup produces the PDF, but does so begrudgingly---look at those
angry errors!

```plain
tup error: File XXX was written to, but is not in .tup/db. You probably should
specify it as an output
```

This is because latexmk writes to files other than main.pdf, and Tup needs to
know about _all_ outputs produced by a command. No problem, let's add them
in.

```tupfile
: main.tex |> latexmk -pdf %f |> %B.pdf %B.fls %B.log %B.aux %B.fdb_latexmk
```

Run `tup` again, and this time the errors should be gone.

Typing `tup` every time we edit a file is kind of annoying, but fortunately
we don't have to. Tup can monitor relevant files for us, and rerun the
appropriate rules when files change.

```sh
$ tup monitor -f -a
```

### Integration with Git version control

Let's add version control to our little project with Git.

```sh
$ git init
```

When we were using Tup earlier, it created a `.tup` folder to keep track of file
states. We should tell Git to ignore this folder.

```sh
$ echo ".tup/" >> .gitignore
```

Furthermore, we don't want to commit our generated PDFs, logs, and whatever else
to version control. Luckily, Tup makes it really easy to exclude generated
outputs from Git. Simply add the `.gitignore` directive to `Tupfile` like so:

```tupfile
.gitignore
: ../main.tex |> latexmk -pdf %f |> %B.pdf %B.fls %B.log %B.aux %B.fdb_latexmk
```

Now when we go to stage everything...

```sh
$ git add .
```

...we can see that the generated files in `build/` are ignored by Git.

```sh
$ git status
[...]
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

  new file:   .gitignore
  new file:   Tupfile.ini
  new file:   build/Tupfile
  new file:   main.tex
```

### Adding a Gnuplot figure

We can add as many rules as we like to our Tupfile. For example, say we want
to add a plot of a sine wave. Create a directory called `figures/` with a
file called `sine.gnuplot` inside.

```gnuplot
set terminal cairolatex input pdf size 8cm,6cm
set output 'sine.tex'
plot sin(x)
```

Add a new rule to `build/Tupfile` for compiling the Gnuplot figure. We will
make the rule general, so that Tup can compile any .gnuplot file added to the
`figures/` directory. We will also group the outputs into a group called
`<figs>`. We then add the `<figs>` group as an _order-only input_ for the
main.tex rule, so that Tup knows that the plots should be built before the
main document.

```tupfile
.gitignore
: foreach ../figures/*.gnuplot |> gnuplot %f |> %B.tex %B.pdf <figs>
: ../main.tex | <figs> |> latexmk -pdf %f |> %B.pdf %B.fls %B.log %B.aux %B.fdb_latexmk
```

Finally, we need to actually embed the plot in the main document, `main.tex`.
Note that we refer to the `sine.tex` as if it is in the current
directory, since all building occurs in `build/`.

```tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
My document is so awesome!

\input{sine.tex}
\end{document}
```

Now if you edit either `main.tex` or `sine.gnuplot`, the PDF will be
automatically rebuilt. Coupled with a nice PDF reader like Evince that refreshes
when the document changes, you have the output displayed instantly as you work.

### Adding a bibliography

Adding a bibliography works pretty much how you would expect.

`references.bib`

```bibtex
@article{awesome,
  author = {Doe, John},
  title = {How awesome things come to exist},
}
```

`main.tex`

```tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
My document is so awesome \cite{awesome}!

\input{sine.tex}

\bibliography{../references}
\bibliographystyle{ieeetr}
\end{document}
```

`build/Tupfile`

```tupfile
.gitignore
: foreach ../figures/*.gnuplot |> gnuplot %f |> %B.tex %B.pdf <figs>
: ../main.tex | <figs> |> latexmk -bibtex -pdf %f \
|> %B.pdf %B.fls %B.log %B.aux %B.fdb_latexmk %B.blg %B.bbl
```

## Conclusion

There you have it, a nice way of setting up automatic LaTeX builds. Building
the document from a fresh clone of the Git repository is as easy as running
`tup init && tup`!
