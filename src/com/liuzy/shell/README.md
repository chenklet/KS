# -----------------------------------------------------

## 以下测试在VMware虚拟机中的CentOS6.5中完成

## 用root登陆执行脚本

## 执行clear.sh
```清除了CA密钥和签发库```

## 执行create.sh
```
CA自签依次输入：cn sh sh liuzy CA CA，然后回车跳过其他输入；
生成Nginx证书请求依次输入：cn sh sh liuzy nginx nginx，然后回车跳过其他输入；
签发Nginx证书输入两次y回车确认
生成Client证书请求依次输入：cn sh sh liuzy client client，然后回车跳过其他输入；
签发Client证书输入两次y回车确认
```

## 执行nginx.sh
```
打开浏览器，输入主机IP，可以访问Nginx主页
```

## 执行nodejs.sh
- 安装NodeJS、项目生成器、后台运行器

## 使用liuzy或普通用户登陆
- 创建项目
```shell
# 新建nodejs项目test
express test -e
# 进入项目目录
cd test
# 安装项目依赖
npm install
# 编辑文件，使访问/时，返回一个JSON字符串
vim routes/index.js
# 删除或用//注释res.render('index', { title: 'Express' });
# 改为
res.send({name:"liuzy",QQ:"416657468"});
```
- 启动项目
```
cd ~
forever start -o test.log test/bin/www
# 看是否运行，项目默认访问端口是3000
lsof -i :3000
## 用root配置nginx
```
- 查看日志
```
cd ~
tail -f test.log
```
- 访问项目
```
浏览器打开http://你主机的IP:3000
会看到服务器返回了一个JSON，内容是：
{"name":"liuzy","QQ":"416657468"}
```

## 配置Nginx

- 编辑/etc/nginx/nginx.conf，在http的大括号内的最后加上配置
```
server {
	listen 80;
	server_name liuzy.com www.liuzy.com;
	location / {
		rewrite (.*) https://www.liuzy.com$1 permanent;
	}
}
server {
	listen 443 ssl;
	server_name liuzy.com www.liuzy.com;
	ssl_certificate /etc/pki/CA/myssl/nginx.crt;
	ssl_certificate_key /etc/pki/CA/myssl/nginx.pem;
	#双向认证配置，现在注释
	#ssl_client_certificate /etc/pki/CA/cacert.pem;
	#ssl_verify_client on;
	location / {
		proxy_set_header Host $host;
		proxy_set_header X-Real-Ip $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		proxy_pass http://127.0.0.1:3000;
		client_max_body_size 20m;
	}
}
```
- 测试配置是否正确，正确会输出successful
`nginx -t`
- 应用配置，没有任何返回信息说明成功了
`nginx -s reload`

## 修改host
- 修改本机Windows系统的host文件
```
(如果你的win8,win10，你先把hosts复制到桌面或项目地方，改好了再拖进来)
打开`C:\Windows\System32\drivers\etc`文件夹
编辑hosts，添加内容：(前面地址是你虚拟机的IP)
192.168.31.186 liuzy.com
192.168.31.186 www.liuzy.com
```
- 测试修改
```
打开cmd，输入ping liuzy.com，观察是否转到虚拟机IP
``` 

## 访问项目
这时，你用浏览器访问http://liuzy.com或者http://www.liuzy.com，请求会被系统DNS到你的虚拟机

虚拟机中的nginx收到请求，会将你的地址重定向到https://www.liuzy.com

然后转发到虚拟机的3000端口，NodeJS项目test会给你返回一个JSON字符串

到此已经完成了单向认证，双向认证请继续

## 双向认证
- 打开双向认证
```
用root用户登陆，编辑/etc/nginx/nginx.conf
解开双向认证注释：
ssl_client_certificate /etc/pki/CA/cacert.pem;
ssl_verify_client on;
应用配置还需要：
nginx -s reload
```
- 此时用浏览器访问，你会看到：(因为你没有被服务器信任的证书)
```
No required SSL certificate was sent
```
- 用程序加载证书请求服务器
```
把create.sh脚本签发的客户端证书下载到你的系统，保存到E盘；
/etc/pki/CA/myssl/client.pem
/etc/pki/CA/myssl/client.crt
/etc/pki/CA/myssl/nginx.crt
运行com.liuzy.test.HttpsTest类的main方法；
看，返回了结果！
```

