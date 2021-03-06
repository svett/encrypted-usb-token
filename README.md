# Building an Encrypted USB Drive for your SSH Keys

You can read the original article from [Tammer Saleh blog](http://tammersaleh.com/posts/building-an-encrypted-usb-drive-for-your-ssh-keys-in-os-x/).

Working on a Platform like Cloud Foundry, which is relied upon by a growing community of "serious" companies requires us to take security seriously as well.
Security is something you know, something you have, and something you are. The commonly agreed upon tenants of strong security is that it requires a combination of "something you know, something you have, and something you are." Two factor authentication includes both of those - usually something you know and something you have.
Here's how we've implemented two factor authentication across the board for our SSH keys using USB keychain drives. This strengthens our access to Github repositories and the numerous deployments we manage.
Follow these instructions to increase your security at home and work as well.

## Format the Drive

Erase
We prefer the Kingston DataTraveler drive due to its size and cost. Once you've found a USB keychain drive to your liking, you'll want to reformat it using OS X's built-in encrypted filesystem.
Plug your drive into your computer and open Disk Utility. Select the disk (not the volume) on the left and navigate to the "Erase" tab. You'll want to name the volume something simple (such as "keys") to make it easier to access on the command line. And, of course, you'll want to format it using "Mac OS Extended (Case-sensitive, Journaled, Encrypted)."
Now, you'll be prompted for your decryption password whenever you insert the drive. Be sure not to save the password into the OS X Keychain.

## Add your SSH Keys

If you don't already have SSH keys, then you'll want to generate a new set. In fact, it's probably a good idea to use this as a chance to create a fresh set either way, just in case yours have been compromised.

You create a new SSH key pair by running `ssh-keygen`:

```
$ ssh-keygen -f /Volumes/keys/id_rsa -C "Tammer Saleh"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is:
xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx Tammer Saleh
The key's randomart image is:
+--[ RSA 2048]----+
|             x   |
|            x  x |
|      x x xxxxxxx|
|     x x x x xxxx|
|      x x x   x  |
|     x x x     x |
|    x x x x      |
|     x   x       |
|                 |
+-----------------+
```

## Script to Load Keys and Eject

At this point, you could use the drive by manually adding the keys to your running agent and ejecting the drive. But that's a lot of typing and feels fairly error prone. Instead, let's script it.

Create the following script on your drive, and name it load:

```
#!/usr/bin/env bash

HOURS=$1
DIR=/Volumes/keys
KEY=$DIR/id_rsa

if [ -z $HOURS ]; then
  HOURS=1
fi

/usr/bin/ssh-add -D
/usr/bin/ssh-add -t ${HOURS}H $KEY
/usr/sbin/diskutil umount force $DIR
```

You will need to mark the script as executable:

`chmod +x /Volumes/keys/load`

Now, you can simply run `/Volumes/keys/load` to load your keys and eject the drive automatically. This makes for a very quick workflow.

```
$ /Volumes/keys/load
All identities removed.
Enter passphrase for /Volumes/keys/id_rsa: <your_password>
Identity added: /Volumes/keys/id_rsa (/Volumes/keys/id_rsa)
Lifetime set to 3600 seconds
Volume keys on disk3 force-unmounted
```

## Make a Backup Image

### Backup Image
Finally, you can make an encrypted backup image in case our USB drive shits the bed.
Again, open Disk Utility. This time, we select the "New Image" icon in the tool bar. Save the image as keys_backup, and configure it to be quite small, encrypted, and using a sparse image to save space.
Once that's done, mount both drives and copy everything from your USB drive to the backup.
Ideally, you'll store your backup somewhere super secure. Another option is to simply create two USB drives, and store one in a locked box. My wife and I have a locked fireproof box that we use for all of our personal documents (passports, etc), which is an ideal location.

## Weaknesses and Future Improvements

While this is infinitely better than leaving your ssh key unprotected on your computer, there are some weaknesses and potential future improvements.
The major weakness is that we're trusting that the host machine hasn't been tampered with. If it has, then we're handing our private keys over to it. Again, this risk exists either way, and isn't made worse through this technique.
The forced eject is blatant cargo culting. When tried without forcing, the drive reports being in use, but lsof shows no processes using it. This is the case even when the script is run from outside the drive, so I'm at a loss.
Ideally, we could store our entire `~/.ssh` directory on the keychain. This requires a symlink from `~/.ssh` to `/Volumes/keys/.ssh`, and has a number of other complications around permissions. We haven't investigated furthur.

