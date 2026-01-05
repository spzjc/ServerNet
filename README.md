（在 Windows PowerShell 执行）
ssh -fN -R 10899:127.0.0.1:7897 -p 16489 root@connect.cqa1.seetacloud.com

-R 10899:127.0.0.1:7897	服务器端 10899 端口映射到你本机 Clash 的 7897
-fN	后台运行，不打开 shell
-p 16489	SSH 端口
root@connect.cqa1.seetacloud.com	服务器登录

在服务器 / autodl 容器设置代理

登录服务器 / 容器后执行：

export ALL_PROXY=socks5h://127.0.0.1:10899
export http_proxy=http://127.0.0.1:10899
export https_proxy=http://127.0.0.1:10899


建议写入 /root/.bashrc 或 /root/.zshrc，自动生效：

echo 'export ALL_PROXY=socks5h://127.0.0.1:10899' >> ~/.bashrc
echo 'export http_proxy=http://127.0.0.1:10899' >> ~/.bashrc
echo 'export https_proxy=http://127.0.0.1:10899' >> ~/.bashrc
source ~/.bashrc

测试

在服务器 / 容器：

curl https://ipinfo.io
