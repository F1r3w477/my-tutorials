# How to create a dedicated Valheim server on a VPS
This will most likely work on all VPS providers, however, I used [Hyper Expert](https://www.hyperexpert.com/hosting.php).
[Hyper Expert Discord](https://discord.gg/wFvF43C)

## Prerequisites for Hyper Expert
- Based in the United States (Recommended based on server locations)

## Step 1 - Get a VPS server! 1vCPU 3GB of RAM minimum. (May not support full 10 players, max tested on 3GB is 6)
I got mine from from Hyper Expert, if you want one from them follow the steps below:
Go to https://www.hyperexpert.com/hosting.php and select the VPS you want and configure. 
- The bare minimum you should get is their VPS1 with 1vCPU, 3GB of RAM, 20GB storage on Pure SSD (You **MUST** configure the service to upgrade to 3GB) only $5.99 per month
- Make sure you select New York or Seattle as your server location.
- I used Ubuntu 18.04 so I recommend using that as well.

## Step 2 - Get your favorite SSH tool and SSH in.
I recommend [MobaXterm](https://mobaxterm.mobatek.net/download.html "MobaXterm Download")
- Open MobaXterm and choose the "New session" button.
- Remote host is your server IP.
- You don't need to specify a user, however it will be most likely be `root`, check your email to confirm.
- Your password should be in the email that you received. 

## Step 3 - Update your system
Before continuing, make sure you've updated you server by running the command:
```bash
sudo apt update && sudo apt upgrade -y
```
- You will get a popup window asking about restarting services. Choose `yes`
- Later on you will get 2 more popup windows that should default to the second option to leave and use the current file, chose that for both.

## Step 4 - Software dependencies

Now you need to install the following software via `apt` one by one, some should be installed already, but we need to make sure:

- If you get an error after typing one of the follow commands that says `Unable to resolve host <your server name>` follow these steps:

type `sudo nano /etc/hosts` and below where it says `127.0.1.1` type `127.0.0.1` hit tab then type `<your server name>` (This is case sensitive)

```
sudo apt install software-properties-common
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install lib32gcc1 steamcmd
```
## Step 5 - Create a steam user
It is recommended that you create a dedicated user for your server. In our case we will create user `steam` with the following command:
```
sudo useradd -m -s /bin/bash steam
```
We will use this user from the moment, so we can log in with him now:
```
sudo su - steam
```
## Step 6 - Install
As `steam` user we will start the installation process, and for this we need to link the steamcmd (in `steam` home directory):
```bash
ln -s /usr/games/steamcmd steamcmd
```
Running the command will take a while to download the update, but it's necessary:
```
steamcmd
```
After the download is complete, you will be prompted to the steamcmd command line:
```
Steam>
```
Valheim will allow you to download the dedicated server as an anonymous steam user. We will use the following command:
```
Steam> login anonymous
```
Output:
```
Connecting anonymously to Steam Public...Logged in OK
Waiting for user info...OK
```
We will want to install the Valheim server in a certain folder, you are free to use any folder as long as it has permissions for `steam` user, but for the purpose of this tutorial we will use `/home/steam/valheim` folder, so we run the command:
```
Steam> force_install_dir /home/steam/valheim
```
Now we want to download the server files and executables, so first of all we need the SteamAppId, in our case the ID is `896660` and you can check on https://steamdb.info/apps/ and search for "Valheim Dedicated Server".
The command to download is:
```
Steam> app_update 896660 validate
```
Output for downloading:
```
 Update state (0x3) reconfiguring, progress: 0.00 (0 / 0)
...
 Update state (0x61) downloading, progress: 87.82 (917786556 / 1045059920)
 Update state (0x0) unknown, progress: 0.00 (0 / 0)
Success! App '896660' fully installed.
```
Once you see:
```
Steam>
```
Type `quit` and hit enter/return

Now you can check the folder `/home/steam/valheim` to see it's content:
```bash
ls -lha /home/steam/valheim
```
Output:
```bash
total 64M
drwxrwxr-x 5 steam steam 4.0K Feb  7 00:56  .
drwxr-xr-x 4 steam steam 4.0K Feb  7 00:40  ..
drwxrwxr-x 2 steam steam 4.0K Feb  7 00:56  linux64
-rwxrwxr-x 1 steam steam 4.7K Feb  7 00:56  LinuxPlayer_s.debug
-rwxrwxr-x 1 steam steam    2 Feb  7 00:56  server_exit.drp
-rwxrwxr-x 1 steam steam  716 Feb  7 00:56  start_server.sh
-rwxrwxr-x 1 steam steam   34 Feb  7 00:56  start_server_xterm.sh
-rwxrwxr-x 1 steam steam    7 Feb  7 00:56  steam_appid.txt
drwxrwxr-x 5 steam steam 4.0K Feb  7 00:56  steamapps
-rwxrwxr-x 1 steam steam  29M Feb  7 00:56  steamclient.so
-rwxrwxr-x 1 steam steam 6.8M Feb  7 00:56  UnityPlayer_s.debug
-rwxrwxr-x 1 steam steam  29M Feb  7 00:56  UnityPlayer.so
-rwxrwxr-x 1 steam steam 119K Feb  7 00:56 'Valheim Dedicated Server Manual.pdf'
drwxrwxr-x 6 steam steam 4.0K Feb  7 00:56  valheim_server_Data
-rwxrwxr-x 1 steam steam 6.2K Feb  7 00:56  valheim_server.x86_64
```
Now all you need to do is backup and modify the file `start_server.sh` which contains:
```bash
export templdpath=$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=892970

# Tip: Make a local copy of this script to avoid it being overwritten by steam. 
# NOTE: Minimum password length is 5 characters & Password cant be in the server name.
# NOTE: You need to make sure the ports 2456-2458 is being forwarded to your server through your local router & firewall.
./valheim_server.x86_64 -name "My server" -port 2456 -world "Dedicated" -password "secret" -public 1 > /dev/null &

export LD_LIBRARY_PATH=$templdpath

echo "Server started"
echo ""
read -p "Press RETURN to stop server"
echo 1 > server_exit.drp

echo "Server exit signal set"
echo "You can now close this terminal"
```

Replace using a text editor(anything from `vim`, `nano` etc, you can even download the file and reupload it:

I used nano, to do this enter `nano start_server.sh` and use your arrow keys to navigate around to edit:

- `My server` with your server name;
- `Dedicated` with the name of your world file database (Optional - not required);
- `secret` with your password that needs to be at least 5 characters long. (If you want no password remove "secret" including the quotes)
Save the file (Ctrl + X, press Y to say yes, hit enter to save) and now you can start your server:
```bash
./start_server.sh
```

On the VPS the server will take about 10-15 minutes to start. Just keep waiting, I promise you it will start. 
- Let the server start up once before transferring your world files to the server.

Hit `RETURN`(also known as `Enter` key) if you want to stop the server.
This is fine to get started as quick as possible, but I like to build a service around this just in case my server crashes and I want better control than just using the same command over and over again, so if you are interested, you might want to check `Step 7`.

## Step 7 - Connecting to your server
- Method 1 - In Game:
You can search for the server in game on the `Join Game` tab, then check `Community` box and complete the search box with your server name.
This might not always work for others because your server might not be in their region, but don't fear because we have another way so you might want to read `Method 2`.

- Method 2 - Steam Server List:
For this method you will want to get your servers IP address. It is located in the email you got or in the service it self.

On your Steam Client, go to `View > Servers > FAVORITES` and click `ADD A SERVER` and add `<your_ip>:2457` then click `ADD THIS SERVER TO FAVORITES`, then it will appear in your list. Click connect and enter the password that you configured in the previous steps(`Step 6 - Step 7` the field `secret`), after that your game will start automatically then click `Start` in game, it will ask you for the password again and it should start your session.


# Updating
To update follow the next steps.

Running this command will take a while to download any updates for steamcmd, but it's necessary:
```
steamcmd
```
After the download is complete, you will be prompted to the steamcmd command line:
```
Steam>
```
Valheim will allow you to download the dedicated server as an anonymous steam user. We will use the following command:
```
Steam> login anonymous
```
Output:
```
Connecting anonymously to Steam Public...Logged in OK
Waiting for user info...OK
```
Now we want to update the server files and executables, so first of all we need the SteamAppId, in our case the ID is `896660` and you can check on https://steamdb.info/apps/ and search for "Valheim Dedicated Server".
The command to update is:
```
Steam> app_update 896660 validate
```
Once you see:
```
Steam>
```
Type `quit` and hit enter/return

Now you are done! Verify that your `start_server.sh` file still look right. If it doesn't you can update it or upload the backup you saved.

Now you can start your server by doing `./start_server.sh`

# Congratulations you are done!

Big shoutout to Skymoon for the help on this as well as [BrotherPatrix](https://github.com/BrotherPatrix) since I used his valheim server ubuntu tutorial as a template.



