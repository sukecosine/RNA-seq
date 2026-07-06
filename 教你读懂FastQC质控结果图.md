# 教你读懂FastQC质控结果图

<br/>测序数据质量关系到我们下游分析，在转录组分析流程（一）|数据质控文章中已经讲了质控软件FastQC的使用方法，运行后会生成两个文件，一个.html文件，一个是.zip文件。使用浏览器打开html文件唰唰唰出来一堆图，看的脑壳疼，今天就讲讲怎么看懂这些图。</br>
<br/>首先欣赏一下总的结果图，绿色的代表“PASS”，黄色的代表“WARNING”，红色就是失败了。打❌的一般是重点关注对象，今天每一项都给大家叨叨一遍。</br>

![image](https://github.com/sukecosine/picture/blob/master/1.png)

（一）	Basic Statistics

![image](https://github.com/sukecosine/picture/blob/master/2.png)

这是基本统计信息，了解自己的数据得从头开始，很明显主要有以下几点：
Filename：进行质控的原始文件名；
Encoding：指测序平台的版本及编码版本号；
Total Sequences：指reads的总数量；
Sequence Length：序列长度；
%GG：所有序列中总GC含量；
（二）	Per base sequence quality

![image](https://github.com/sukecosine/picture/blob/master/3.png)

这个图是所有reads每个位置综合的碱基质量值，是重头戏。横轴为read位置，纵轴是quality，由-10*log10(p)计算得到，其中p为测错错误率。比如quality为20时，此时p=0.01。
很明显背景根据质量值划分了三个区间:红色表示质量较差，橙色表示质量合理，绿色表示高质量，该图基本都分布在绿色背景。看完背景会发现很多黄色箱型图，是该位置所有碱基的25%-75%质量分布，上下须是10%-90%质量分布。而红线表示中位数，蓝线表示平均质量。
一般来说，read质量会随着测度长度的增加而有所下降。要是质量值大部分落在红色区域就要考虑重新测序了，若只有少部分落在橙色，可以对质量较低的碱基使用Trim_galore等工具去除。
如果reads任意位置的中位数低于25或下四分位数低于10，报“WARN”；若任意位置的中位数低于20下四分位数低于5，报“FAIL”。
（三）	Per tile sequence quality

![image](https://github.com/sukecosine/picture/blob/master/4.png)

只有使用Illumina library时，这个图才会出现，蓝蓝的很晃眼。横轴代表碱基位置，纵轴是tile的编号，主要显示测序过程中由于不可控因素影响测序质量。
颜色是从冷色调到暖色调，冷色指质量较高，暖色指质量较差，so蓝色代表测序质量很高。
产生警告或错误的原因可能是气泡穿过flowcell或flowcell上的污迹。若暖色出现较多，可以在后续分析中把该tile测序结果去除。
（四）	Per sequence quality scores

![image](https://github.com/sukecosine/picture/blob/master/5.png)

这个图是碱基总体质量的分布情况，也是很有价值滴。横轴是质量值，纵轴是每个值对应的reads数据。此图绝大部分序列质量都在30以上。对于二代测序，一般要求95%以上的碱基质量能达到20，85%以上的碱基质量能达到30。
若绝大部分序列平均质量低于27，相当于0.2%的错误率，报“WARN”；若绝大部分序列平均质量低于20，相当于1%的错误率，报“FAIL”。
（五）	Per base sequence content

![image](https://github.com/sukecosine/picture/blob/master/6.png)

这个图是每个位置ATCG平均比例图，横轴是序列中的每个碱基位置，纵轴是百分比。四条线分别代表ATCG，理论上AT相等CG相等且基本稳定，有偏差最好在1%左右。但由于引物为随机引物，前几个碱基的GC含量通常不稳定，会报“FAIL”，可以选择去掉。
若AT或者GC在任何位置的差值大于10%，报“WARN”；若AT或者GC在任何位置的差值大于20%，报“FAIL”。
（六）	Per sequence GC content

![image](https://github.com/sukecosine/picture/blob/master/7.png)

这是平均GC含量的分布图，横轴是GC含量，纵轴是该GC含量下对应的reads数。红线是实际分布，蓝线是理论分布。由于测序偏向性，人类基因组的GC含量一般40%左右。曲线形状偏差往往由于文库污染，若红色的线出现双峰，应该是混入其他物种的DNA序列。
若实际分布偏离理论分布的reads数目大于15%，报“WARN”；若实际分布偏离理论分布的reads数目大于30%，报“FAIL”。
（七）	Per base N content

![image](https://github.com/sukecosine/picture/blob/master/8.png)

这个图对reads的每个位置统计N的比率，N的产生是由于测序仪器不能辨别是ATCG哪个碱基造成。横轴代表reads上碱基位置，纵轴代表N占比。若测序正常，红线应该是趋近0的直线。
若任意位置的N比例超过5%，报“WARN”；若任意位置的N比例超过20%，报“FAIL”。
（八）	Sequence Length Distribution

![image](https://github.com/sukecosine/picture/blob/master/9.png)

该图是序列长度的统计，横轴为测序出来的read长度，纵轴为数量。一般测出的长度是完全相等，呈现单峰，但某些平台的读取长度本身就不同，会报“WARN”，可以忽略。
若所有序列的长度不相等，报“WARN”；若任何序列的长度为0，报“FAIL”。

![image](https://github.com/sukecosine/picture/blob/master/10.png)

该图是统计完全一样的reads出现频率，横坐标是重复的次数，纵坐标是该次数下重复reads的占比。上图中10%的reads观察到两个重复。蓝线是测序reads的重复分布，红线是原始数据去重复序列后的比例。
若重复序列占总数的20%以上，报“WARN”；若重复序列占总数的50%以上，报“FAIL”。
（十）Overrepresented sequences

![image](https://github.com/sukecosine/picture/blob/master/11.png)

该图统计Over-representedsequence出现情况，白话就是某个序列大量出现。在RNA-seq中，由于序列不受随机片段的影响，相同序列自然出现很大部分，常出现警告信息。
当Over-represented的reads数超过总reads数0.1%时，报”WARN“，当Over-represented的reads数超过总reads数1%时报”FAIL“。
（十一）Adapter Content

![image](https://github.com/sukecosine/picture/blob/master/12.png)

这个图也很重点，查看序列两端接头存在情况。横坐标为read位置，纵坐标为有adapter序列的占比。正常情况为趋于0的直线说明没有adapter。
测序时加接头是为了结合flowcell，一般由于插入片段的长度大于测序read的长度，是测不到接头。但RNA-seq的插入片段本身较短容易测到接头，你就会看到直线在两端有起伏，可以用cutadapt等工具切除。
当有adapter的序列占总reads数5%以上时，报”WARN“，当有adapter的序列占总reads数10%以上时报”FAIL
### 总结
以上就是FastQC结果图分析，搞懂以后就可以动动手指去过滤数据了。

<br/>参考资料：</br>
<br/>（1）http://www.bioinformatics.babraham.ac.uk/projects/fastqc/</br>
<br/>（2）【知乎】从零开始完整学习全基因组测序数据分析：第三节 数据质控</br>



