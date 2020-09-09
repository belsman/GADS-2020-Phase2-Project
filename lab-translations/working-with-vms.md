# Working with Virtual Machines

## Overview

> In this lab, you set up a game application—a Minecraft server.

> The Minecraft server software will run on a Compute Engine instance.

> You use an n1-standard-1 machine type that includes a 10-GB boot disk, 1 virtual CPU (vCPU), and 3.75 GB of RAM. This machine type runs Debian Linux by default.

> To make sure there is plenty of room for the Minecraft server's world data, you also attach a high-performance 50-GB persistent solid-state drive (SSD) to the instance. This dedicated Minecraft server can support up to 50 players.

## Objectives

> In this lab, you learn how to perform the following tasks:

* Customize an application server

* Install and configure necessary software

* Configure network access

* Schedule regular backups

## Task 1: Create the VM
> Create the VM (**mc-server**) with the specs stated in the overview with the following command:

gcloud compute instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --address=35.226.222.182 --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=25158464534-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --tags=minecraft-server --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-00-0b61e61d3a88/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

> This creates a VM in the default VPC with a reverse external IP address.

## Task 2: Prepare the data disk

### Create a directory and format and mount the disk

1. For mc-server, click SSH to open a terminal and connect.

2. To create a directory that serves as the mount point for the data disk, run the following command:

   sudo mkdir -p /home/minecraft

3. To format the disk, run the following command:

   sudo mkfs.ext4 -F -E lazy_itable_init=0, lazy_journal_init=0,discard, /dev/disk/by-id/google-minecraft-disk

4. To mount the disk, run the following command:

   sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft


## Task 3: Install and run the application

> The Minecraft server runs on top of the Java Virtual Machine (JVM), so it requires the headless Java Runtime Environment (JRE) to run.

### Install the Java Runtime Environment (JRE) and the Minecraft server

1. In the SSH terminal for mc-server, to update the Debian repositories on the VM, run the          following command:
  
  sudo apt-get update

2. After the repositories are updated, to install the headless JRE, run the following command:
  
  sudo apt-get install -y default-jre-headless

3. To navigate to the directory where the persistent disk is mounted, run the following command:

  cd /home/minecraft

4. To install wget, run the following command:

  sudo apt-get install wget

5. If prompted to continue, type **Y**

6. To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:

  sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar


### Initialize the Minecraft server

1. To initialize the Minecraft server, run the following command:

  sudo java -Xmx1024M -Xms1024M -jar server.jar nogui

  **The Minecraft server won't run unless you accept the terms of the End User Licensing Agreement (EULA).**

2. To see the files that were created in the first initialization of the Minecraft server, run the following command:

  sudo ls -l

  **You could edit the server.properties file to change the default behavior of the Minecraft server.**

3. To edit the EULA, run the following command:

  sudo nano eula.txt

4. Change the last line of the file from **eula=false** to **eula=true**

5. Press Ctrl+O, ENTER to save the file and then press Ctrl+X to exit nano.

  **Don't try to restart the Minecraft server yet. You use a different technique in the next procedure.**


### Create a virtual terminal screen to start the Minecraft server

> If you start the Minecraft server again now, it is tied to the life of your SSH session: that is, if you close your SSH terminal, the server is also terminated. To avoid this issue, you can use screen, an application that allows you to create a virtual terminal that can be "detached," becoming a background process, or "reattached," becoming a foreground process. When a virtual terminal is detached to the background, it will run whether you are logged in or not.

1. To install screen, run the following command:

  sudo apt-get install -y screen

2. To start your Minecraft server in a **screen** virtual terminal, run the following command: (Use the -S flag to name your terminal mcs)

  sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui

### Detach from the screen and close your SSH session

1. To detach the screen terminal, press **Ctrl+A**, **Ctrl+D**. The terminal continues to run in the background. To reattach the terminal, run the following command:

  sudo screen -r mcs


2. To exit the SSH terminal, run the following command:
 
  exit

**Congratulations! You set up and customized a VM and installed and configured application software—a Minecraft server!**

## Task 4: Allow client traffic

> Up to this point, the server has an external static IP address, but it cannot receive traffic because there is no firewall rule in place. Minecraft server uses TCP port 25565 by default. So you need to configure a firewall rule to allow these connections.

### Create a firewall rule

gcloud compute --project=qwiklabs-gcp-00-0b61e61d3a88 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server


## Task 5: Schedule regular backups

> Backing up your application data is a common activity. In this case, you configure the system to back up Minecraft world data to Cloud Storage.

1. SSH to the mc-server

2. Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME. To make it unique, you can use your Project ID. Run the following command:

  export YOUR_BUCKET_NAME=<Enter your bucket name here>

3. Verify it echos

  echo $YOUR_BUCKET_NAME

4. To create the bucket run the following command:

  gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup

  **If this command failed, you might not have created a unique bucket name. If so, choose another bucket name, update your environment variable, and try to create the bucket again.**

5. Verify bucket creation

### Create a backup script

1. In the mc-server SSH terminal, navigate to your home directory:

  
2. To create the script, run the following command:

    sudo nano /home/minecraft/backup.sh 

3. Copy and paste the following script into the file:

  #!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'

4. Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano.

**The script saves the current state of the server's world data and pauses the server's auto-save functionality. Next, it backs up the server's world data directory (world) and places its contents in a timestamped directory (<timestamp>-world) in the Cloud Storage bucket. After the script finishes backing up the data, it resumes auto-saving on the Minecraft server.**

5. To make the script executable, run the following command:

  sudo chmod 755 /home/minecraft/backup.sh

### Test the backup script and schedule a cron job

1. In the mc-server SSH terminal, run the backup script:

  . /home/minecraft/backup.sh

2. To verify that the backup file was written.

**Now that you've verified that the backups are working, you can schedule a cron job to automate the task.**


3. In the mc-server SSH terminal, open the cron table for editing:

  sudo crontab -e

4. When you are prompted to select an editor, type the number corresponding to nano, and press ENTER.

5. At the bottom of the cron table, paste the following line:

  0 */4 * * * /home/minecraft/backup.sh

  **That line instructs cron to run backups every 4 hours.**

6. Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano.


## Task 6: Server maintenance

> To perform server maintenance, you need to shut down the server.

### Connect via SSH to the server, stop it and shut down the VM

1. In the mc-server SSH terminal, run the following command:

  sudo screen -r -X stuff '/stop\n'

2. Shut down the server with this command:

### Automate server maintenance with startup and shutdown scripts

> Instead of following the manual process to mount the persistent disk and launch the server application in a screen, you can use metadata scripts to create a startup script and a shutdown script to do this for you.


### Task 7: Review

> In this lab, you created a customized virtual machine instance by installing base software (a headless JRE) and application software (a Minecraft game server). You customized the VM by attaching and preparing a high-speed SSD data disk, and you reserved a static external IP so the address would remain consistent. Then you verified availability of the gaming server online. You set up a backup system to back up the server's data to a Cloud Storage bucket, and you tested the backup system. Then you automated backups using cron. Finally, you set up maintenance scripts using metadata for graceful startup and shutdown of the server. All via the command-line.
