---
title: Progress Bars in Jupyter Notebooks
subtitle: When will it end?
date: 2017-11-21T00:00:00.000Z
summary: ""
draft: false
featured: false
authors:
  - Charles
lastmod: 2020-12-13T00:00:00.000Z
tags:
  - programming
categories: []
projects: []
image:
  caption: ""
  focal_point: ""
  placement: 2
  preview_only: false
---
When working on a deep learning project for one of my classes, I found myself wanting a progressbar for the training of a RNN. I had used them for other tasks in the past via [tqdm](https://github.com/tqdm/tqdm) or [progressbar2](https://pypi.python.org/pypi/progressbar2). This is actually a fork of an older, no longer maintained package: [progressbar](https://pypi.python.org/pypi/progressbar). I assume that's the reason for why you import the module as `import progressbar` and not `import progressbar2`. Anyways, I just did what I had done before: 

```python
import time
import progressbar

pb = progressbar.ProgressBar()
for i in pb(range(10)):
    time.sleep(0.5)
```

Now while this works beautifully if you have a simple function in the loop as above, things get messy when you want to print things inside of the loop. Say you want to print the validation accuracy every couple epochs, and have the progress bar tracking the overall progress of the training. If you try with the above method, the progress bar will be printed, then your text, then a piece of the progress bar ... you get the idea, it's pretty horrible. Luckily, the guys from progressbar2 though of this, and provide us with the following:

```python 
import time
import progressbar

pb = progressbar.ProgressBar(redirect_stdout=True)
for i in pb(range(10)):
    print("By the way, we're on iteration ", i)
    time.sleep(0.5) 
```

If you run this code, you'll see that the progress bar stays at the bottom and the text you want to print shows up above. The way this works is a little hacky, and if you know a little about how the standard io works in Unix you already know why: once a line is printed, you can't go back up and change it (that is, without resorting to low level console stuff like curses). 
The workaround it to intercept the output you want to display. It then prepends your output with a carriage return `\r`. This returns the cursor to the start of line, allowing you to write *over top* existing text. If you've never seen it before, try `print("blue is the best color\r red")` to see for yourself. It also pads your text to the right with whitespace (if need be) to overwrite (pun kindof intended) the previously printed text. If it needs to update the progress bar, it prints it
without the newline character at the end, (as in the usual case), to allow for overwriting of the updated progress bar at the next change.
This is a neat trick, and works well if you're okay with the progress bar being at the bottom.

This is all well and good if not for the fact that Jupyter Notebooks are not terminal emulators and don't support the carriage return. bummer.  

The solution I found was to use Jupyter's HTML support, and I ended up forking and tuning a short routine that you can just copy paste into your notebook. The link to the github repo is [here](https://github.com/metaember/log-progress). It uses widgets, so it's not longer text-based. In a notebook, that's probably okay. You can call it like this:

```python
for i in log_progress(range(10)):
    print ("Look at me!", i)
    time.sleep(0.5)
```

Now it works like a charm, and has no additional dependencies other than a recent version of Jupyter Notebook. Neat!
It should be mentioned however, that this simple code lacks some of the niceties of the aforementioned packages, such as ETA calculation and the like. But hey, it works!
