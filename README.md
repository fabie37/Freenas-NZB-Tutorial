# Freenas-NZB-Tutorial
Getting Radarr and SABnzbd to actually work on freenas using the default plugins.
**Version:** FreeNAS-11.3-U3.2

## Freenas, Jails and... Permissions.

The #1 reason it took me hours to understand what was going on in freenas was that permissions are pretty important. Like <b>REALLY </b> important.  So to give you the basic structure of it before I forget it all, let's start with how permissions are handled with jails. 

This really really frustrated me so if you need help, please give me a message. 

In the main web gui, under accounts you'll find options for both users and groups. 
There's a simple relationship between them, a group can have many users and a user can have many groups. Or in sql, many-to-many. 

I'll create a group called MediaServer for my MediaServer dataset. This is pretty self explanitory so, go into <b><i>accounts -> groups : add</i></b> and then just name it whatever you like. 

Now in freenas once you have your pool set up, and your simple dataset ( typically called tank or something like that, I'l call mine MediaServer) you need to set a group who can access that dataset. Simply follow these steps -
![Dataset Permissions  1](https://ibb.co/wB9nPBn)
![Dataset Permissions 2](https://ibb.co/2s7c8tR)
And click save at the bottom. 

Great, thats that sorted. Now, in Freenas you have jails, whenever you install a plugin.. you have a jail, whenever you install any third party app... you have a jail. Fine so what are they? They are like mini containers which do one ( or many) specific thing/s which excludes itself from your main data you're storing. The thing is that the users and groups you set in the main web gui translate into these self contained containers. So basically if you create a user in the freenas gui, the <b> id </b> not the username gets passed down to the jails. This means that a user with say the username monkeyman37 with uid of 233 is treated as the same user as cinderellaman with uid of 233. 

So at this point, go and install both radarr and SABnzbd from the plugin store. 
Let's mess with SAB first shall we? 
Right. 
Go to Jails -> sabnzbd (or whatever you named it) -> STOP -> SHELL

Now `GROUP` is just the name of your group you made before, mine is MediaServer. 
Whereas `ID` is the id of the group, check this in the accounts -> groups and it should be the first thing you see.


    cd ..
    pw groupadd GROUP -g ID
    mkdir /mnt/Downloads/
    chown -R _sabnzbd /mnt/ /mnt/Downloads
    chgrp GROUP /mnt/
    chgrp GROUP /mnt/Downloads
    pw usermod _sabnzbd -G GROUP
    
    
Create, now here's the fun part, in your root directory type the command `ls -ln` and on the `mnt` folder you should see your group ID and just left of the is the jail user id, ie the user _sabnzbd. Now, copy that number, you're gonna need it. Go to accounts -> users -> add, you can literally call this anything, I called it NZB, but under User ID* tab, paste in the number you recorded earlier on, in my case it was 350. <B> MAKE SURE TO GIVE WRITE PERMISSIONS </B> by clicking all the boxes. 

That should be all the command line stuff sorted for SAB.

Now we are pretty much gonna do the same for Radarr. So go to Jail -> radarr -> STOP -> SHELL and type this in.

    cd ..
    pw groupadd GROUP -g ID
    mkdir /mnt/Downloads/
    chgrp GROUP /mnt/
    chgrp GROUP /mnt/Downloads
    pw usermod radarr -G GROUP

And run the same code on the root directory `ls -ln` get that number left of the group id on the mnt folder, make a user with that id and...<B> MAKE SURE TO GIVE WRITE PERMISSIONS </B>

Cool cool...

So that's all the permissions done. Thank god.

Now for Mount Points.  Go to jails and make sure radarr and sab are not running. Click on the drop down for radarr and select Mount points. Go to actions -> add and use this address (assuming you called your jails sabnzbd and radarr) 

    Source: /mnt/HomeNas/iocage/jails/sabnzbd/root/mnt/Downloads
    Destination: /mnt/HomeNas/iocage/jails/radarr/root/mnt/Downloads

Now what this will do is basically allow the two jails to have access to the same folder with the same name... this is important. Then go to actions -> add again and add this 

    Source:  /mnt/HomeNas/MediaServer/Movies	// Your movie dataset
    Destination: /mnt/HomeNas/iocage/jails/radarr/root/media
    
So when you go to add movies on radarr, the path you will want to save them to is /media/ as that folder will radarr access to your dataset. 


Right, so start of with setting up SABnzbd. Just enter your host details, if you need recommendations..  https://www.reddit.com/r/usenet/wiki/providers here's a reddit list. Go Wild. 

So once you're up and running click on the cog on the top right of the page, go to folders and change <B><i> Completed Download Folder </i></B> from Dowloads/Complete to /mnt/Downloads. Then set <b> <i>Permissions for completed downloads </i></b> to 777. Save changes and we're done here.  Go to the general tab scroll down to the API key and copy that. We'll need it for later. 

Moving on to raderr, we need to set up our indexer and download client. Go to settings -> indexers and literally use any indexer you can find, I use NZBGeek, make sure if you are using them to subscribe to get your API key. Just click on the plus icon, select Newznab, called it a name, add your indexers URL and API key and that's it.

Now on the download client, click the plus icon, select SABnzbd, give it a name. Enter your api key we copied earlier. And for the IP/host, you'll need to go back to plugins on your freenas gui, go to plugins, drop down the radarr plugin and select the IPv4 address from there, excluding the subnet ie the /. Put that in as your host and give it a test. Save it. And you're pretty much done. 

I hope you found this useful to some extent and if you have any problem do give me a message and happy freenasing! 


