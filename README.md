# installquarkblockchainnode
Linux系统搭建夸克区块链节点
#一、安装docker
# 1.要求系统内核在3.10版本以上，检查内核
 uname -r
# 2.安装命令
  yum install docker
# 3.启动docker
   systemctl start docker
# 4.设置开机自启
  systemctl enable docker

# 二、安装节点

# 1. 创建目录, 保存节点数据
mkdir -p /data/qk_node

# 2. 切换到数据保存目录
cd /data/qk_node

# 3. 
wget https://static.quarkblockchain.cn/app/pc/qk_poa.json -O qk_poa.json

# 4. 
docker run -it --rm -v /data/qk_node:/root/qk_node  chenjia404/qk_node init /root/qk_node/qk_poa.json --datadir /root/qk_node/qk_poa 

# 5.
wget https://static.quarkblockchain.cn/app/pc/static-nodes.json -O /data/qk_node/qk_poa/static-nodes.json

# 6. 启动节点
docker run -it --name qk_poa_node -v /data/qk_node:/root/qk_node -p 8545:8545 -p 30303:30303 -p 30303:30303/udp -d chenjia404/qk_node --syncmode snap --snapshot --datadir /root/qk_node/qk_poa --networkid 20181205 --v5disc --txpool.pricelimit 1000000000 --light.serve 20 --light.maxpeers 200 --maxpeers 2000 --http --http.addr 0.0.0.0 --http.vhosts "" --allow-insecure-unlock  --http.api "net,web3,eth,personal,clique,txpool" --http.corsdomain "*" console

运行公共rpc节点，需要配置 http.vhosts ，里面配置你的域名，如果有多个域名，使用英文逗号(,)分割，另外不需要--allow-insecure-unlock 参数。

# 7. 自动更新镜像
docker run -d --name watchtower-qk-node --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --cleanup -i 3600  qk_poa_node


# 8.常见指令
关闭容器
docker stop qk_poa_node    
删除容器
docker rm  qk_poa_node  
docker attach qk_poa_node  
# 三、配置https
#1.安装宝塔
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh





#宝塔ssl配置文件（修改域名即可）





     server {
	listen 80;
	listen [::]:80;
	server_name rpc.chelizi.org.cn;
	root /www/wwwroot/rpc.chelizi.org.cn/;
	gzip on;
	gzip_min_length  1k;
	gzip_buffers     4 16k;
	gzip_http_version 1.0;
	gzip_comp_level 6;
	gzip_types       text/plain application/x-javascript application/javascript application/json text/css application/xml;
	gzip_vary on;
	gzip_disable        "MSIE [1-6]\.";

	location ~* \.(ttf|ttc|otf|eot|woff|font.css)$ {
		add_header Access-Control-Allow-Origin "*";
		add_header Access-Control-Allow-Headers X-Requested-With;
		add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
	}
		
	
	location / {
		try_files $uri @proxy;
	}
	
	location @proxy { 
			proxy_pass  http://127.0.0.1:8545;
	 
			#Proxy Settings
			proxy_redirect     off;
			proxy_set_header   Host             $host;
			proxy_set_header   X-Real-IP        $remote_addr;
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
			proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
			proxy_max_temp_file_size 0;
			proxy_connect_timeout      90;
			proxy_send_timeout         90;
			proxy_read_timeout         90;
			proxy_buffer_size          4k;
			proxy_buffers              4 32k;
			proxy_busy_buffers_size    64k;
			proxy_temp_file_write_size 64k;
	   }



	access_log  /www/wwwlogs/rpc.chelizi.org.cn.log;
      }

      server {
	listen 443 ssl http2;
	server_name  rpc.chelizi.org.cn;
	root /www/wwwroot/rpc.chelizi.org.cn;
    
	location ~* \.(ttf|ttc|otf|eot|woff|font.css)$ {
		add_header Access-Control-Allow-Origin "*";
		add_header Access-Control-Allow-Headers X-Requested-With;
		add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
	}
	

	ssl_certificate    /www/server/panel/vhost/cert/rpc.chelizi.org.cn/fullchain.pem;
      ssl_certificate_key    /www/server/panel/vhost/cert/rpc.chelizi.org.cn/privkey.pem;
	
	ssl_session_cache shared:SSL:10m;
	ssl_session_timeout 10m;
	
	ssl_protocols               TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
	ssl_ecdh_curve              X25519:P-256:P-384:P-521;
	ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
	ssl_prefer_server_ciphers   on;
	resolver 8.8.8.8 valid=300s;
	resolver_timeout 10s;
	add_header  Strict-Transport-Security "max-age=99999999;" always;
	
	location / {
		try_files $uri @proxy;
	}
	
	location @proxy { 
			proxy_pass  http://127.0.0.1:8545;
	 
			#Proxy Settings
			proxy_redirect     off;
			proxy_set_header   Host             $host;
			proxy_set_header   X-Real-IP        $remote_addr;
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
			proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
			proxy_max_temp_file_size 0;
			proxy_connect_timeout      90;
			proxy_send_timeout         90;
			proxy_read_timeout         90;
			proxy_buffer_size          4k;
			proxy_buffers              4 32k;
			proxy_busy_buffers_size    64k;
			proxy_temp_file_write_size 64k;
	   }

	access_log  /www/wwwlogs/https_rpc.chelizi.org.cn.log;
}


