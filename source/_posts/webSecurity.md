title: webSecurity
date: 2015-09-07 06:43:51
tags: security
---
**XSS攻击：**允许恶意web用户将代码植入到提供给其它用户使用的页面中。比如包括html代码和客户端脚本。

利用XSS漏洞旁路掉访问控制——例如同源策略。---这种类型的漏洞如：钓鱼网站

跨站脚本攻击是新型的“缓冲区溢出攻击“，而javascript是新型的'shellCode'。

在2007年OWASP所统计的所有安全威胁中，跨站脚本攻击占到了22%，高居所有Web威胁之首。

XSS攻击危害包括：
1、盗取各类用户帐号，如机器登录帐号、用户网银帐号、各类管理员帐号
2、控制企业数据，包括读取、篡改、添加、删除企业敏感数据的能力
3、盗窃企业重要的具有商业价值的资料
4、非法转账
5、强制发送电子邮件
6、网站挂马
7、控制受害者机器向其它网站发起攻击

攻击利用手法的类型：
----
A、本地利用漏洞:这种漏洞存在于页面中客户端脚本自身。如：
>-A给B发送一个恶意构造了Web的URL，B点击并查看了这个URL。
>-恶意页面中的JavaScript打开一个具有漏洞的HTML页面并将其安装在Bob电脑上。
>-具有漏洞的HTML页面包含了在Bob电脑本地域执行的JavaScript。
>-Alice的恶意脚本可以在Bob的电脑上执行Bob所持有的权限下的命令。

B、反射式漏洞：与A有些类似，不同的是Web客户端使用Server端脚本生成页面为用户提供数据时，如果未经验证的用户数据被包含在页面中而未经HTML实体编码，客户端代码便能够注入到动态页面中。如：
>-A经常浏览某个网站，此网站为B所拥有。B的站点运行A使用用户名/密码进行登录，并存储敏感信息。
>-C发现B的站点包含反射性的XSS漏洞。
>-C编写一个利用漏洞的URL，并将其冒充为来自B的邮件发送给A。
>-A在登录到B的站点后，浏览C提供的URL。
>-嵌入到URL中的恶意脚本在A的浏览器中执行，就像它直接来自B的服务器一样。此脚本盗窃敏感信息(授权、信用卡、帐号信息等)。
>-然后在A完全不知情的情况下将这些信息发送到C的Web站点。

C、存储式漏洞：骇客将攻击脚本上传到Web服务器上，使得所有访问该页面的用户都面临信息泄漏的可能，其中也包括了Web服务器的管理员。
>-B拥有一个Web站点，该站点允许用户发布信息/浏览已发布的信息。
>-C注意到B的站点具有类型C的XSS漏洞。
>-C发布一个热点信息，吸引其它用户纷纷阅读。
>-B或者是任何的其他人如A浏览该信息，其会话cookies或者其它信息将被C盗走。

**传统防御技术：**采用的模式匹配方法一般会需要对“javascript”这个关键字进行检索，一旦发现提交信息中包含“javascript”，就认定为XSS攻击。

缺陷显而易见：可以通过插入字符或完全编码的方式躲避检测，如javascript中加入多个tab键/空格字符/回车/回车符\r/完全编码。

**升级的防御技术：**
1、对所有用户提交内容进行可靠的输入验证，包括对URL、查询关键字、HTTP头、POST数据等，仅接受指定长度范围内、采用适当格式、采用所预期的字符的内容提交，对其他的一律过滤。
2、实现Session标记(session tokens)、CAPTCHA系统或者HTTP引用头检查，以防功能被第三方网站所执行。
3、确认接收的的内容被妥善的规范化，仅包含最小的、安全的Tag(没有javascript)，去掉任何对远程内容的引用(尤其是样式表和javascript)，使用HTTP only的cookie。


事件如：新浪微博XSS受攻击事件，2011年6月28日晚。

**DDOS攻击：**分布式拒绝服务攻击，Nick Sullivan撰文介绍了攻击者如何利用恶意网站、服务器劫持和中间人攻击发起DDoS攻，并说明了如何使用HTTPS以及即将到来的名为“子资源一致性（Subresource Integrity，简称SRI）”的Web新技术保护网站免受攻击。

现代网站的大部分交互都来自于JavaScript。网站通过直接向HTML中添加JavaScript代码或者通过HTML元素<script src="">从远程位置加载JavaScript实现交互功能。JavaScript可以发出HTTP(S)请求，实现网页内容异步加载，但它也能将浏览器变成攻击者的武器。如：
{% codeblock %}
function imgflood() {  
  var TARGET = 'victim-website.com'
  var URI = '/index.php?'
  var pic = new Image()
  var rand = Math.floor(Math.random() * 1000)
  pic.src = 'http://'+TARGET+URI+rand+'=val'
}
setInterval(imgflood, 10)
{% endcodeblock %}
>上述脚本每秒钟会在页面上创建10个image标签。该标签指向“victim-website.com”，并带有一个随机查询参数。如果用户访问了包含这段代码的恶意网站，那么他就会在不知情的情况下参与了对“victim-website.com”的DDoS攻击。

这种攻击之所以有效是因为HTTP中缺少一种机制使网站能够禁止被篡改的脚本运行。为了解决这一问题，W3C已经提议增加一个新特性子资源一致性。该特性允许网站告诉浏览器，只有在其下载的脚本与网站希望运行的脚本一致时才能运行脚本。这是通过密码散列实现的。

密码散列可以唯一标识一个数据块，任何两个文件的密码散列均不相同。属性integrity提供了网站希望运行的脚本文件的密码散列。浏览器在下载脚本后会计算它的散列，然后将得出的值与integrity提供的值进行比较。如果不匹配，则说明目标脚本被篡改，浏览器将不使用它。不过，许多浏览器目前还不支持该特性，Chrome和Firefox正在增加对这一特性的支持。

中间人攻击：
-----
中间人攻击是攻击者向网站插入恶意JavaScript代码的最新方式。在通过浏览器访问网站时，中间会经过许多节点。如果任意中间节点向网页添加恶意代码，就形成了中间人攻击。

加密技术可以彻底阻断这种代码注入。借助HTTPS，浏览器和Web服务器之间的所有通信都要经过加密和验证，可以防止第三者在传输过程中修改网页。因此，将网站设为HTTPS-only，并保管好证书以及做好证书验证，可以有效防止中间人攻击。


**工具：**用Netsparker扫描网站。

引起攻击的几个原因：
----
1.前端和后端的输入验证不够规范统一及完整.

2.对于用户输入的数据完全信任是危险的，没有编码和过滤一些恶意字符。如：
网址： /product/productSearch.aspx? ?coid=3&pmid=D710%20(Epic%204G%20Touch)&kw=%22%2Balert(9)%2B%22&IsPromotion=0
>-第三个参数kw因为在URL中显示的是编码后的，参数的实际值是”+alert(9)+”。
>-productSearch.aspx页面，参数值传过来没有经过hmtl编码。后端C#程序一个类变量searchkw被赋值”+alert(9)+”。
>-页面前端执行了如下一段js代码
{% codeblock %}
<script  type="text/javascript">

getNews("searchDisplay.aspx?pcid=<%=currentPcid %>&pmid=<%=pmid %>&coid=<%=coid %>&psid=&counter=<%=currentCount %>&kw=searchkw%>&isPromotion=<%=isPromotion %>", "searchDiv1");

</script>
{% endcodeblock %}
这样的话”+alert(9)+”的第一个双引号把前面的代码都截断了，第二个双引号把后面的都截断了。中间就只剩下alert(9),  便会执行alert(9)这句代码。这句代码可以替换成任意有攻击性的javascript代码段，比如获取cookie的值x=document.cookie;然后提交给攻击者。

3.数据库查询没有使用asp.net中安全的参数化操作。

解决这几个问题,有以下三步需要做：
-----

1.对于数字类型的参数值、用户输入的部分前端和后端都需要验证数据类型的正确性.

2.对于字符串类型的参值或者用户输入的部分后端通过使用AntiXssLibrary.dll来编码字符串.

3.所有SQL语句中拼接字符串的全部使用SqlParameter类封装传入SQL的参数.