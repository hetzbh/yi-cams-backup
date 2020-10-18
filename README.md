# yi-cams-backup
How to backup videos from Yi based cameras

Many people buy their security cameras from Yi technologies. Those cameras are working pretty well, and they are cheap.
However, those cameras are being sold as "Cloud camera" and as a user, you can either save the video temporarily to a MicroSD, or you can use their cloud offering, with some serious high price tags, and the reocrding stays on the cloud for 7-30 days, so you cannot (officially) have an archive footage and you cannot create a local backup.

Thankfully, projects created [like this](https://github.com/TheCrypt0/yi-hack-v4) by TheCrypt0 and [this project](https://github.com/roleoroleo/yi-hack-MStar) by roleoroleo - you can hack your camera, and within few minutes, you can use your camera again and have some extra functionality.

In order to use these instructions, you'll need to "hack" your Yi camera. most of those cameras are supported by one of the projects. Please check both pages and follow their instructions to download a file, extract it to a MicroSD, insert the MicroSD to camera and power on. After a 30-60 seconds, your camera should have blue light - which means the hack worked. Check what is the IP of your camera using the Yi app (or Kama app), and from a browser, browse to: http://IP:8080 (replace the IP with the actual IP). Don't forget to set the username and password, and the root password. After that, go to the maintenance menu and reboot. 

with the new hack, you'll have new options, such as SSH, FTP, Telnet etc. You'll can also see in the menu (under the Motion Events) an option to use FTP to upload your videos from your camera to any NAS storage which supoprts FTP, and you can select either to erase the original file or keep it on the MicroSD.

The problems with FTP are:
* You cannot synchronize between the camera and your NAS storage
* FTP is totally insecure
* You cannot select how many days to synchronize
* You cannot use any security communication between the camera and the NAS (note: the camera, after hack, include SCP, but it's not in the GUI)

What I would suggest, is using another tool to create a synchronized backup. The tool: **[rsync](https://en.wikipedia.org/wiki/Rsync)**

With rsync, you can:
* Connect securly to the camera so perform backup
* You can set the amount of days/hours to backup (see [here](https://www.magic4hosting.com/eng/knowledge-base/sync-files-with-rsync-newer-then-x-days/) for example)
* You can set the amount of time from the actual record until it's synced to the NAS (no more waiting few hours for the files to appear on the NAS)
* You can use SSH keys to connect to the camera and prevent password logins.

Requirements:
* A Linux machine, or a Synology NAS, or FreeNAS - any machine that supports rsync, which will be the destination machine
* A bit of Linux knoeledge (I'm just giving the basics, feel free to modify it as you wish)
* SSH to the camera.

Lets begin:
1. Login to your camera **as root** (use SSH, and disable Telnet, brw) with the user **root** and the IP of the camera (example: `ssh root@192.168.100.3`). By default it does not require password
2. Type the following commands: 
```sh
mkdir -p /tmp/sd/yi-hack/bin && mkdir -p /tmp/sd/yi-hack/lib
cd /tmp/sd/yi-hack/bin
wget https://github.com/JBBgameich/rsync-static/releases/download/continuous/rsync-arm
mv rsync-arm rsync
cd /usr/bin
ln -s /tmp/sd/yi-hack/bin/rsync .
```
3. Type `exit` to logout from the camera
4. Lets perform a simple test to see if a remote rsync works: type `ssh root@IP rsync` (don't forget to replace IP with the IP of the camera). If everything works, you should see lots of text from the rsync command output. If not, something went wrong, go back and follow the instructions.

Thats it. From here, go to your Linux/FreeNAS/NAS and create a cron job to backup your videos. The videos are located at **/tmp/sd/record/**

The video files are saved in a specific directory and file names, using this format:

Example: 2020Y10M18D03H/31M00S60.mp4

Where: 

Directory name:
2020Y - is the year
10M - is the month
18D - is the day in the month
03H - is the hour (in 24 hours format)

Filename format:
31M - Is the minute 
00S - Is the second the camera started recording
60 - Is the duration of the recording.

Please note: each recording length is maximum 60 seconds, and then the camera creates another file, etc.. etc..
If the filename has 00S, it probably means it's a continuation from a previous file (if you're into joining MP4 files - do it with FFMPEG). 

Setting rsync on Linux:

If you want to run rsync from a Linux machine, you can perform the following steps:

1. Create a passwordless SSH key on your Linux machine (example: `ssh-keygen`), skip the passphrase (or else you might have issues with cron)
2. Type `ssh-copy-id root@IP` (where IP is the IP of the camera), type the root password for the camera
3. From now on, you can SSH the camera from your user in your Linux machine without password
4. Create a directory which will store the files. If you plan to grab all the videos and keep them, then prepare a big storage solution (preferable a RAID solution)
5. Here is an rsync command example: `rsync -av -P root@IP:/tmp/sd/record/ /media/cctv-videos/`

Where:

IP - the IP of your camera
/media/cctv-videos/ - is the directory where to copy the directories and files to

After succesful first time copying, create a cron to do the job for you. [See here](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/) for instructions how to use cron.

Notes:
* DO NOT use compression (`-z` or `--compress`) - all files are MP4 and you cannot compress them. Also, the CPU on the camera is very weak, compression will kill it
* You might not be able to access the camera from your phone while copying the files to your destination machine (Weak CPU, remember?), so I suggest to do a sync at night
* If you're replacing the MicroSD, re-install the rsync binary. **DO NOT** copy the binary to the internal storage, or else your camera will be "bricked"
* If you only want to copy X amount of days from the camera, you might want to take a look [here](https://www.magic4hosting.com/eng/knowledge-base/sync-files-with-rsync-newer-then-x-days/) how to perform this.
* The detection of the Yi cameras is really bad, and it will record tons of useless video. Enable Motion, and set it to Low (specially if you have any animal at your house)

Credits:
* TheCrypt0 and roleoroleo for their projects and investment in the project
* JBBgameich - for the static arm build of rsync binary

Good Luck ;)
