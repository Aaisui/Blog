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



很明显的变量覆盖，下面还紧跟了一个cfile，这样的话前半段很显然就是用cfile可以达成任意文件读取的效果，后半段利用pearcmd即可，这里在我[别的文章](https://github.com/ysllz/Blog/issues/15)里面已经写过了故不再赘述了

