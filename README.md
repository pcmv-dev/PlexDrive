
<center>
<h1 align="center">PlexDrive</h1>
<h4 align="center">Automate Uploads to Cloud Storage</h4>

# Info
Automate uploading to your cloud storage. This uses "Mergerfs" for compatibility with Sonarr, Radarr, and streaming to Plex/Emby. Automate uploading from a local or remote machine running Ubuntu 18.04 or Debian 9/10.

# About
This guide will help you get started and is by no means the best way of doing things... this is what works for me. I created this repo for my own reference.

# Disclaimer

I am not responsible for anything that could go wrong. I am not responsible for any data loss that could potentialy happen. You agree to use these scripts at your own risk.

# Requirements
- Script has only been tested on **Debian 9/10** and **Ubuntu 18.04+**

:warning: Script is still experimental!
# Installation
1. Download the PlexDrive repo
```
sudo apt update && sudo apt install git -y && sudo git clone https://github.com/pcmv-dev/PlexDrive.git /opt/plexdrive
```
2. Set permissions and make scripts executable
```
sudo chmod -R +x /opt/plexdrive/install && sudo chmod +x /opt/plexdrive/plexdrive-upload && sudo chown -R $USER:$USER /opt/plexdrive
```
3. Run the installer script to install Rclone, MergerFS, and Fuse
```
sudo bash /opt/plexdrive/install/plexdrive
```
4. Create the mount directory and set permissions
```
sudo mkdir /mnt/plexdrive && sudo chown $USER:$USER /mnt/plexdrive
```
5. Install automount services
```
sudo cp /opt/plexdrive/systemd/* /etc/systemd/system/ && \
sudo systemctl daemon-reload && \
sudo systemctl enable rclone.service && \
sudo systemctl enable plexdrive.service
```
# Change Fusermount Permission

If the script fails to modify fuse.conf you can do this manually

You must edit  /etc/fuse.conf to use option "allow_other" by uncommenting "user_allow_other"
```
sudo nano /etc/fuse.conf
```
# Configure Rclone

The install script should set your permissions for your **rclone.conf** but if not run the following.
If you don't set permissions rclone will complain that it is needs sudo/root to run, which we do not want.
```
sudo chown -R $USER:$USER $HOME/.config/rclone
```
Create your rclone.conf
```bash
rclone config
```
I assume most use Google Drive so make sure you create your own client_id 
> [MAKING YOUR CLIENT ID](https://rclone.org/drive/#making-your-own-client-id)

```bash
[googledrive]
type = drive
client_id = <CLIENTID>
client_secret = <CLIENTSECRET>
scope = drive
token = {"access_token":"**********"}
server_side_across_configs = true

[googledrive_encrypt]
type = crypt
remote = googledrive:encrypt
filename_encryption = standard
directory_name_encryption = true
password = <PASSWORD>
password2 = <PASSWORDSALT>
```

# Upload Script
### Do not run the script as sudo/root unless you are running everything as root or you will have permission problems
### This script uploads new files to your cloud storage
Configure the **plexdrive-upload** script. You only need to modify "REMOTE AND MOUNT" "SERVICE ACCOUNTS" and "DISCORD NOTIFICATIONS"
```bash
# REMOTE AND MOUNT
REMOTE="plexdrive" # Name of rclone remote mount NOTE: Choose your encrypted remote for sensitive data
UPLOADREMOTE="plexdrive" # If you have a second remote created for uploads put it here. Otherwise use the same remote as REMOTE
MEDIA="plexdrive" # Local share name NOTE: The name you want to give your share mount

# SERVICE ACCOUNTS
# Drop your .json files in your "/opt/plexdrive/rclone/service_accounts"
# Name them "sa_account1.json" "sa_account2.json" etc.
USESERVICEACCOUNT="Y" # Y/N. Choose whether to use Service Accounts NOTE: Bypass Google 750GB upload limit
SERVICEACCOUNTNUM="25" # Integer number of service accounts to use.

# DISCORD NOTIFICATIONS
DISCORD_WEBHOOK_URL="" # Enter your Discord Webhook URL for notifications. Otherwise leave empty to disable
DISCORD_ICON_OVERRIDE="https://raw.githubusercontent.com/rclone/rclone/master/graphics/logo/logo_symbol/logo_symbol_color_256px.png" # The bot user image
DISCORD_NAME_OVERRIDE="RCLONE" # The bot user name
```
# Setup Cron Job

## Manual Entry
> Add plexdrive-upload to crontab

> Example: 0 */5 * * * /opt/plexdrive/plexdrive-upload 2>&1
```
$ crontab -e
```
## Using Provided Script in the "Install" folder
Don't run as root!
```
$ bash /opt/plexdrive/install/addcron
```

[Crontab Calculator](https://corntab.com/)

## Credits
This project makes use of, integrates with, or was inspired by the following projects:

* [BinsonBuzz](https://github.com/BinsonBuzz/unraid_rclone_mount) Original Rclone Scripts
* [no5tyle](https://github.com/no5tyle/UltraSeedbox-Scripts) for Discord notifications
