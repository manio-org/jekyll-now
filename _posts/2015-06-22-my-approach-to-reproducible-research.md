---
layout: post
title:  "My Approach To Reproducible Research"
date:   2015-06-22 16:16:01 -0600
categories: science
---





The goal is simple. During my research, I often need to run a lot of different workloads, plot the results and write some analysis text. My goal is to:
- find out parameters I used to generate the results. So I can answer questions "why I see an outlier in my figure?"
- find out the script that I used to generate the plot. So I can improve the figure for publication.
- rerun the whole program and get the results. So I can produce new results by changing the old one. 

To do so, I use:
- Github
- R Markdown (RStudio)
- `Makefile.py`
- `get_github_url.py`
- `analyzer.r`
- `download_github_private_file.py`
- `source_private_github_file.r`

[https://github.com/junhe/reproducible-research-template](https://github.com/junhe/reproducible-research-template)

Here are some guidelines for myself.

## Manage all code by one github repository
Centralized management is easier. Using Git, you have all access to the history of all your code. 

## Never write commands in command line
If you write `./my-awesome-code parameter1 parameter2`, you will never find out what `parameter1` and `parameter2` were after two months.

## Put ALL scripts to `Makefile.py`
If you put your parameters and everything in a file like `Makefile.py`, you will be able to find out what you did in what day. You don't need to remember parameter except to run `./Makefile.py`. Don't use `./Makefile.py`'s arguments, for the same reason. 

## Use `get_github_url.py` to get plotting script
Currently, `get_github_url.py` snapshots the current code and put the following script to copyboard of Mac OS. 

``` r
# this requires curl installed in your OS
library(devtools)
source_url("https://gist.github.com/junhe/1f7e41f4c2829486e46f/raw/source_private_github_file.r")
source_private_github_file("doraemon", "analysis/analyzer.r", "599060f45d97538b9dffda4b54ab88d1e7eff006")
```

If you copy and paste the code above to R Mardown, it will source `analysis/analyzer.r` in project "doraemon", which contains the ploting script.

## Use organized script `analyzer.r` to plot
This template makes it easier to have reusable plotting code.

## Use R Markdown to integrate plots (as code chunk) and analysis text
This is literate programming. Code and analysis are together. This is the ultimate output of the project, where you can find insights.

## Put R Markdown files to Github repository
The Github repository, which will never be lost, will be the central place where you will find everything you need to reproduce the results months or years later. 
