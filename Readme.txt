               Raspberry Pi Python Motion Capture and goggle drive syncing using grive
               -----------------------------------------------------------------------
      grive (google drive) capable raspberry pi security camera using python motion detection

Recently I have been working on a grive capable security camera using two types of security camera cases.  
One is a small fake plastic security cam case from Amazon.  Model A or B fits inside with wifi only.

http://www.amazon.com/gp/product/B004D8NZ52/ref=oh_details_o01_s00_i00?ie=UTF8&psc=1

Here is a larger aluminum camera case that I have a model B installed in.  This one has room for a usb power supply etc..

http://www.amazon.com/Monoprice-108431-Outdoor-Camera-Switchable/dp/B007VDTTTM/ref=sr_1_72?ie=UTF8&qid=1393884556&sr=8-72&keywords=fake+security+camera

I may do a youtube video on How To setup these cases with the raspberry pi computer and camera module installed.

Raspberry pi forum information on lightweight python motion detection topic

http://www.raspberrypi.org/forums/viewtopic.php?f=43&t=45235&p=362504&sid=06ba253d0d74f92733bc6862a224aafe#p362504

After some work I now have the Raspberry Pi security camera's working efficiently from a software point of view.  
The current configuration uses a modified version of pimotion.py script to save files to a number sequence instead of a date-time sequence. 
I also added some code (not mine) to optionally put date/time information directly on the photo images.  
This is convenient to see the exact time stamp that the photo was taken. 
Using number sequencing allows limiting the number of files that need to get synchronized to my google drive. 
It was too much to manage all the dated files and cleanup in google drive.  
This method restricts the number of motion files that need to get updated via grive. Files are overwritten in Round robin fashion.
If you need more history you can write a routine to save google drive files from a synchronized PC folder to a dated archive folder
using a windows robocopy freefilesync or similar program through a batch file.  Synchronization uses a rpi compiled version of grive.  
This requires slightly modifying the source code to make it compatible with the RPI.
  
The tar file is a complete setup including a precompiled grive to reduce the effort required to get this working.  
To automate the security camera operation, I have setup pimotion.py to run from a /etc/init.d/pimotion.sh bash script
by copying skeleton file to pimotion.sh script (sample included).  Then modified to run your pimotion.py script on boot.
see later in post for more setup detail.

You must have a raspberry pi model A or B with the latest raspbian build and pi camera module installed and working.  
There are several tutorials showing how to do this so it is not covered here. 
This assumes you know how to cut and paste into nano or similar text editor on the pi using ssh. 
You also need an operational internet connection via wifi or wired connection.  
Wifi needs to be setup to work on boot with no desktop in order for the camera to sync unattended with your google drive.  
I have written the pimotion python script and bash sync scripts to make it somewhat independent of the folder names etc.  
This minimizes hard coding folder names in the scripts.  
If you run the script manually from the command line then settings and activity information will be displayed.
Just a little setup for pimotion.py  If you already have a pimotion.py then mv existing file to another file name

Download and Setup Instructions
-------------------------------
ssh into an internet connected raspberry pi and execute the following commands.  
Note change picam to a folder name of your choice if required.

cd ~
mkdir picam
cd ./picam
# Download pimotion.tar file from my github account
wget https://raw.github.com/pageauc/pi-motion-grive/master/pimotion.tar
# Extract tar files to current folder
tar -xvf pimotion.tar
# Install required grive libraries from the internet
sudo ./grive_setup.sh 
# This will setup google drive security connection see details below
# You will need to cut the rpi grive url displayed from above command
# into chrome browser on pc with a logged in google account then past the resultant
# security hash file back into grive via ssh session on rpi. 
sudo ./grive -a        
# Create a google_drive folder if it does not exist and start motion capture using pi camera module

sudo ./pimotion.py

ctrl-c to exit pimotion.py     

Setup grive security to your google account
-------------------------------------------
From ssh session on raspberry pi execute the following

sudo ./grive -a 

perform the following
Cut and paste the resulting grive url into chrome url box or other pc web browser that is logged into your google account
Cut and paste the resulting web browser security code back to raspberry pi grive session.
grive will indicate if the operation was successful

Once grive has been initialized successfully with the grive -a option then
copy the /home/pi/.grive and /home/pi/.grive_state files to the /home/pi/google_drive folder per above code. 
This will allow grive to be executed from the /home/pi folder so it does not have to be in the google_drive folder.

sudo cp ./.grive google_drive
sudo cp ./.grive_state google_drive
sudo ./sync.sh

Change pimotion.py settings as required
---------------------------------------
pimotion.py is the modified pimotion.py script based on previous modified brainflakes script.  
I modified the script to allow the use of a number sequence to restrict the total number of files
that need to get sync'd to my google drive using grive.  Currently set to 500 images 
Please note this includes PIL imageFont and imageDraw python modules to optionally put a date-time stamp on each photo.
See later for details on settin up grive security and login to your google account.


Setup init.d script to auto launch pimotion.py on boot-up of raspberry pi
-------------------------------------------------------------------------

cd /etc/init.d
sudo cp skeleton pimotion.sh
chmod +x pimotion.sh
sudo nano pimotion.sh   # change appropriate entries to point to your pimotion.py script
sudo update-rc.d pimotion.sh defaults
cd ~


Use nano to modify any pimotion.py script settings as required eg Threshold, Sensitivity, image prefix, numbering range, etc.  
nano ./pimotion.py
I have the pimotion.py camera image settings set to flipped due to the camera module mounting position in the fake security camera case.
You may also want to change sensitivity and/or threshold as needed as well as the filename prefix etc.  Ctrl-X to save and exit

Details if you wish to compile grive yourself (Optional)
--------------------------------------------------------
You will need to download the tar file and extract it on your raspberry pi or compile a modified version of  grive in order to sync files to your google drive
see instructions url link below.  Compiling takes a little while and you must edit the specified /home/pi/grive/libgrive/src/drive/State.cc file per the web link below.  If you have problems read the posts.  When you initialize grive with google I opened an ssh session to the raspberry pi on my windows 7 PC and then cut and pasted the grive -a url to chrome browser while logged into google.  This takes you to a screen that returns a very long security code.  I then cut and pasted this into the RPI session and everything worked just fine.  I did not login to google on the pi.  I only needed the PC to be logged in and paste the authentication code back to the pi from the PC.  I don't think you need a logged in google account on the pi as the post mentions.  At any rate it worked for me and I had to try several times since I was trying to avoid having grive executable in the google_drive folder.  By using the -p option and copying the grive hidden config files to the rpi google_drive folder I managed to get everything to work.   
http://raspberrywebserver.com/serveradmin/back-up-your-pi-to-your-google-drive.html
or this link might be even better
http://www.pihomeserver.fr/en/2013/08/15/raspberry-pi-home-server-synchroniser-le-raspberry-pi-avec-votre-google-drive/

Once compile is successful copy the grive executable to the folder that pimotion.py and sync.sh are located
modify this to suit your folder structure.

To test you can launch pimotion.py from a ssh terminal session

cd ~
sudo ./pimotion.py


From a second ssh terminal run sync.sh (make sure that motion was detected and files are in the google_drive folder to sync).  
You should see a /home/pi/sync.lock file.  This is created by pimotion.py when motion photos were created.


cd ~
sudo ./sync.sh


You should see files being synchronized in both directions.  This is normal.  
There are google drive apps for Apple, Android, Mac and PC.  
This will allow you to access the raspberry pi motion capture security camera files on other computers or handheld devices.
Assuming you have wifi/wired network to start on boot headless and the crontab and init.d setup then the camera will work
immediately on booting (unattended).  I also setup a cronjob to reboot the rpi once a day but this may not be necessary for you.  
I did it since I intend to leave the rpi security camera run remotely and this gives a chance for system to recover should there be a glitch.
Also if you have problems check permissions.  The init.d will run as root so the files in the google_drive folder will be owned by root.  
Just check to make sure you run with sudo ./pimotion.sh and sudo ./sync.sh if you are in terminal sessions.
Once you know sync.sh is working OK you can automate the sync by running it in as a cronjob.

sudo crontab -e

Paste the following line into the crontab file nano editor and modify folder name and frequency if required.
currently executes every minute. 

*/1 * * * * /home/pi/picam/sync.sh >/dev/nul

ctrl-x to exit nano and save crontab file
This cron job will run once a minute.  You can change to suit your needs.  
If grive is already running or there are no files to process then the script simply exits. 
Also if grive has been running for more than 5 minutes it is killed.  
This can be changed in the script if you wish.

Good Luck
Claude Pageau 
