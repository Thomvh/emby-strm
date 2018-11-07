# Pre-reqs
## Make user filesystem
sudo mkdir /u01; sudo chmod 777 /u01

## Install pre-reqs
sudo apt-get -y install python-crypto acl libwww-perl

## Install emby (change package for newest stable version)
wget https://github.com/MediaBrowser/Emby.Releases/releases/download/3.5.3.0/emby-server-deb_3.5.3.0_amd64.deb
sudo dpkg -i emby-server-deb_3.5.3.0_amd64.deb

## Clone git's
cd /u01
git clone https://github.com/ddurdle/GoogleDrive-VideoStream_extra.git
git clone https://github.com/ddurdle/Python-GoogleDrive-VideoStream.git

## Download ffmpeg build
wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-64bit-static.tar.xz
tar -xvf ffmpeg-release-64bit-static.tar.xz
rm ffmpeg-release-64bit-static.tar.xz

## Make the emby directories
mkdir /u01/emby-bk; mkdir /u01/emby-cache; mkdir /u01/emby-metadata; mkdir /u01/emby-transcode; mkdir /u01/recordings; mkdir /u01/STRM; ln -sf /u01/STRM /u01/data; sudo chmod 775 /u01/emby-*;sudo chmod 775 /u01/recordings; sudo chown durdle:emby /u01/emby-*; sudo chown durdle:emby /u01/recordings

## Setup swap
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon -s

# Networking for GCE
## Update firewall
in GCE->VPC Network->Firewall rules
create a firewall rule:
name: emby
targets: all instances in the network
source IP range: 0.0.0.0/0
specified protocols/ports: tcp:8096

create a firewall rule:
name: google-videostream
targets: all instances in the network
source IP range: 0.0.0.0/0
specified protocols/ports: tcp:9988;tcp:9989

# Cleanup script
#/u01/cleanup-tmp.sh
find /var/lib/emby/transcoding-temp/ -type f -mmin +60 -delete
chmod +x /u01/cleanup-tmp.sh

## Setup crontab
sudo crontab -e
56 11 * * * /usr/sbin/service emby-server restart
0 * * * * sh /u01/cleanup-tmp.sh


# Emby Settings 
## Settings to update in Emby
- name of instance
- Settings, Cache Path = /u01/emby-cache
- emby premier (add key)
- Library->Advanced, Metadata path = /u01/emby-metadata
- add opensubtitles login
- Transcoding, Transcoding temporary path =  /u01/emby-transcode
- Transcoding, disable "allow subtitle extraction on the fly"


# Videostream settings
## Settings to update in videostream
- Settings, add username + password for ui
- Settings, Server log = /tmp/server.log
- Settings, Scheduler log = /tmp/scheduler.log
- Settings, Passthrough = true
- enroll account
- schedule task, extraction select account enrolled, select drive/teamdrive enter path (/u01/STRM/), change download nfo/srt= true, change cataloge to false

# Videostream extra's
## Setup video_stream extra's
- Copy ffmpeg and ffprobe to ffmpeg.oem and ffprobe.oem in /opt/emby-server/bin
  cp ffmpeg ffmpeg.oem
  cp ffprobe ffprobe.oem
- Copy config.cfg from transcoders folder to /opt/emby-server/bin
  cp /u01/GoogleDrive-VideoStream_extra/transcoders/config.cfg /opt/emby-server/bin/config.cfg
- Symlink emby_ffmpeg.pl and emby_ffprobe.pl to /opt/emby-server/bin as ffmpeg and ffprobe
  ln -s /u01/GoogleDrive-VideoStream_extra/transcoders/emby_ffmpeg.pl /opt/emby-server/bin/ffmpeg
  ln -s /u01/GoogleDrive-VideoStream_extra/transcoders/emby_ffmprobe.pl /opt/emby-server/bin/ffmprobe
- Retrieve a copy of the /opt/emby-server/lib and ffmpeg + ffprobe from a known good emby (such as 3.3.1.12), place the lib folder as
  /opt/emby-server/lib.332 and ffmpeg and ffprobe in /opt/emby-server/bin as
  ffmpeg.oem.332 and ffprobe.oem.332 -- making it line up with the paths
  noted in variable FFMPEG_OEM_332 in the config.cfg.  Then change the
  parameter in the config.cfg file ALT_FFMPEG_DETERMINATOR variable to => '';
- Change 'FORCE_REMUX_AUDIO' in the config.cfg to '0'.
  And change 'PATH_TO_FFMPEG' to /u01/ffmpeg-release-64bit-static
