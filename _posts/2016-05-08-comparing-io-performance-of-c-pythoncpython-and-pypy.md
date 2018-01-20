---
layout: post
title:  "Comparing IO Performance of C++, Python(CPython) and Pypy"
date:   2016-05-08 16:16:01 -0600
categories: python IO systems
---


Simple, access files with the same IO patterns but different languages (C++, Python) and see the difference. 

<!--<img src="../images/comparecpppython.png" alt="comparecpppython" width="641" height="405" class="aligncenter size-full wp-image-1068" />-->
![image]({{ "/assets/images/comparecpppython.png" | absolute_url }})

The performance depends on the pattern. It could be c++ < (less time than) pypy < python, or c++ = pypy = python, or python < pypy < c++, or ... For example, with pattern SeqSmallWriteNoFsync, python < pypy < c++. Python might have cached write without actually making the system calls. 

Source code at: <a href="https://github.com/junhe/py-cpp-io-perf" target="_blank">https://github.com/junhe/py-cpp-io-perf</a>

R Script to plot:

```r
# ssd, python 2.7
d.text.ssd="language pattern time
py RandSmallWriteFsync 1.53192591667
py SeqSmallWriteNoFsync 0.132789850235
py SeqSmallWriteFsync 1.51046991348
py SeqSmallRead 0.417271852493
py SeqBigRead 0.44149184227
py RandSmallRead 2.42655897141
py RandSmallWriteFsync 1.53023505211
py SeqSmallWriteNoFsync 0.132742881775
py SeqSmallWriteFsync 1.49960803986
py SeqSmallRead 0.388828992844
py SeqBigRead 0.433161973953
py RandSmallRead 2.42879295349
cpp RandSmallWriteFsync 1.308090
cpp SeqSmallWriteNoFsync 0.84903
cpp SeqSmallWriteFsync 1.326069
cpp SeqSmallRead 0.399079
cpp SeqBigRead 0.422075
cpp RandSmallRead 2.246225
cpp RandSmallWriteFsync 1.304056
cpp SeqSmallWriteNoFsync 0.87404
cpp SeqSmallWriteFsync 1.325295
cpp SeqSmallRead 0.387681
cpp SeqBigRead 0.413929
cpp RandSmallRead 2.184642
py RandSmallWriteFsync 1.53299999237
py SeqSmallWriteNoFsync 0.130995988846
py SeqSmallWriteFsync 1.50311684608
py SeqSmallRead 0.310713768005
py SeqBigRead 0.416656970978
py RandSmallRead 2.31285500526
cpp RandSmallWriteFsync 1.302690
cpp SeqSmallWriteNoFsync 0.84644
cpp SeqSmallWriteFsync 1.323407
cpp SeqSmallRead 0.394427
cpp SeqBigRead 0.417460
cpp RandSmallRead 2.228274
py RandSmallWriteFsync 1.51506304741
py SeqSmallWriteNoFsync 0.134629011154
py SeqSmallWriteFsync 1.47410011292
py SeqSmallRead 0.336603879929
py SeqBigRead 0.408558130264
py RandSmallRead 2.27934789658
py RandSmallWriteFsync 1.53826785088
py SeqSmallWriteNoFsync 0.132005929947
py SeqSmallWriteFsync 1.49633002281
py SeqSmallRead 0.343593120575
py SeqBigRead 0.416095972061
py RandSmallRead 2.34551906586
cpp RandSmallWriteFsync 1.312224
cpp SeqSmallWriteNoFsync 0.83977
cpp SeqSmallWriteFsync 1.326887
cpp SeqSmallRead 0.382488
cpp SeqBigRead 0.406615
cpp RandSmallRead 2.207618
cpp RandSmallWriteFsync 1.281245
cpp SeqSmallWriteNoFsync 0.80139
cpp SeqSmallWriteFsync 1.301576
cpp SeqSmallRead 0.374503
cpp SeqBigRead 0.405859
cpp RandSmallRead 2.188872
py RandSmallWriteFsync 1.52480697632
py SeqSmallWriteNoFsync 0.131967067719
py SeqSmallWriteFsync 1.49699020386
py SeqSmallRead 0.309700012207
py SeqBigRead 0.422698020935
py RandSmallRead 2.32114505768
cpp RandSmallWriteFsync 1.310140
cpp SeqSmallWriteNoFsync 0.79494
cpp SeqSmallWriteFsync 1.309160
cpp SeqSmallRead 0.392876
cpp SeqBigRead 0.418434
cpp RandSmallRead 2.215911
pypy SeqSmallWriteNoFsync 0.244556903839
pypy RandSmallWriteFsync 1.38171887398
pypy SeqSmallWriteFsync 1.38917589188
pypy SeqSmallRead 0.388851165771
pypy SeqBigRead 0.424990177155
pypy RandSmallRead 2.22544789314
cpp SeqSmallWriteNoFsync 0.87881
cpp RandSmallWriteFsync 1.273606
cpp SeqSmallWriteFsync 1.322351
cpp SeqSmallRead 0.394807
cpp SeqBigRead 0.419752
cpp RandSmallRead 2.220217
cpp SeqSmallWriteNoFsync 0.79990
cpp RandSmallWriteFsync 1.282682
cpp SeqSmallWriteFsync 1.322388
cpp SeqSmallRead 0.397584
cpp SeqBigRead 0.420078
cpp RandSmallRead 2.208946
pypy SeqSmallWriteNoFsync 0.209247112274
pypy RandSmallWriteFsync 1.34968805313
pypy SeqSmallWriteFsync 1.36747908592
pypy SeqSmallRead 0.403924942017
pypy SeqBigRead 0.422988176346
pypy RandSmallRead 2.22062897682"

# pypy virtual machine /tmp
d.text.tmp.pypy="language pattern time
pypy SeqSmallWriteNoFsync 0.162170886993
pypy RandSmallWriteFsync 20.8120908737
pypy SeqSmallWriteFsync 20.8292589188
pypy SeqSmallRead 0.662232875824
pypy SeqBigRead 0.483371019363
pypy RandSmallRead 6.84131407738
cpp SeqSmallWriteNoFsync 0.177787
cpp RandSmallWriteFsync 16.896599
cpp SeqSmallWriteFsync 15.543781
cpp SeqSmallRead 0.585383
cpp SeqBigRead 0.597393
cpp RandSmallRead 6.622789
cpp SeqSmallWriteNoFsync 0.156806
cpp RandSmallWriteFsync 15.904151
cpp SeqSmallWriteFsync 14.342114
cpp SeqSmallRead 0.559932
cpp SeqBigRead 0.602838
cpp RandSmallRead 5.818669
pypy SeqSmallWriteNoFsync 0.182167053223
pypy RandSmallWriteFsync 16.6570448875
pypy SeqSmallWriteFsync 15.632062912
pypy SeqSmallRead 0.536597967148
pypy SeqBigRead 0.384449005127
pypy RandSmallRead 5.90277290344
py SeqSmallWriteNoFsync 0.16601896286
py RandSmallWriteFsync 16.3286008835
py SeqSmallWriteFsync 16.409607172
py SeqSmallRead 0.599189043045
py SeqBigRead 0.388226032257
py RandSmallRead 5.94078016281
cpp SeqSmallWriteNoFsync 0.159943
cpp RandSmallWriteFsync 16.297557
cpp SeqSmallWriteFsync 15.545674
cpp SeqSmallRead 0.632510
cpp SeqBigRead 0.654464
cpp RandSmallRead 5.782673
cpp SeqSmallWriteNoFsync 0.161760
cpp RandSmallWriteFsync 16.15376
cpp SeqSmallWriteFsync 15.363488
cpp SeqSmallRead 0.621055
cpp SeqBigRead 0.580540
cpp RandSmallRead 5.865869
py SeqSmallWriteNoFsync 0.168658971786
py RandSmallWriteFsync 16.7761938572
py SeqSmallWriteFsync 16.9714078903
py SeqSmallRead 0.5575299263
py SeqBigRead 0.377434015274
py RandSmallRead 6.16861820221"

plot_perf {
d = read.table(text=d.text, header=T)
p = ggplot(d, aes(x=pattern, y=time, color=language)) +
geom_point(size=5, alpha=0.5) +
ggtitle(title) +
expand_limits(y=0) +
ylab("Duration (sec)") +
theme(axis.text.x = element_text(angle=90, hjust=1, vjust=0.5, size=14))
return(p)
}

p1 = plot_perf(d.text.tmp.pypy, 'virtual machine, ssd, /tmp')
p2 = plot_perf(d.text.ssd, 'datacenter node, ssd, /dev/sdc')
do.call('grid.arrange', c(list(p1, p2), ncol=2))
```
