# README: Instructions on How to Start a Minecraft Server

## Steps

1. Go to EC2 and click **Launch Instance**.
2. Under **Name and Tags**, enter **Minecraft Server**.
3. In **Applications and OS Images**, choose **Amazon Linux 2023 AMI** with the **64-bit (Arm) version**.
4. Select **t4g.small** under **Instance Type** (if you do not see this, you did not select **64-bit (Arm)**).
5. For **Key Pair (login)**, use an existing key or create a new key and save the `.pem` file to your desktop. You will need this to SSH into the EC2 instance to run a bash script later.
6. Under **Network Settings**, edit and select **Default VPC** and **No preference** for subnet. Ensure **Auto-assign Public IP** is enabled.
7. Under **Firewall**, select **Create security group**. Name your security group **Minecraft Security Group** and enter **Minecraft Security Group** under description.
8. Under **Inbound security group rules**, click **Add security group rule** and select **Anywhere** under **Source type**. Add `0.0.0.0/0` as the source and `25565` as the port range.
9. Click **Launch Instance**.
10. Navigate to the EC2 instance dashboard and on the left, click **Instances**. This should show you a list of instances.
11. Select the checkbox next to **Minecraft Server** and under **Actions**, select **Connect**.
12. Under **Connect**, there should be a tab called **SSH Client**. Click it and copy the example command at the bottom to connect to your EC2 instance using the copied command.
   - Note: You will need to be in the directory where your key file is located.
13. Once connected, run the following commands to create a bash script:

```touch loop.bash```

```vim loop.bash```

14) push i on your keyboard and pase the fallowing program into the file 
```
#!/bin/bash

# *** INSERT SERVER DOWNLOAD URL BELOW ***
# Do not add any spaces between your link and the "=", otherwise it won't work. EG: MINECRAFTSERVERURL=https://urlexample


MINECRAFTSERVERURL=https://piston-data.mojang.com/v1/objects/84194a2f286ef7c14ed7ce0090dba59902951553/server.jar


# Download Java
sudo yum install -y java-17-amazon-corretto-headless
# Install MC Java server in a directory we create
adduser minecraft
mkdir /opt/minecraft/
mkdir /opt/minecraft/server/
cd /opt/minecraft/server

# Download server jar file from Minecraft official website
wget $MINECRAFTSERVERURL

# Generate Minecraft server files and create script
chown -R minecraft:minecraft /opt/minecraft/
java -Xmx1300M -Xms1300M -jar server.jar nogui
sleep 40
sed -i 's/false/true/p' eula.txt
touch start
printf '#!/bin/bash\njava -Xmx1300M -Xms1300M -jar server.jar nogui\n' >> start
chmod +x start
sleep 1
touch stop
printf '#!/bin/bash\nkill -9 $(ps -ef | pgrep -f "java")' >> stop
chmod +x stop
sleep 1

# Create SystemD Script to run Minecraft server jar on reboot
cd /etc/systemd/system/
touch minecraft.service
printf '[Unit]\nDescription=Minecraft Server on start up\nWants=network-online.target\n[Service]\nUser=minecraft\nWorkingDirectory=/opt/minecraft/server\nExecStart=/opt/minecraft/server/start\nStandardInput=null\n[Install]\nWantedBy=multi-user.target' >> minecraft.service
sudo systemctl daemon-reload
sudo systemctl enable minecraft.service
sudo systemctl start minecraft.service

# End script
```
push esc and :wq to save and quit

15. Run the command ```chmod +x loop.bash``` to make the file executable.
16. Run the command ```bash loop.bash``` and then wait for the system to run.
17. You should now be able to connect to the server in Minecraft 1.20.1 using the IP and port ```<your_public_ipv4>:25565```.
18. your public ipv4 addreess can be found at the ec2 instances dashbord and clicking the checkbox. At the bottom it should say **Public IPv4 address**
