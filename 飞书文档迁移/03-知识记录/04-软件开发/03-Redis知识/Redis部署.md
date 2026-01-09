---
created: '2024-04-11 15:56:39'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/QHNAdyMHMoawTPxR6xscQaranjb
modified: '2024-04-11 15:56:52'
source: feishu
title: Redis部署
---

Redis部署
  make install PREFIX=/gzw/soft/redis-7.0.3
sed "s/6479/6480/g" redis-6479.conf > redis-6480.conf
sed "s/6479/6481/g" redis-6479.conf > redis-6481.conf
sed "s/6479/6482/g" redis-6479.conf > redis-6482.conf
sed "s/6479/6483/g" redis-6479.conf > redis-6483.conf
sed "s/6479/6484/g" redis-6479.conf > redis-6484.conf
sed "s/6479/6485/g" redis-6479.conf > redis-6485.conf
sed "s/protected-mode yes/protected-mode no/g"
sed -i "s/6379/6485/g" grep 6379 -rl redis-6485.conf 
sed -i "s/# requirepass foobared/requirepass wgtest/g" grep '# requirepass foobared'-rl redis-6485.conf 
./redis-cli -h 172.30.13.56 -p 6479
3主3从
./redis-cli -h 172.30.13.56 -p 6479 --pass  wgtest --cluster create 172.30.13.56:6479 172.30.13.56:6480 172.30.13.56:6481 172.30.13.56:6482 172.30.13.56:6483 172.30.13.56:6484 --cluster-replicas 1
sed -i "s/protected-mode yes/protected-mode no/g" grep 'protected-mode yes' -rl redis-6479.conf 
./redis-server ../../node/redis-6479.conf 
sed -i "s/bind 172.30.13.56/#bind 172.30.13.56/g" grep 'bind 172.30.13.56' -rl redis-6479.conf 
