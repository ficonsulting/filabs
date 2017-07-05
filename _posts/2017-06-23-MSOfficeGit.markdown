---
layout: post
comments: true
title:  "Microsoft Office and Git"
subtitle: "ExcelCompare, pandoc, and Python"
date: 2017-06-21
authors: jon_hill
header-img: "img/msofficegit.jpg"
permalink: /:title
---

This is a tutorial on how to configure Git to manage Microsoft Office files. It'll cover docx, xls, xlsm, xlsx, and pptx. I hope that it will give you a sense for how to customize Git to meet your needs, especially if you find it difficult to track what has changed in your Microsoft Office files.

Microsoft Office products are often frustrating for developers and data scientists to work with. They are messy zipped collections of xml that behave in strange ways, change your data, and pose a risk to the reproducibility of your work. If you have added ".zip" to the end of a Microsoft Office file, sorry. There is nothing useful in there. If you change one cell of a moderately complicated financial model, it will impact literally dozens of files under the hood, so tracking the xml in Git is not very informative. All you want to know is `Assumptions!A1 100 -> 105` and `Projections!A1 1500 -> 1575`.

This type of information is incredibly useful from an auditing, risk, and reproducibility standpoint. Unlike Sharepoint which tracks check-in and check-out activity, Git gives a complete history of line-by-line changes. You don't need to open every version of a file to determine how/when something changed. You can call `git diff` or query the Git log to answer many of your questions. Git also encourages a much more expressive account of how and why things have changed through commit messages. This ensures that your work is reproducible and auditable. Future you will be grateful. Plus, Git is fun. It saves you time (in the long run) and helps you manage risk, so you can relax a bit while you work with Microsoft Office files that change in unexpected ways.

The world of Git without Microsoft Office is a much simpler place. You can collaborate with large teams and merge changes with relative ease. You can branch and sandbox different versions without waiting for someone else to finish. It's a developer's paradise. That will never be true for your Microsoft Office files. A Git merge on a binary file will fail. That is why Sharepoint has a check-in and check-out style of collaboration, which involves a lot of waiting and much lower productivity levels. This tutorial will not unlock the full potential of Git with Microsoft Office files; they will still ruin your day once in a while. But for those of us who are stuck with Microsoft Office, this tutorial should help.

<small>**Note:** If Source Tree is your favorite Git GUI, they have made an opinionated stance against running `git diff` on binaries because, they are right, you should probably avoid including binaries in your repos if you can.</small>

# ExcelCompare

![cake](https://avatars0.githubusercontent.com/u/262532?v=3&s=400)

[na-ka-na](https://github.com/na-ka-na), A.K.A. Sanchay Harneja, is the author of [ExcelCompare](https://github.com/na-ka-na/ExcelCompare), a command-line tool for diffing Excel workbooks. If you tell Git to run it as a custom `diff` tool, Git will crack open the binary with ExcelCompare. This is much better than the default message:

{% highlight git %}
$ git diff
$ Binary files a/Book1.xlsx and b/Book1.xlsx differ
{% endhighlight %} 

First, [download](https://github.com/na-ka-na/ExcelCompare/releases) ExcelCompare. Place it in a path **without** spaces. You can escape spaces from Git commands, but no spaces in the path will just make your life easier. I put it in the root directory of my Git repo.

Next, update your .git/config and .gitattributes files to run a custom command on Excel files when running `git diff`. Git looks in .gitattributes for rules to apply its configuration, and you need to tell it to use the "excel" method on files with xls, xlsx and xlsm file extensions. Then in .git/config, tell Git what the "excel" method is.

### .gitattributes
If you don't have a .gitattributes file in your repo's root directory, create one with your favorite text editor. I'm a fan of NotePad++. Add these file extensions.
{% highlight git %}
*.xlsx diff=excel
*.xlsm diff=excel
*.xls diff=excel    
{% endhighlight %}

### .git/config
Inside the hidden folder, .git, open `config` with your text editor and add the following command to execute ExcelCompare as the "excel" method for `git diff`.
{% highlight git %}
[diff "excel"]
    command = full/path/to/excel_cmp.bat
    binary=true
{% endhighlight %}

Now let's try this again.

{% highlight git %}
$ git diff
$ DIFF   Cell at      Assumptions!A1 => '105'  v/s '100'
$ DIFF   Cell at      Projections!A1 => '1500' v/s '1575'
$ Excel files Book1.xlsx and C:\Users\jhill\AppData\Local\Temp\hckpya_Book1.xlsx differ
{% endhighlight %}

Awesome! If you want to know how a client's financial model has been impacted by an assumption change, just run `git diff` after saving it. This is incredibly useful when you have lots of tabs and you are not sure how they all relate, and many other excel comparison tools force you to manually save different versions of the same file which increases the risk that the wrong version gets used or changes get lost in the mix. Now, you can work on the same file and commit your changes. Fun, right? Wrong. ExcelCompare slows down your `git diff` calls, so I am not completely sold. If someone knows of a faster tool, please comment!

# Word to Markdown
Pandoc is a great tool for moving between HTML, docx, pdf, LaTeX, etc. You can use it to convert .docx files into markdown because that is a simple format that Git can work with. The default `git diff` on docx will tell you that an entire line has changed but leave it to you to figure out which words differ. When tracking your writing/edits, it would be nice to track changes at a more granular level. The markdown versions of docx will help you zero in on specific words and highlight them with a custom diff.

### .gitattributes
Add the file extension to .gitattributes.
{% highlight git %}
*.docx diff=word    
{% endhighlight %}

### .git/config
Now, tell git to use Pandoc to convert your word docx into markdown.
{% highlight git %}
[diff "word"]
    textconv=pandoc --to=markdown
    binary=true
    prompt=false
[alias]
    wdiff = diff --word-diff=color --unified=1
{% endhighlight %}

Then, run `git wdiff mydoc.docx`, and you will see additions in green and deletions in red on a word-by-word basis. You can also look at the full history of a file with `git log -p --word-diff=color mydoc.docx`.

# Power Point and Python
I wrote a quick Python script to process Power Point files because there is no equivalent to ExcelCompare for Power Point presentations. Honestly, I'm not sure there should be... but it was fun for the purpose of demonstration here. This script requires Python3 and python-pptx. If you have Python3, you can install python-pptx with `pip install python-pptx`.

### pptx-textconv.py
{% highlight python %}
import sys
from pptx import Presentation

path_to_presentation = sys.argv[1]

prs = Presentation(path_to_presentation)

# text_runs will be populated with a list of strings,
# one for each text run in presentation
text_runs = []

for slide in prs.slides:
    for shape in slide.shapes:
        if not shape.has_text_frame:
            continue
        for paragraph in shape.text_frame.paragraphs:
            for run in paragraph.runs:
                text_runs.append(run.text)
print(text_runs)
{% endhighlight %}

You know the drill at this point...

### .gitattributes
Add the file extension to .gitattributes.
{% highlight git %}
*.pptx diff=powerpoint    
{% endhighlight %}

### .git/config
Now, tell git to use the Python script to convert your presentation into text.
{% highlight git %}
[diff "powerpoint"]
    textconv=python full/path/to/pptx-textconv.py
    binary=true
{% endhighlight %}

This will give you a dictionary comparison of your Power Point presentation's text, and it works well with the alias `wdiff` you created earlier. Try out `git wdiff MyPres.pptx` to see what has been added or deleted from a presentation.

# Conclusion
Microsoft Office files will find their way into your repos. Now you can dig into your .gitattributes file and .git/config settings to track them appropriately. These instructions are at the repo level, but you can just as easily apply them to your global .gitconfig and make it the default for all projects. If you have questions or comments, please join the discussion below! 