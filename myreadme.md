1. Langkah pertama:
    - buat file di /etc/nginx/sites-available/nama_domain menggunakan sintak touch
    - buka file dengan command nano nama_domain
    - isi file dengan dibawah ini

    server {
        listen 80;
        server_name payment-services.msglow.today;

        root /var/www/html/default;
        index index.php index.html index.htm;

        location / {
            return 200 'temporary config for certbot verification';
            add_header Content-Type text/plain;
        }
    }

    - setelah selsai ctrl+x lalu tekan enter untuk otomatis di save dan exit

2. Langkah kedua
    - reload nginx agar aktif dengan comman sebagai berikut:
    - sudo nginx -t
    - sudo systemctl reload nginx

3. Langkah ketiga
    - jalankan certibot untuk membuat sertifikat baru
    sudo apt update
    sudo apt install certbot python3-certbot-nginx -y

    - lalu jalankan
    sudo certbot certonly --nginx -d payment-services.msglow.today

4. Langkah keempat
    - buka lagi file ssl nya yang ada di /etc/nginx/sites-available/nama_domain dengan sintak nano nama_domain
    - lalu ubah konfigurasinya menjadi 

    server {
        if ($host = payment-services.msglow.today) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
        server_name payment-services.msglow.today;
        listen 80;
    }
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        root /var/www/html/default;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name payment-services.msglow.today;
        ssl_certificate /etc/letsencrypt/live/payment-services.msglow.today/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/payment-services.msglow.today/privkey.pem; # managed by Certbot

        location / {
            proxy_pass http://127.0.0.1:8085;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_ssl_verify off;
            proxy_read_timeout 500;
            proxy_connect_timeout 500;
            proxy_send_timeout 500;
        }
        location ~ /\.ht {
            deny all;
        }
        location ~ /(docs|redoc|openapi.json) {
            allow 10.100.0.0/16;
            allow 10.0.0.0/16;
            deny all;
            proxy_pass http://127.0.0.1:8085;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_ssl_verify off;
            proxy_read_timeout 500;
            proxy_connect_timeout 500;
            proxy_send_timeout 500;
        }

    }

    - setelah selesai tekan ctrl+x lalu tekan enter

5. Langkah kelima
    - aktifkan ssl dengan command
    sudo ln -s /etc/nginx/sites-available/payment-services.msglow.today /etc/nginx/sites-enabled/

6. Langkah keenam
    - test dan reload nginx:
    - sudo nginx -t
    - sudo systemctl reload nginx
