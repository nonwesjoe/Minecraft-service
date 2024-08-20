Minecraft java server launch
==
## First of all
* Lets just say, you are in ubuntu 22 environment, and already have a mc server folder. and your user name is 'max'.
* Get your mc server folder prepared, lets assume the name of it is 'MC' in path /home/max/ and its mc veriosn 1.20.1 fitting java17.  
* so install some packacge with commands below  
sudo apt update  
sudo apt install -y openjdk-17-jdk  
sudo apt install -y screen  
* now in your MC folder run your launch command start.sh or run.sh to start the server. and it is done starting a server! 
## Get your server ip by natfrp tools
* make some pre-jobs for you 'max'.  
mkdir /home/max/natfrp  
sudo apt install -y tar zip unzip
* download then first for linux amd64 or aarch64.  
for amd64  
<code>wget https://nya.globalslb.net/natfrp/client/launcher-unix/3.1.4/natfrp-service_linux_amd64.tar.zst</code>  
for aarch64  
<code>wget https://nya.globalslb.net/natfrp/client/launcher-unix/3.1.4/natfrp-service_linux_arm64.tar.zst</code>
* unzip the file in your folder 'natfrp' and give some premissions.   
tar -I zstd -xvf natfrp-service_*.tar.zst  
rm natfrp-service_*.tar.zst   
sudo chmod +x frpc natfrp-service  
sudo chown max:max frpc natfrp-service
* make a systemd file name natfrp.service in /etc/systemd/system/  
copy below
<pre>[Unit]
Description=SakuraFrp Launcher
After=network.target

[Service]
User=natfrp
Group=natfrp

Type=simple
TimeoutStopSec=20

Restart=always
RestartSec=5s

ExecStart=/home/natfrp/natfrp-service --daemon

[Install]
WantedBy=multi-user.target
</pre>  
and then put it in below command.   
  nano /etc/systemd/system/natfrp.service  
* make some initial jobs
<pre>  
cd /home/max/natfrp/  
systemctl start natfrp.service  
sleep 3  
systemctl stop natfrp.service
</pre>
make sure file below it generated  
ls -ls .config/natfrp-service/  
* then set up your password and login your acconut by  
cd /home/max/natfrp  
./natfrp-service remote-kdf YOUR PASSWORD  
and then you will get your "remote_management_key" DONT LOSE IT.  
nano .config/natfrp-service/config.json  
change the input below  
<pre>{
   "token": "your account key",
   "remote_management": true,
   "remote_management_key": "YOUR PASSWORD AFTER PROCESS",
   "log_stdout": true, 
}
</pre>
* start systemd service
<pre>systemctl enable --now natfrp.service
systemctl status natfrp.service
#see the ouput
journalctl -u natfrp.service -f
</pre>
* go to Web page to maneger remote control  
https://www.natfrp.com/remote/v2
# Make some auto-launch jobs (Optional)
## if your server never turn off then auto-start/stop server  
<pre>mkdir /home/max/autojobs  
cd /home/max/autojobs  </pre>
<code>nano mc.sh</code>  
make a mc.sh file to launch mc-server, put below command in it
<pre>
#!/bin/bash
cd /home/max/MC  
#notice that start.sh is your mc-server launch file(or it could be run.sh)
sh start.sh  
</pre>
use screen session to manager, the session name is 'mc'.

<code>nano screen.sh</code>  
put it down there  
<pre>
#!/bin/bash
screen -S mc -dm
sleep 3
screen -S mc -X stuff "sh /home/max/autojobs/mc $(echo -ne '\r')"
</pre>
also make a turn off script  
<code>nano mcoff.sh</code>  
put it down also  
<pre>#!/bin/bash
screen -S mc -X stuff 'say close in 10 seconds \n'
sleep 12
screen -S mc -X stuff "stop \n"
sleep 65
screen -S mc -X quit
</pre>
give some executable permissions.  
<code>sudo chmod +x *</code>  
finally use crontab to autotask  (i set 10:00 launch and 24:00 turn off server)
<code>crontab -e</code>  
add them (adjustable)  
<pre>
#off at 24:00
0 0 * * * /bin/bash /home/max/autojobs/mcoff.sh
#on at 10:00
0 10 * * * /bin/bash /home/max/autojobs/screen.sh
</pre>
