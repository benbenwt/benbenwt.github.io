### wireshark分割

```
https://jingyan.baidu.com/article/63f2362854be2e0208ab3d20.html
cd /pathtoWireshark
editcap -c  分割份数  待分割文件目录\文件名.pcap 分割后存储的文件目录\文件名.pcap
```



### pcap恶意

```
pcap index:https://mcfp.felk.cvut.cz/publicDatasets/
MACCDC比赛中的流量:https://www.netresec.com/?page=MACCDC
netresec网站: https://www.netresec.com/?page=PcapFiles
相关博客: https://www.cnblogs.com/bonelee/p/11379587.html

说几个我经常用的，免费的：
1.  Malware  Traffic  Analysis:  http://www.malware-traffic-analysis.net/2018/index.html    这个网站每天更新，主要是欧美地区的新鲜流行木马样本，基本上当天更新的马都很新～
2.Virus  Bay：  https://beta.virusbay.io/    这个算是社区贡献吧

收费的：
1.Virustotal  Intelligence：https://www.virustotal.com/intelligence  这个是VT提供的，你所在的公司要付钱给VT，这样你可以去根据HASH和自定义YARA去找样本。
2.Abusix：恶意垃圾邮件提供商，每天提供大量的新鲜的垃圾邮件，80%内容是恶意的。
3.Support  Intelligence：收集各大反病毒厂商收集的样本，然后转手卖给各大IOC提取商～
4.Lexsi：同Support  Intelligence

 

 

你好，比如说，我想分析利用MS17_010漏洞的病毒，又或者我想分析某款病毒分变种，有没有什么网站能够跟你条件来查样本呢？
网站能够根据条件来查样本，一般你需要去各大在线沙盘的网站，例如  Hybird-Analysis，根据Tag来找，找到了根据HASH来找样本

曾经也遇到楼主的问题，收集了一些国外的样本下载网站：
1）https://www.hybrid-analysis.com/    这个网站可以下载，但是需要注册账号，个人注册需要提交三个以上博客或者原创技术文章链接，使用企业邮箱申请的通过的比较快一些

2）https://app.any.run/    这个网站是一个免费沙箱，可以浏览其他人跑的样本结果，也可以下载样本，不需要注册账号就能下载，注册也是免费的！

3）http://vxvault.net/ViriList.php  这个！！没下载过

4）http://malc0de.com/database/  每天更新最新样本

最后老外推荐的样本资源：https://zeltser.com/malware-sample-sources/
```

