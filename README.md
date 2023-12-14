# Build-Secure-Networks-GCP

## Scenario

- You are a security consultant brought in by Jeff, who owns a small local company, to help him with his very successful website (juiceshop). Jeff is new to Google Cloud and had his neighbor's son set up the initial site.
- The neighbor's son has since had to leave for college, but he made sure the site was running. Help Jeff and perform appropriate configuration for security. Below is the current situation:

![image](https://i.imgur.com/g3svAtn.png)

### Tasks

- To configure this simple environment securely. The first task is to set up appropriate firewall rules and virtual machine tags. Also, need to ensure that SSH is only available to the bastion via IAP.

For the firewall rules, make sure:

• The bastion host does not have a public IP address.

• You can only SSH to the bastion and only via IAP.

• You can only SSH to juice-shop via the bastion.

• Only HTTP is open to the world for juice-shop.

![image](https://i.imgur.com/3OKeeT0.png)


**Step 1. Check the firewall rules. Remove the overly permissive rules.**

- Remove the overly permissive rules
```
export IAP_NETWORK_TAG=grant-ssh-iap-ingress-gl-357

export INTERNAL_NETWORK_TAG=grant-ssh-internal-ingress-gl-357

export HTTP_NETWORK_TAG=grant-http-ingress-gl-357

export ZONE=us-central1-b

gcloud compute firewall-rules delete open-access
```

![image](https://i.imgur.com/BW12y4f.png)

**Step 2. Navigate to Compute Engine in the Cloud Console and identify the bastion host.**
The instance should be stopped and restarted.

**Step 3. Make the bastion host the machine authorized to receive external SSH traffic.**

- Create a firewall rule that allows SSH (tcp/22) from the IAP service.
- The firewall rule must be enabled for the bastion host instance using a network tag of SSH IAP network tag.
```
gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags $IAP_NETWORK_TAG --network acme-vpc
 
gcloud compute instances add-tags bastion --tags=$IAP_NETWORK_TAG --zone=$ZONE
```

![image](https://i.imgur.com/AXg6yYU.png)

**Step 4. The juice-shop server serves HTTP traffic.**

- Create a firewall rule that allows traffic on HTTP (tcp/80) to any address.
- The firewall rule must be enabled for the juice-shop instance using a network tag of HTTP network tag.
```
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags $HTTP_NETWORK_TAG --network acme-vpc
 
gcloud compute instances add-tags juice-shop --tags=$HTTP_NETWORK_TAG --zone=$ZONE
```

![image](https://i.imgur.com/GZvyRyW.png)

**Step 5. Connect to juice-shop from the bastion using SSH.**

- Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address.
- The firewall rule must be enabled for the juice-shop instance using a network tag of SSH internal network tag.
```
gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags $INTERNAL_NETWORK_TAG --network acme-vpc
 
gcloud compute instances add-tags juice-shop --tags=$INTERNAL_NETWORK_TAG --zone=$ZONE
```

**Step 6. In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to juice-shop.**

![image](https://i.imgur.com/OjQDnJp.png)

## Reference
https://www.cloudskillsboost.google
