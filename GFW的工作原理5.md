[GFW的工作原理（5）](https://plus.google.com/+GhostAssassin/posts/K8zYdnEoG9Z)
                                ——HTTP，老大哥在看着你！

话说不知还有没有人记得，在google还没有被完全封锁的时候，如果你尝试搜索“胡萝卜”都会被直接断网15分钟。那么这究竟是怎么回事呢？

那时候google搜索并不支持HTTPS，也就是说当你输入关键词回车之后，搜索请求是明文发送到google服务器的。对，明文！“我大概能猜到是怎么回事了：GFW的关键词过滤，可具体原理是怎么样的呢？为什么胡萝卜这词都变成敏感词了呢？”

要把这胡萝卜问题讲清楚，需要一个家伙过来帮忙：HTTP（HyperText Transfer Protocol，超文本传输协议）。“嗨，大家好，我是应用层的HTTP。平时大家要浏览网页，就是在和我打交道，我的功能可是很强大的，你们看文章看视频都要靠我......”

......算了，还是我自己来说吧：HTTP是工作在TCP之上的应用层协议，专用端口为80，工作流程非常简单：请求——》响应，专门负责封装用户请求和目标网站返回的网页，是浏览器最好的朋友。

当用户输入一个URL并回车时，首先是DNS开始工作，获得IP地址[1]；然后TCP就出场了，建立连接[2]；接下来就是HTTP的专场了：HTTP把用户请求封装成数据包，然后发送出去，在这一过程中有不同的命令：GET，HEAD，POST，DELETE，TRACE，OPTION，PUT，CONNECT这些，其中最常用的是GET和POST。GET表示向一个特定的来源请求页面，POST则是向目标主机传送要求的数据（例如登录时传送账户信息）。除了命令之外，HTTP数据包还要传送以下数据：HTTP Headers 和user data，HTTP header中有URL和HOST等信息，cookie[3]就是一种特殊的HTTP headers。具体这次就不展开了，关键在于不管是Headers还是data，他们都是明文的。（“我不干！你还有一大堆没有介绍呢！”“拜托，下次会给你开专场的！”）

具体到胡萝卜问题，情况就是你键入了胡萝卜之后，回车，那么带有胡萝卜明文的HTTP数据包就被发送出去了，命令是GET。而GFW则静静的等在那里：哈，来了一个GET包！马上复制并匹配黑名单！啊，里面有“胡”啊（胡河蟹，你懂的），那么马上发送RST包阻断[2]！不仅如此，还要屏蔽源IP地址所有请求15分钟！（恭喜你，断网了）

事实上GFW还会检查URL和HOST呢，一旦匹配上黑名单，也会马上发送RST包阻断。

而且更恐怖的是，GFW不仅会检查从墙内发出的GET数据包，而且还会检查从墙外发回的数据包呢，目标网站的响应结果也会被GFW关键字匹配过去，一旦匹配黑名单成功就直接RST了。研究表明GFW相关设备并不仅仅设置在国际出口，实际上是分布式的，在各省的骨干网上都有相应的黑名单匹配设备，而且各省封锁情况不同。[4]

啊，似乎已经说得差不多了，还有一些比较零碎的原理知识，下一篇再说吧！

科普文链接：https://plus.google.com/109790703964908675921/posts/TpdEExwyrVj

参考资料：
1，SSL/TLS的原理以及互联网究竟是如何工作的（5）
                                                             ————DNS和他的兄弟
https://plus.google.com/109790703964908675921/posts/8KX9pDDjQxq
2，GFW的工作原理（4）
                                ——“连接被重置！”
https://plus.google.com/109790703964908675921/posts/Hwcm9QURSYh
3，好奇的商业公司，烦人的广告，出卖你的浏览器，不要太独特https://plus.google.com/109790703964908675921/posts/HM2mjn6zrRG
4，Empirical Study of a National-Scale Distributed Intrusion Detection System: Backbone-Level Filtering of HTML Responses in China
http://www.google.com/url?q=http%3A%2F%2Fciteseerx.ist.psu.edu%2Fviewdoc%2Fdownload%3Fdoi%3D10.1.1.191.206%26rep%3Drep1%26type%3Dpdf&sa=D&sntz=1&usg=AFQjCNHS4_oZaK7FgXeT6YCVHqstaPTBtQ﻿
