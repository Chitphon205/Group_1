# Wed_Group_1 คะแนนที่ได้ 100/100
##คำสั่งที่ใช้
gcloud config set compute/region europe-west4**เปลี่ยนเป็นของเรา**
export REGION=europe-west4  **เปลี่ยนเป็นของเรา**
export Zone=europe-west4-c    **เปลี่ยนเป็นของเรา**

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

gcloud compute instance-templates create my-template \
        --metadata-from-file startup-script=startup.sh \
        --machine-type=e2-medium


gcloud compute instance-groups managed create my-instance-group \
        --base-instance-name web-server \
        --size 2 \
        --template my-template


gcloud compute firewall-rules create allow-tcp-rule-259 \  **เปลี่ยนเป็นชื่อของเรา**
  --network=default \
  --action=allow \
  --rules=tcp:80


gcloud compute http-health-checks create my-health-check


gcloud compute instance-groups managed \
        set-named-ports my-instance-group \
        --named-ports http:80 

gcloud compute backend-services create my-backend-service \
        --protocol HTTP \
        --http-health-checks my-health-check \
        --global


gcloud compute backend-services add-backend my-backend-service \
        --instance-group my-instance-group \
        --global

gcloud compute url-maps create my-server-map \
        --default-service my-backend-service

gcloud compute target-http-proxies create http-lb \
        --url-map my-server-map

gcloud compute forwarding-rules create my-rule \
      --global \
      --target-http-proxy http-lb \
      --ports 80

gcloud compute forwarding-rules list
