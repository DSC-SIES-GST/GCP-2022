# Getting Started: Create and Manage Cloud Resources: Challenge Lab

> Launch the lab [here](https://google.qwiklabs.com/focuses/10258?parent=catalog)
## Your challenge

As soon as you sit down at your desk and open your new laptop you receive several requests from the Nucleus team. Read through each description, then create the resources.

Change the following in the codes ```<INSTANCE NAME>, <LAB ZONE>, <LAB REGION>, <PORT>, <FIREWALL RULE>.```

## Solving tasks

### Task 1: Create a project jumphost instance

* Run the following from the **Cloud Terminal**:

```yaml
gcloud compute instances create <INSTANCE NAME> \
          --network nucleus-vpc \
          --zone <LAB ZONE>  \
          --machine-type f1-micro  \
          --image-family debian-9  \
          --image-project debian-cloud \
          --scopes cloud-platform \
          --no-address
```
### Task 2: Create a Kubernetes service cluster

* Run the following from the **Cloud Terminal**:

```yaml
gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network nucleus-vpc \
          --region <LAB REGION>
gcloud container clusters get-credentials nucleus-backend \
          --region <LAB REGION>
kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port <PORT>
```

### Task 3: Setup an HTTP load balancer

* Run the following from the **Cloud Terminal**:

```yaml
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region <LAB REGION>
gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region <LAB REGION>
gcloud compute firewall-rules create <FIREWALL RULE> \
          --allow tcp:80 \
          --network nucleus-vpc
gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region <LAB REGION>
gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global
gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region <LAB REGION> \
          --global
gcloud compute url-maps create web-server-map \
          --default-service web-server-backend
gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
gcloud compute forwarding-rules list
```
