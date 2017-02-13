---
date:        "2013-04-04"
title:       "LaTeX and Git in an Academic Environment"
# description: "How to make writing a thesis eaiser with Git and Latex"
tags:        ["Latex", "git"]
topics:      ["Scripts"]
slug:        "latex-with-git"
---

I hate word processors, I end up spending more time trying to figure out how to adjust the margins and get the graphs to look right then I spend writing the paper.  I am thrilled that M$ Word will do nearly everything for me, but I have no desire to spend twenty minutes trying to find the right wizard.  Content should always be separated from presentation, this is especially true when working on very long documents or any complex documents that contain multiple images, graphs, or mathematical and engineering notation.  While word processors can handle these scenarios, it is far more frustrating and the finished product will never look as good as a professionally typeset document.

Just using LaTex though is not enough, when working on large documents it helps greatly to be able to visualize the document growing and if necessary roll back any changes made.  This becomes crucial when writing a dissertation, not only may you want to work on it from multiple locations but many professors will be constantly giving you corrections and advice.  By using branches you can make life much simpler.  I am going to assume you have a basic understanding of git from this point forward, if you need a refresher course then take a look [here][1].   Creating an initial repo leaves you with a master branch, this should be dedicated to only a finished product that is ready for release, that way you always have a solid baseline to work with.  From here I would create a branch for drafts

{{< highlight text >}}

git -b checkout drafts

{{< /highlight >}}

This will create the branch and check it out at the same time.  Now create all the content to your hearts desire.  As you complete sections you can merge them back into the master branch with

{{< highlight text >}}

git checkout master
git merge drafts

{{< /highlight >}}

As long as you have committed everything correctly this should merge with no problems.  Now, when you go to hand this into your advisers and they decide to make some corrections you don't want to add these directly into the final copy nor do you want to add them to the draft copy you are keeping.  In this case you would create another branch titled adviser.

{{< highlight text >}}

git -b checkout adviser
    
{{< /highlight >}}

This will allow you to work with the current draft as it was handed in and incorporate the recommended changes.  At this point you will then be able to work forward with these changes, if you decide at a later time that further research warrants the removal of these changes you can do so without affecting the master branch or the draft you handed it as a specific stage, then when necessary you can merge those changes into the master branch.  Using this method you can keep branching out with each additional set of corrections and still maintain control over your work in the master and draft branches.

Another useful aspect of branches is the ability to split a very large project into logical sections, like chapters in a book.  Each chapter would be its own branch in this case, then as you complete a chapter you can merge it into the master.  On thing to keep in mind as well is that any document wide changes, especially ones to the preamble should be made on their own.  This will save a lot of headache if you need to change something later on down road.  Remember commit early and commit often.

These are all good ideas, but in order to fully utilize them effectively you are going to have to change your methodology some.  Git is designed for source code, which is generally line by line, when writing a document though we tend to write in paragraphs.  This will not work very well with git due to the fact that when you make a change to a sentence it will be detected as a change to the entire paragraph.  Latex, by nature, will handle all spacing so placing each sentence on it's own line will pose no problem from a typesetting point of view.  Many people tend to do their proofreading with the compiled document anyway so once you get used to hitting the Enter key after every sentence, you will be all set.

A great advantage to git as well is the built in [diff][2] command that will take two sources and compare them highlighting the changes.  There is also a way to do this with the compiled version using [latexdiff][3].  This tool will allow you to track changes across the compiled versions saving you some headache and eyestrain.

You may also want to track your revision number, author, or other information on the compiled document as well.  This may come in handy when submitting several versions of a nearly identical document.  Using the [vc][4] bundle, you can extract much of the metadata from your commits and place it into a compiled document.  Unfortunately unlike subversion git will not modify the source code in any way so directly placing the data in with tags is impossible.

{{< highlight text >}}

immediate\write18{sh ./vc}
input{vc}
usepackage{fancyhdr}
pagestyle{fancy}
fancyfoot[LE,LO]{Git Rev: \VCRevision}
fancyfoot[RE,RO]{Date: \GITAuthorDate}

{{< /highlight >}}

This block, placed in the document preamble, will execute the vc script and input it into the document.  I use the [fancy header][5] package to add footers with the revision number and date of the commit.  The vc documentation goes into more detail but you can pull almost anything from the commit with this package.  In a future post I will show a working example of this in a template for an apa6 style research paper including a bibliography and will talk a little more about customizing and extending latex using lua.

[1]: http://git-scm.com/book
[2]: https://www.kernel.org/pub/software/scm/git/docs/v1.7.3/git-diff.html
[3]: https://www.inf.unibz.it/dis/wiki/doku.php?id=latex%3adiff
[4]: http://www.ctan.org/tex-archive/support/vc
[5]: ftp://www.tug.org/tex-archive/macros/latex/contrib/fancyhdr/fancyhdr.pdf