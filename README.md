# nginxConfig-vod ubuntu 20버전 기준
nginx vod config

<a href="http://orlinux.com/archives/9241"> 참고(이 사이트 예제는 cent os 기준)</a>
-수동 컴파일에 필요한 기본 설치 내용은 스킵
## nginx 설치 

```
wget http://nginx.org/download/nginx-1.17.8.tar.gz  
git clone https://github.com/kaltura/nginx-vod-module.git
버전은 맘대로
```

## 압축해제 및 폴더 이동
```
cd /clone받은 위치
nginx-1.17.8.tar.gz 압축해제
tar -xvzf nginx-1.17.8.tar.gz
cd nginx-1.17.8
```


## 컴파일
```
컴파일이 실패 한다면 해당 오류명의 라이브러리? 를 설치 
./configure --add-module=../nginx-vod-module \
--with-pcre \
--with-http_auth_request_module \
--with-http_degradation_module \
--with-http_geoip_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_image_filter_module \
--with-http_perl_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module  \
--with-http_stub_status_module \
--with-http_v2_module \
--with-stream_ssl_module \
--with-stream \
--with-threads \
--with-http_mp4_module \
--with-file-aio \
--with-cc-opt="-O3" \

성공 했다면
make  ->  make install (없다고 나오면 apt install gogo) 
```

## nginx conf 설정

```
worker_processes  1;
user  root;
 
events {
    worker_connections  1024;
}
 
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
 
    server {
        listen       80;
        server_name  localhost;
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log; 
        location / {
            root   html;
            index  index.html index.htm;
        }
 
        error_page   500 502 503 504  /50x.html;
 
        location = /50x.html {
            root   html;
        }
 
        # vod settings
        vod_mode local;
        vod_fallback_upstream_location /fallback;
        vod_last_modified 'Sun, 19 Nov 2000 08:52:00 GMT';
        vod_last_modified_types *;
        vod_align_segments_to_key_frames on;
        vod_manifest_segment_durations_mode accurate;
        vod_segment_duration 1000;
        vod_hls_index_file_name_prefix playlist;
 
        # vod caches
        vod_metadata_cache metadata_cache 512m;
        vod_response_cache response_cache 128m;
 
        # gzip mainfests
        gzip on;
        gzip_types application/vnd.apple.mpegurl;
 
        # file handle caching / aio
        open_file_cache          max=1000 inactive=5m;
        open_file_cache_valid    2m;
        open_file_cache_min_uses 1;
        open_file_cache_errors   on;
        aio on;
 
        location /contents {
                root /home/hls;
                vod hls;
                add_header Access-Control-Allow-Headers '*';
                add_header Access-Control-Expose-Headers 'Server,range,Content-Length,Content-Range';
                add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
                add_header Access-Control-Allow-Origin '*';
                expires 100d;
        }
        location /vod_status {
                vod_status;
                access_log off;
        }
    }
}
```

## contents 폴더 생성

```
mkdir -p /home/hls/contents
```

## /home/hls/contents에 동영상 파일 넣고 재생


## service 추가(<a href="https://opentutorials.org/module/384/4511"> 참고 [생활코딩]</a> )

```
sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx;
sudo chmod +x /etc/init.d/nginx;

service nginx status  # 현재 실행중인 NGXIN의 상태를 체크
service nginx stop    # 서버 정지
service nginx start   # 서버 시작
```

