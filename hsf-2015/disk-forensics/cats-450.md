HSF Cats - Disk Forensics (450 pts)
==================================
Writeup by team Arxenix from NHSS

First Steps
-----------
After first extracting the cats zip file, we see several items, including a **vmware.log** file and a **[.vmdk](https://en.wikipedia.org/wiki/VMDK)** file. This instantly indicates that a [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine) hard drive file is present, and should be opened up with vmware or virtualbox. This writeup will show how to use it with [Oracle VM VirtualBox](https://www.virtualbox.org/).

Loading the Virtual Machine
---------------------------
In order to load a virtual machine using this .vmdk file, there's a few steps that need to be done:

 1. Download and install Oracle VM VirtualBox for your operating system from [this link](https://www.virtualbox.org/wiki/Downloads)
 2. Open it up
 3. Click the "New" button in the top left
 4. Name it whatever you want, but make sure the OS is ubuntu

 ![Step 4](http://i.imgur.com/ssIc4P6.png?1)
 5. Leave memory size at default and click next
 6. Choose "Use an Existing Hard Disk" option and click the folder icon

 ![Step 6](http://i.imgur.com/pMaTOVU.png?1)
 7. Select the Cats.vmdk file

 ![Step 7](http://i.imgur.com/3VejKga.png?1)
 8. Click "Create"

The Virtual machine will be created and will be using the Cats.vmdk hard disk. Boot up the VM by double-clicking on it.

Exploring the Environment for Clues
-----------------------------------
After the VM boots up, we log in by entering the provided password: `F3lyn34LifE!`

Our first attempt was to view the ~/.bash_history file for command logs, but (as suspected) it had been cleared.

 1. We then opened up the terminal application and ran the command `sudo du -s *` in the home directory to find the total sizes of each directory in hopes of finding one that might have useful files inside.

 ![Step 1](http://i.imgur.com/6bSuraI.png?1)
 2. The `catz` file in the home directory is large, and seemed like it might be a mountable filesystem (because this is a file forensics challenge, and it returned "data" after running `file catz`). We tried mounting it in several ways with the `mount` command, but nothing worked and the filesystem type couldn't be identified.
 3. The next largest directory was `Documents`, so we navigated into that and listed the files.

 ![Step 3](http://i.imgur.com/Evc0BWC.png?1)
 4. The directory contained another Documents directory and a Pictures directory, which is a little strange. We navigated into the `Pictures` directory next and listed the files.

 ![Step 4](http://i.imgur.com/QlkWf72.png?1)
 5. It seemed to be files to install **[veracrypt](https://veracrypt.codeplex.com/)**, a software used to encrypt filesystems. Maybe this is what was used on the `catz` file!
 6. We then navigated to the second `Documents` directory, to find files that seemed to be for installation of some sort of tool.

 ![Step 6](http://i.imgur.com/lcKElxS.png?1)
 7. We opened up the README file, and found that it was for installation of a keylogger!

 ![Step 7](http://i.imgur.com/MOyBsLF.png?1)
 8. Scrolling further down the README file, we found the file in which keystrokes were logged to: `/var/log/logkeys.log`

 ![Step 8](http://i.imgur.com/tVRmcSE.png?1)
 9. We open up the file `/var/log/logkeys.log` using `sudo` (the sudo password is the same as the account password), and find the keystrokes that the last user made.

 ![Step 9](http://i.imgur.com/mtDx5IM.png?1)
 10. Since we hadn't used veracrypt before and weren't 100% sure that Me0wL3tMeInPl$ was the password (even though it looks like it is), we ran the same commands that the last user did, starting with `veracrypt -t -c`, and found that they did indeed enter `Me0wL3tMeInPl$` when prompted for the password.
 11. Our initial suspicions seem to be confirmed, and we must now decrypt the `catz` file with veracrypt! Looking through the veracrypt documention with `man veracrypt`, we find that we can decrypt and mount it to a directory using `veracrypt -p Me0wL3tMeInPl$ catz`.

 ![Step 10](http://i.imgur.com/Yr3spGf.png?1)

Looking Into the Mounted File
-----------------------------

 1. Upon navigation into the mount directory, we see that there are many PDF's inside of the CATZZ directory. We opened all of these up and looked through them, but didn't find anything at first glance.

 ![Step 1](http://i.imgur.com/ojpEltp.png)
 2. We then tried to do a direct `grep -i flag *` in the directory in case the flag was inside the raw data of any of the files. However, this didn't work either.
 3. Since some of the PDF's had a substantial amount of text, and we knew that it is possible for text to be hidden "behind images" or be "invisible" inside PDFs, we installed a useful command-line utility known as [pdfgrep](https://pdfgrep.org/), and ran the command `pdfgrep -i flag *`

 ![Step 3](http://i.imgur.com/suEIjNW.png?1)
 4. Success! the grooming.pdf seems to have the text "Flag" inside of it, so we run another pdfgrep to find out what it is: `pdfgrep -i flag grooming.pdf`

 ![Step 4](http://i.imgur.com/MovFj3g.png?1)
 5. We have now found the flag and can submit it for a nice 450 points. Pretty easy overall!
