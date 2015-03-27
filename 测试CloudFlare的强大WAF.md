## 测试CloudFlare的强大WAF ##

发帖者：Chris Ueland　　　　　　　　　　　 日期：2014.12.29

![标题图](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/12/Scalescale-Cloudflare-WAF_v2_reduced_height.png)

**应用**　　　　　　　　　　　　　　　　　　　 **规则**

**HTTP服务器**：　　[nginx](http://nginx.com/)　　　　　　　　　  　**打开规则**：　　　[OWASP](https://www.owasp.org/index.php/Main_Page)

**APP服务器**：　　 [OpenResty](http://openresty.org/)　　　　　　　　**系统性能分析**

**JIT编译器**：　　　[LuaJIT](http://luajit.org/)　　　　　　　　　　**火焰图**：　　　　[SystemTap generated](https://github.com/brendangregg/FlameGraph)

**算法**　　　　　　　　　　　　　　　　　　
　 **实时分析**：　　　[Nginx SystemTap Toolkit](https://github.com/openresty/nginx-systemtap-toolkit)

  **字符串匹配**：　　[Aho-Corasick](https://github.com/cloudflare/lua-aho-corasick)　　　　　

---

我第一次听约翰的演讲是在旧金山举行的Nginx.Conf发布会。他精彩的演示了一个大规模，高容量的WAF（Web应用防火墙）平台，此平台已经搭建完成。在这次采访中，他将阐述他的设计目标，基准测试以及新推出的WAF。这个想法是采用Nginx和LUA来优化每一个不起眼的问题来阐述WAF的性能指标和一些测试，

–Chris / ScaleScale / MaxCDN

---

### WAF背后的远见是什么？ ###

CloudFlare希望能为大批量用户提供一个WAF。这就意味着要做两件事情：与现有的mod_security WAF兼容，这样才能有效利用现有的规则集；让人们用自己熟悉的WAF（包括CloudFlare的员工和客户）来编写新规则。
 
### CloudFlare WAF如何工作 ###

在一些常见威胁或者特殊攻击接近你的服务器之前，CloudFlare的WAF能阻拦网络边缘攻击，从而保护你的网站。它涵盖了台式机、移动网站以及应用程序。

Web应用防火墙（WAF）通过对HTTP的请求进行异常检测。它着眼于GET和POST请求和应用规则，以帮助从合法的网站访问者中过滤掉非法流量。当然，你可以决定是否要阻止、对抗或者模拟攻击。在任何非法流量入侵原始Web服务器之前，CloudFlare的WAF通过阻止和对抗来阻拦。

![CloudFlare WAF工作图](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/11/how-cloudflare-works4.png)

CloudFlare的Web应用防火墙（WAF）能自动保护这些类型的网站攻击：

<table>
<tbody>
<tr><td>SQL注入，垃圾评论</td><td>跨站点脚本（XSS）</tr>
<tr><td>分布式拒绝服务（DDoS）攻击</td><td>应用程序特定的攻击（WordPress，CoreCommerce）</td></tr>
<tbody>
</table>

### 测试CloudFlare的XSS防护 ###

打开www.jgc.org，我们很容易测试CloudFlare WAF运行情况。将某个包含一个基本XSS脚本的虚拟变量用一个简单的GET操作就会触发安全功能，并跳出一个已经被封锁的网页。

### 请求头文件 ###

![请求头文件图](http://a2.qpic.cn/psb?/V108KV171LGmJd/sGhOzqppAI0sN3fSlbMZyuJ.XwKXbRxRa9S4ClJZ8bo!/b/dBUAAAAAAAAA&ek=1&kp=1&pt=0&bo=rQKUAK0ClAADCC0!&sce=0-12-12&rf=viewer_311)

### 响应头文件 ###

![响应头文件图](http://a4.qpic.cn/psb?/V108KV171LGmJd/7TqD96kqVvcPVvZCHquE2sRUu5UPBS.GtQzG8ShXQnE!/b/dA8AAAAAAAAA&ek=1&kp=1&pt=0&bo=rQKVAK0ClQADCC0!&sce=0-12-12&rf=viewer_311)

[点击这里查看由WAF所产生的错误页面](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/11/blocked2.png)

### 初始新规则从何而来？ ###

我们的方法是将开源OWASP的规则集和基于CloudFlare客户入侵流量开发的潜在规则结合。目前，自定义规则能阻止大多数被禁止的请求。

我们通过攻击事件和漏洞来开发潜在规则，然后建立一个测试套件（采用正面和负面的测试以确保正确的规则）。WAF拥有一个大型自动测试套件，能获取整个规则集以确保正常工作。

<table>
<tbody>
<tr><td></td><td>最新添加的WAF规则</td><td></td></tr>
<tr><td>说明</td><td>应用</td><td>博客文章</td></tr>
<tr><td>Drupal 7 SQL注入</td><td>SA-CORE-2014-005</td><td>Drupal 7 SA-CORE-2014-005 SQL Injection Protection</td></tr>
<tr><td>Shellshock</td><td>Shellshock (软件bug)</td><td>Shellshock：黑客如何使用它来攻击系统
Shellshock启用保护所有客户</td></tr>
<tr><td>WHMCS Zero Day漏洞</td><td>WHMCS Security Advisory for 5.x</td><td>第0天修补一个WHMCS零日
快速部署WAF规则来保护你的站点</td></tr>
<tbody>
</table>

我们提炼GETs，POSTs等所有请求。WAF里的有一个自定义程序，可以查看POST数据并且通过MIME的类型和侦查实际字节来判断数据是什么。

WAF未向所有客户开放。只有付费用户才能收到WAF。

我们与用户合作，以确定他们网站的具体规则，并定期将WAF规则下放来阻止特定网站的攻击。今后，我们计划推出一款用户界面，用户可以为自己的网站编写和上传的规则。

### 你对速度要求高吗？你的理念是什么？　###

答案是肯定的。速度很重要，因为CloudFlare的规模以及我们服务的其中一部分就是性能指标。我们拥有各种各样的基准工具，但也许更重要的是我们的指标体系，作为检测实时信息和历史性能信息（包括WAF性能）。

我们的目标是通过WAF处理每一个请求的平均时间在1ms内。目前，我们处理每个请求的平均时间为100微秒（十分之一毫秒）。例如，在刚刚过去的24小时，我们已经封锁12亿个HTTP请求（即约14000次每秒）。

**#statpron**
<table>
<tbody>
<tr><td>14000次封锁请求/秒　　　　　　　　</td><td>12亿次封锁请求/天　　　　　　　　</td></tr>
<tr><td>目标：执行所有规则时间<=1ms</td><td>实际执行时间：约400μs</td></tr>
<tr><td>1937个字符串匹配</td><td>5682个常用规则</td></tr>
<tr><td>102个Cloudflare规则</td><td></td></tr>
<tbody>
</table>

### 首次推出，你发现何种延迟？ ###

我们首次编写和测试代码时，发现有10ms左右的延迟。在采用类似记忆功能技术优化之后，部分结构发生了变化（主要采用闭包来消除），其延迟接近1毫秒。此后，WAF投入成产，并且采用[SystemTap](https://github.com/openresty/nginx-systemtap-toolkit)和内部工具来分析LuaJIT和PCRE的工作性能。我们与迈克·鲍尔（LuaJIT的维护人员）密切合作，以确保JITed是WAF的特定功能。

我们不会用Lua，而是一直用LuaJIT来生产。因为在x64系统上，LuaJIT比Lua的性能更好。（见[http://luajit.org/performance_x86.html](http://luajit.org/performance_x86.html)）

### 你如何提高速度，查找出慢速指令？ ###

对于初始的WAF代码调整，我们采用以Lua为基础的分析工具（我们自己也写了一部分）来监视Lua代码的性能，实现WAF的功能。但当投入生产，我们采用[SystemTap](http://github.com/openresty/nginx-systemtap-toolkit)和[flamegraphs](https://github.com/brendangregg/FlameGrap)识别热点并对其进行优化。我们并不需要改变任何基础结构。我们也没有购买或使用任何新的硬件。WAF主要是采用增强型的CPU。

　　　　　　　　　　　　　　　　　　　　<1ms 延迟
![测试延时图](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/11/wafgraph3.png)

在我们采用新的WAF之前，CloudFlare将Apache和Nginx合并运行仅仅只是为了能够使用mod_security。然而这个合并过程是非常缓慢和困难的。最终它没有能与CloudFlare的业务增长成比例，所以我们采用Ｎginx+LuaJIT的新WAF来运行。

CloudFlare经营着世界上最大Ｎginx+LuaJIT的部署之一。为了处理某个请求而剃掉的每一微秒都会产生严重的影响，所以我们决定对LuaJIT开源项目作一些变化。

本项目的总体目标是获得WAF阻止响应时间的中值/在现实场景中允许在1毫秒内做出决定。在线性时序信息的测试工具下检测WAF的性能来优化。我们采用非常精密的SystemTap型仪表在CloudFlare网络中运行WAF，

SystemTap产生的信息被送入引擎收录，分析并产生一个火焰图来显示代码在哪里运行。

![火焰图](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/12/Screen_Shot_2013-08-23_at_2.57.02_PM_1.png)

在火焰图显示早期广泛使用闭包，这导致LuaJIT运行缓慢。为消除他们的使用重写编译器的某些部分，使其运行速度更快。

下面是认定为热功能相同的信息产生另一种看法。这表明，字符串匹配和规范表达式是最高代价的操作。

　　　　　　　　　![占比图](http://2zmzkp23rtmx20a8qi485hmn.wpengine.netdna-cdn.com/wp-content/uploads/2014/12/Screen_Shot_2013-08-23_at_2.55.07_PM.png)

为了使这些配套功能运行得更快，我们已经实现了自己版本的Aho-Corasick算法。[Aho-Corasick算法](http://www.youtube.com/watch?v=d24CyiU1JFk)是一种快速的字符串匹配算法，它可以匹配大量关键字同时处理输入文本。相对于单独使用Boyer-Moore搜索，需要多次通过的字符串的搜索，该算法的优点在于，在大量的文本通过一次就可以匹配多个字符串。在[这篇文章](http://architects.dzone.com/articles/algorithm-week-aho-corasick)中，作者展示了Aho-Corasick如何用Haskell实现。 CloudFlare开源在[Golang](https://github.com/cloudflare/ahocorasick)和[基于LUA的C++](https://github.com/cloudflare/lua-aho-corasick)中实施传统的Aho-Corasick。

优化Lua语言、LuaJIT编译器以及WAF就是意味着在nginx内部的所有Lua WAF能非常快速和灵活的运行。

### 阅读和观看更多关于NGINX采用Lua构建低延迟的WAF ###

[观看约翰](http://www.youtube.com/watch?v=nlt4XKhucS4)在[YouTube](http://www.youtube.com/watch?v=nlt4XKhucS4)上的“在NGINX内部采用Lua构建低延迟WAF”演讲。你也可以点击[这里](https://github.com/cloudflare/jgc-talks/blob/master/nginx.conf/2014/cloudflare-lua-waf.pdf)下载这个视频。

### 相关文章 ###

*[Rolling Your Own CDN – Build A 3 Continent CDN For $25 In 1 Hour](http://www.scalescale.com/rolling-your-own-cdn-build-a-3-continent-cdn-for-25-in-1-hour/)

*[The Making of OnMetal](http://www.scalescale.com/the-making-of-onmetal/)

*[News From Around the Web](http://www.scalescale.com/news-around-web/)

*[Open Source Software to Help You Scale](http://www.scalescale.com/open-source-software-help-scale/)

*[Utilizing GPUs in The Cloud](http://www.scalescale.com/utilizing-gpus-cloud/)
