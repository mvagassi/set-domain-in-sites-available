URL API DEV : <https://ims-api-dev.msglow.app/docs#/>

=====================GET STARTED====================
- buka terminal ketik python -m venv venv
=====================GET STARTED====================

=====================INSTALLATION PROJECT===================
- masuk ke dalam environment project "source venv/bin/activate". jika menggunakan terminal fish "source venv/bin/activate.fish"
- install library "python -m pip -r requirements.txt" (jika project anda sudah ada file requirements.txt)
- pastikan jika ingin menginstall library baru harus di record pada file requirements.txt dengan command "python -m pip freeze > requirements.txt"
  fungsinya agar project bisa berjalan dengan baik dan tidak ada library yang missing dan perhatikan versi dari library tersebut.
=====================INSTALLATION PROJECT===================

==============RUN FASTAPI IN LOCAL============
- uvicorn app.main:app --reload --port 8081

lalu buka terminal baru untuk run celery
- celery -A app.main.celery_app worker
==============RUN FASTAPI IN LOCAL============

==================STRUKTUR FOLDER==================
api -> list endpoint
services
middleware
models -> untuk menyimpan model atau struktur table dari database
controllers
schemas
helpers
adapters
core -> config.py
==================STRUKTUR FOLDER==================

==============DOCKERRIZE=====================
login git : docker login registry.msglow.cloud
UNTUK BUILD DOCKER :
    - command : docker buildx build -t <nama repo>
    - contoh : docker buildx build --platform linux/amd64 -t registry.msglow.cloud/services/service-ims/fastapi-erp:1.0.1 .
    setelah selesai build maka di push ke git berikut adalah command nya
    - docker push  registry.msglow.cloud/services/service-ims/fastapi-erp:1.0.1

    untuk melihat docker yang berjalan command nya adalah
    - docker ps
    untuk melihat list image dari docker command nya adalah
    - docker images
    untuk menjalankan docker command nya adalah
    - docker run -d -p 8008:8007 registry.msglow.cloud/services/service-ims/fastapi-erp:1.0.1
    port 8008 adalah port yang digunakan untuk mengakses.
    port 8007 adalah port yang di setting di Dockerfile

    untuk menghapus image docker command nya adalah
    - docker rmi <ID> -f
==============DOCKERRIZE=====================

==============SERVER====================

- sintak untuk masuk ke docker file fastapi-erp:
    docker exec -ti fastapi-erp sh

- sintak untuk melihat cronjob di server:
    crontab -e

- sintak run scheduler manual on SERVER:
    docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims:1.0.2 sh -c "python src/jobs/run_sync_data_member.py

- show log service on server
    docker logs fastapi-erp -f
==============SERVER====================

==============DEVELOP DOCKER DI SERVER=======

- buat folder di opt/nama_project
- login docker : docker login registry.msglow.cloud
- lakukan docker pull : docker pull <nama image> lihat di git packages & registries -> container registry -> project
- lakukan stop versi docker sebelumnya : docker stop <name> example fastapi-erp
- lakukan remove versi docker sebelumnya : docker rm <name> example fastapi-erp
- run docker : docker run -d --log-driver gelf --log-opt gelf-address=udp://180.131.130.6:5044 --restart always --name fastapi-erp -p 8081:8007 -v /opt/docker/fastapi-erp/config.py:/app/core/config.py -v /opt/docker/fastapi-erp/supervisord.conf:/etc/supervisord.conf -v /opt/docker/fastapi-erp/report:/app/static/reports -e TZ=Asia/Jakarta registry.msglow.cloud/services/service-ims/fastapi-erp:<version>

- run docker service payment
docker run -d --log-driver gelf --log-opt gelf-address=udp://180.131.130.6:5044 --restart always --name fastapi-payment -p 8085:8010 -v /opt/docker/fastapi-payment/config.py:/app/core/config.py -v /opt/docker/fastapi-payment/config_community_men.py:/app/core/config_community_men.py -v /opt/docker/fastapi-payment/supervisord.conf:/etc/supervisord.conf -e TZ=Asia/Jakarta registry.msglow.cloud/services/xendit-payment-service/fastapi-payment:<version>

- run docker service ims
docker run -d --log-driver gelf --log-opt gelf-address=udp://180.131.130.6:5044 --restart always --name fastapi-erp -p 8081:8007 -v /opt/docker/fastapi-erp/config.py:/app/core/config.py -v /opt/docker/fastapi-erp/supervisord.conf:/etc/supervisord.conf -v /opt/docker/fastapi-erp/report:/app/static/reports -v /opt/docker/fastapi-erp/uploads/product_images:/app/static/uploads/product_images -v /opt/docker/fastapi-erp/uploads/documents:/app/static/uploads/documents -v /opt/docker/fastapi-erp/uploads/product_certificates:/app/static/uploads/product_certificates -e TZ=Asia/Jakarta registry.msglow.cloud/services/service-ims/fastapi-erp:<verison>

- untuk melihat list service docker yang sedang berjalan : docker ps -a
- untuk melihat list image docker : docker images
- untuk delete image docker : docker rmi <id>
==============DEVELOP DOCKER DI SERVER=======

===============SETTING CRONTAB ON SERVER===================
45 0 * * * /root/script/clearcache.sh > /dev/null 2>&1
0 1 * * * /root/script/clearSession.sh > /dev/null
0 23 * * * /bin/find /opt/docker/fastapi-erp/report/* -mtime +7 -delete > /dev/null 2>&1

0 4,7,10,13,16,19,22,1 * * * /bin/docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims sh -c "python src/jobs/run_sync_data_member.py" >> /var/log/ims/sync_data_member.log

10 0 * * * /bin/docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims sh -c "python src/jobs/run_daily_stock_produk.py" >> /var/log/ims/daily_stock_produk.log

40 0 * * * /bin/docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims sh -c "python src/jobs/run_daily_hpp_produk.py" >> /var/log/ims/daily_hpp_produk.log

5 23 * * * /bin/docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims sh -c "python src/jobs/run_closing_daily_stock_produk.py" >> /var/log/ims/closing.log

55 0 * * * /bin/docker run --rm -v /opt/docker/fastapi-erp/cron/config.py:/app/core/config.py -t fastapi-scheduler-ims sh -c "python src/jobs/run_opening_stock_produk.py" >> /var/log/ims/opening_stock_produk.log

# 0 */3 * * * /home/vicky/backup_db_scripts/backup_tables.sh >> /home/vicky/backup_database/backup_log.txt 2>&1
===============SETTING CRONTAB ON SERVER===================

allow 103.164.116.198;
allow 103.164.116.196;
allow 36.95.228.196;
allow 103.127.169.178;
allow 103.158.162.82:
allow 36.95.228.194;

===============SETTING TAGING SCHEDULER IMS==================
docker tag registry.msglow.cloud/services/scheduler-ims/fastapi-scheduler-ims:1.0.13 fastapi-scheduler-ims:latest
