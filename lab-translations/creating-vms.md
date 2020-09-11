# Creating Virtual Machines

## Overview
> In this lab, you will explore the Virtual Machine instance options and create several VMs with different characteristics.

## Objectives

> In this lab, you explore the available options for VMs and see the differences between locations.

> In this lab, you learn how to use the command-line options to perform the following tasks:

* Create a utility VM
* Create a windows VM
* Create a custom VM

### Task1: Create a utility virtual machine from the Command-line
> To create a standard virtual machine called **utility-vm** in us-central1-c with 1vCPU and 3.75GB memory and no external IP. Use the following command:

```gcloud compute instances create utility-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=utility-vm --reservation-affinity=any```

### Task 2: Create a Windows virtual machine from the Command-line
> To create a windows image virtual machine **window-vm** in the europe-west2-a zone; with 1vCPU and 7GB memory; 100GB of boot-disk size of an SSD persistent type, run the
following command:

```gcloud compute instances create windows-vm --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=5251233221-compute@developer.gserviceaccount.com --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=windows-vm --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any```

> Enable **HTTP Traffic and HTTPS Traffic**. The following commands enable ingress http(s) traffic into the nextwork as specified ports.

```gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server```

```gcloud compute firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server```


### Task 3: Create a custom virtual machine from the Command-line
> To create a **custom-vm** within the us-wes1-b zone; with 6 vCPU and 32GB memory on a debian-9 image, run the following:

```gcloud compute instances create custom-vm --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=custom-vm --reservation-affinity=any```

> List the virtual machines in your current project by running the following command.
```gcloud compute instances list```


### SSH to the custom-vm from the Command-line

* Use ``gcloud compute ssh custom-vm`` to **ssh** into the vm

* To see information about unused and used memory and swap space on your custom VM, run the following command:
   ```free```

* To see details about the RAM installed on your VM, run the following command:
  ```sudo dmidecode -t 17```

* To verify the number of processors, run the following command:
  ```nproc```

* To see details about the CPUs installed on your VM, run the following command:
  ```lscpu```

* To exit the SSH terminal, run the following command:
  ```exit```

### Thanks for completing this lab.
