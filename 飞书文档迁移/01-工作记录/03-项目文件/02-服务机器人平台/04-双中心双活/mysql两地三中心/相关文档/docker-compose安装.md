---
created: '2024-12-05 15:09:01'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Jp9LdF1x2ofnJjxhbe7cTGvWnBe
modified: '2024-12-24 10:22:53'
source: feishu
title: docker-compose安装
---

docker-compose安装

#下载版本包
https://github.com/docker/compose/releases 
#点开后下载带这个后缀的文件：docker-compse-linux-X86_64
#如果上面的版本没有这个后缀，就往下翻找之前的版本

#上传版本包、下载好之后，将这个文件传给我们对应的服务器
# 然后将文件更改名字
mv docker-compose-linux-x86_64 docker-compose

# 移动到user/local/bin中
mv docker-compose /usr/local/bin

# 切换至对应文件夹中
cd /usr/local/bin

# 赋权最高的权限
chmod +x /usr/local/bin/docker-compose

# 查看版本，输出版本信息，安装成功
docker-compose version
