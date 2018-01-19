---
layout: post
title:  "减肥－我的人体实验与统计分析 – 照片,腰围,胸围,体重,体脂…（最多3个月30磅）2012年2月9日更新"
date:   2012-02-02
categories: OS
---

2011年暑假，吃多了。5月份从学校宿舍出来住，终于不用在食堂吃了，所以每天开心的..几乎每天两瓶啤酒，然后吃很多。其实在宿舍住的时候也吃不少，也常喝啤酒。反正，体重是到了180 lbs（163斤）了。是历史上最重的一次了。

以前有几次也是类似的情况，大多数是在放假回家的时候吃多了，然后洗澡的时候实在看不下去了。就开始减肥。我专心地减肥大概有三次。前两次是暴力型的：
<ul>
	<li>每天3个小包子（学校边上小摊卖的那种，1块钱三个），或者最多再吃一个或两个苹果</li>
	<li>每天爬到岳麓山山顶，跳1000个绳</li>
</ul>
这样搞久了要死人的，每天感觉头被压住了一样。所以我一般一个“疗程”5~6天。隔一段时间再来一次。这样的话，每天能瘦大概1斤。2008年夏天，搞了两个疗程，瘦了10斤。跟人说起这事，总觉得别人的惊讶程度不够。我不开心。

后来又胖了一次，用了非暴力的方法来减。
<ul>
	<li>每天在食堂吃饭，吃三两（或二两）饭加一个荤菜。以前我都是吃一荤一素的。</li>
	<li>一周至少爬4次山。不跳绳。</li>
</ul>
这样的结果是2个月10斤吧。记不太清了，好像又是1个月10斤。

我把减肥当作一个项目来做的。这是一个在自己身上的实验。在这个实验中，我觉得有一个很大的问题就是：坚持了一段时间，摸摸自己身上的肉，感觉不出变瘦了，觉得没希望，就停下来了。或者哪天吃了顿大餐，过一两天一称，发现重了很多，就觉得反弹太快的，也就放弃了。

为什么玩游戏的人那么入迷哦，原因之后就是他们成就被量化了，而且有非常及时的feedback。你的成就可能是杀了几个人、得到了多少分、升级到了“泡泡导师”（我玩QQ泡泡龙的:)），等等。有及时的feedback。比如格斗游戏时，你能看到自己有多少血对方有多少血，你一个大招过去， 对方的血唰的一下往下降，你就开心了。TED上有个视频是说如何把游戏中的这些设计用在教学中的，挺好玩的。

2011年8月24号开始，我开始又一轮的减肥。与之前的不同的是，这次我每天记录自己的体重、腰围、胸围、体脂、饮食、运动，并照相。这就是我“血”。然后，我把这些数据导入到电脑时，用统计软件<a href="http://www.r-project.org/" target="_blank">R</a>绘图分析。我想找出是什么在影响我的体重、影响有多大。
<p style="text-align: center;"><img class="alignnone wp-image-845 size-full" src="http://www.manio.org/cn/wp-content/uploads/2011/10/facecompare-0907-1024.jpg" alt="" width="500" height="780" /></p>
<p style="text-align: center;"><em>9月7日和10月24日对比照（9月6号开始照相的，所以之前没有照片可比。这里用9月7号的照片是因为7号的开了闪光灯，效果好一些。</em></p>
两个月后，减了20磅，相当于15瓶娃哈哈纯净水的重量。
<p style="text-align: center;"><a href="http://www.manio.org/cn/the-art-of-losing-weight/wahaha2/" rel="attachment wp-att-879"><img class="alignnone" title="wahaha2" src="http://www.manio.org/cn/wp-content/uploads/2011/10/wahaha2-160x160.jpg" alt="" width="160" height="160" /></a></p>
在从8月24日到现在的时间里，我的减肥还是比较温和的：
<ul>
	<li>不吃米饭，只吃菜。不吃零食，不喝高能饮料。</li>
	<li>每天到健身房运动40分钟。</li>
	<li>每天做180（早90，晚90）个<a href="http://www.youtube.com/watch?v=tl_Hp1Wf52k">Ab wheel</a>(膝着地）用时十几分钟。</li>
</ul>
上面说的“不”和“每”不是绝对的，有时聚餐就吃的比较多（能从下面的曲线看出来，哪天要是体重暴增了，就知道那天有人请吃饭了）。。。有时喝多了，上床就睡了，第二天还要吐。。有时在外地。。这些情况会没法完成。不过，AB WHEEL只有两三次没做，健身房也只是两三次。
<p style="text-align: center;"><img class="aligncenter wp-image-850 size-full" title="workout-weight" src="http://www.manio.org/cn/wp-content/uploads/2011/10/workout-weight2.png" alt="" width="502" height="551" />此图显示的是运动的类型是如何影响体重的，体重的单位是磅(lbs)</p>
图中的小圆圈是我的测量值。直线是做linear regression得到的，它们能表示出体重下降的速度。

Elliptical+Spinning指的是40分钟连续运动，前20分钟是<a href="http://en.wikipedia.org/wiki/Elliptical_trainer">Elliptical</a>，后20分钟是<a href="http://en.wikipedia.org/wiki/Indoor_cycling">Spinning</a>。

Treadmill是跑步机。UpperBody指的是上半身的器械训练，时间在30~40分钟。这些都是在健身房做的，Ab wheel我是每天都一样。

从图中可以看到，跑步机的效果要好于spinning。做spinning的时候我很少出汗。但在跑步机上跑的时候出很多汗。

在做这些运动时，我的心跳一般都保持在120~140之间。
<p style="text-align: center;"><img class="aligncenter wp-image-851 size-full" title="workout-waist" src="http://www.manio.org/cn/wp-content/uploads/2011/10/workout-waist.png" alt="" width="502" height="551" /></p>
上图显示的是运动的类型是如何影响腰围的。可以看到Ellipical+Treadmill的效果要大大好于Ellipical+Spinning。
<p style="text-align: center;"><img class="aligncenter wp-image-852 size-full" title="workout-chest" src="http://www.manio.org/cn/wp-content/uploads/2011/10/workout-chest.png" alt="" width="521" height="523" /></p>
胸围也是在Ellipticals+Treadmill时下降得最快，如上图。

下面是数据的曲线图。（点击图片可看大图）
<p style="text-align: center;"><img class="aligncenter wp-image-855 size-full" title="waist-chest" src="http://www.manio.org/cn/wp-content/uploads/2011/10/waist-chest.png" alt="" width="826" height="554" /></p>
<p style="text-align: center;"><img class="aligncenter wp-image-856 size-full" title="waistfat" src="http://www.manio.org/cn/wp-content/uploads/2011/10/waistfat.png" alt="" width="826" height="554" /></p>
<p style="text-align: center;"><img class="aligncenter wp-image-857 size-full" title="weight" src="http://www.manio.org/cn/wp-content/uploads/2011/10/weight.png" alt="" width="826" height="554" /></p>
<p style="text-align: center;"><img class="aligncenter wp-image-862 size-full" title="IMG_8677" src="http://www.manio.org/cn/wp-content/uploads/2011/10/IMG_8677.jpg" alt="" width="480" height="640" /></p>
<p style="text-align: center;">十月份做记录的纸。</p>
<p style="text-align: left;">现在我开始每天游泳1小时。记录中...</p>
现在总结的几个减肥的要点是：
<ul>
	<li>及时的feedback（体重，腰围，体脂，照片）。feedback能让你及时调整策略，并给你坚持下去的成就感</li>
	<li>show off。 如此文，这是另一个动力。</li>
	<li>节食（我认为比运动更关键）</li>
	<li>运动</li>
</ul>

<hr />

&nbsp;
<h2>2012年2月更新</h2>
2011年12 月11日~2012年1月11日在中国，堕落得无法记录.. 那堕落的生活让体重一个月暴长...

<img class="aligncenter wp-image-893 size-full" title="Feb-8 Weight" src="http://www.manio.org/cn/wp-content/uploads/2011/10/Feb-8-Weight.png" alt="" width="1117" height="586" />体重曲线

<img class="alignnone wp-image-894 size-full" title="Feb-8 Waist" src="http://www.manio.org/cn/wp-content/uploads/2011/10/Feb-8-Waist.png" alt="" width="1117" height="586" />
腰围曲线

<img class="alignnone wp-image-897 size-full" title="Feb-front" src="http://www.manio.org/cn/wp-content/uploads/2011/10/Feb-front.jpg" alt="" width="500" height="517" />

<img class="alignnone wp-image-899 size-full" title="Feb" src="http://www.manio.org/cn/wp-content/uploads/2011/10/Feb.jpg" alt="" width="400" height="411" />

&nbsp;
