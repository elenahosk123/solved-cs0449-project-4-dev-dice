Download Link: https://assignmentchef.com/product/solved-cs0449-project-4-dev-dice
<br>



<strong>Project Description</strong>

Standard UNIX and Linux systems come with a few special files like /dev/zero, which returns nothing but zeros when it is read, and /dev/random, which returns random bytes. In this project, you will write a device driver to create a new device, /dev/dice which returns randomly selected rolls of a 6-sided die.

<strong>How It Will Work</strong>

For this project, we will need to create three parts: the device driver, the special file /dev/dice, and a test program to convince ourselves it works

The test program will be an implementation of the game of craps.

<strong>Driver Implementation</strong>

Our device driver will be a character device that will implement a read function (which is the implementation of the read() syscall) and returns appropriate die rolls (a single die roll is a 1 byte value from 0 to 5 and we must support read()ing an arbitrary count of these). As we discussed in class, the kernel does not have the full C Standard library available to it and so we need to get use a different function to get random numbers. By including &lt;linux/random.h&gt; we can use the function get_random_bytes(), which you can turn into a helper function to get a single byte:

unsigned char get_random_byte(int max) {

unsigned char c;

get_random_bytes(&amp;c, 1);

return c%max;

}




<strong>The Game of Craps (30 points)</strong>

Using your driver, you will be implementing the game of Craps. For those unfamiliar with the rules, Craps is played with two dice which are rolled together. The sum of these two dice is computed and if the sum is:

<ul>

 <li>2, 3, or 12 – the player immediately loses.</li>

 <li>7 or 11 – the player immediately wins.</li>

 <li>Any other number is referred to as the “point.” The goal then becomes to keep rolling the dice until:</li>

</ul>

o   The player rolls a 7 – the player loses

o   The player rolls the “point” – the player wins.

Your job is to make a simple craps game.  In this game the player will enter his or her name, the program will display a welcome message, and ask if the player would like to play or quit.  The program will then roll two dice, and display them and their total.  If the player has won, display a message congratulating them. If the player has lost, report it to them. Otherwise, store this first roll as the “point” roll and keep rolling until the player wins or loses.  When the game ends, ask if the player would like to play again. Example:

Welcome to Jon’s Casino!

Please enter your name: <strong>Jonathan</strong>

Jonathan, would you like to Play or Quit? <strong>play</strong>




You have rolled 5 + 2 = 7

You Win!




Would you like to play again? <strong>no</strong>




Goodbye, Jonathan!

<strong>Hints</strong>

<ul>

 <li>Remember that rolling two 6-sided dice is different than rolling one 12-sided die.</li>

 <li>Use string input, do not make an integer menu or use single characters.</li>

</ul>

<ul>

 <li>Gets each die to display by reading bytes from the /dev/dice file.</li>

</ul>

<ul>

 <li>You may write the craps program using stdlib’s rand() for testing, but make sure you eventually replace it with your read()s of /dev/dice</li>

</ul>

<strong>Installation and Example </strong>

<ol>

 <li>On thoth.cs.pitt.edu, login and cd to your /u/SysLab/USERNAME directory.</li>

 <li>tar xvfz ../shared/hello_dev.tar.gz</li>

 <li>cd hello_dev</li>

 <li>Open the Makefile with the editor of your choice. (E.g., nano Makefile)</li>

 <li>We need to setup the path to compile against the proper version of the kernel. To do this, change the line:</li>

</ol>

KDIR  := /lib/modules/$(shell uname -r)/build

to

KDIR  := /u/SysLab/shared/linux-2.6.23.1

<ol>

 <li>Build the kernel object. The ARCH=i386 is important because we are building a 32-bit kernel on a 64-bit machine.</li>

</ol>

make ARCH=i386

<ol>

 <li>Download and launch QEMU. For Windows users, you can just double click the qemu-win.bat that is supplied in the zipfile available on my website. Mac users should download and install Q.app and use the tty.qcow2 disk image provided in the main zipfile. Linux users are left to install qemu as appropriate and also use the tty.qcow2 disk image provided in the main zipfile.</li>

 <li>When Linux boots under QEMU login using the root/root account (username/password).</li>

 <li>We now need to download the kernel module you just built into the kernel using scp:</li>

</ol>

scp <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="42171107100c030f0702362a2d362a6c21316c322b36366c272637">[email protected]</a>:/u/SysLab/USERNAME/hello_dev/hello_dev.ko .

<ol>

 <li>Load the driver using insmod:</li>

</ol>

insmod hello_dev.ko

<ol>

 <li>We now need to make the device file in /dev. First, we need to find the MAJOR and MINOR numbers that identify the new device:</li>

</ol>

cd /sys/class/misc/hello

cat dev

<ol>

 <li>The output should be a number like 10:63. The 10 is the MAJOR and the 63 is the MINOR.</li>

 <li>We use mknod to make a special file. The name will be hello and it is a character device. The 10 and the 63 correspond to the MAJOR and MINOR numbers we discovered above (if different, use the ones you saw.)</li>

</ol>

cd /dev/

mknod hello c 10 63

<ol>

 <li>We can now read the data out of /dev/hello with a simple call to cat:</li>

</ol>

cat /dev/hello

<ol>

 <li>You should see “Hello, world!” which came from the driver. We can clean up by removing the device and unloading the module:</li>

</ol>

rm /dev/hello

rmmod hello_dev.ko

<strong>What to Do Next</strong>

The code for the example we went through comes from http://www.linuxdevcenter.com/pub/a/linux/2007/07/05/devhelloworld-a-simple-introduction-to-device-drivers-under-linux.html?page=2 which I’ve mirrored at: <a href="https://people.cs.pitt.edu/~jmisurda/teaching/cs449/valerie-henson-device-drivers-hello.pdf">http://people.cs.pitt.edu/~jmisurda/teaching/cs449/valerie-henson-device-drivers-hello.pdf</a>

Read that while going through the source to get an idea of what the Module is doing. Start with the third section entitled “Hello, World! Using /dev/hello_world” and read up until the author starts describing udev rules; we are not using udev under QEMU.

When you have an idea of what is going on, make a new directory under your /u/SysLab/USERNAME directory called dice_driver:

mkdir dice_driver

Copy and rename the hello_dev.c from the example, and copy over the Makefile. Edit the Makefile to build your new file. Change all the references of “hello” to “dice_driver”.

<strong>Building the Driver</strong>

To build any changes you have made, on thoth in your dice_driver directory, simply:

make ARCH=i386

If you want to force a rebuild of everything you may want to remove the object files first:

rm *.o

<strong>Copying the Files to QEMU</strong>

From QEMU, you will need to download the driver that you just built.  Use scp to download the driver to a home directory (/root/ if root):

scp <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="471214021509060a0207332f28332f69243469372e333369222332">[email protected]</a>:/u/SysLab/USERNAME/dice_driver/dice_driver.ko .

<strong>Loading the Driver into the Kernel in QEMU</strong>

As root (either by logging in or via su):

insmod dice_driver.ko

<strong> </strong>

<strong>Making the /dev/</strong><strong>dice</strong><strong> Device</strong>

Like in the example, we’ll need to determine the MAJOR and MINOR numbers for this particular driver.

cd /sys/class/misc/dice_driver

cat dev

and use those numbers to make the /dev/dice file:

cd /devmknod dice c MAJOR MINOR

<strong>Unloading the Driver from the Kernel in QEMU</strong>

As root (either by logging in or via su):

rmmod dice_driver.ko

Then you can remove the /dev/dice file:

rm /dev/dice

<strong>Implementing and Building the </strong><strong>craps</strong><strong> Program</strong>

Since thoth is a 64-bit machine and QEMU emulates a 32-bit machine, we should build with the -m32 flag:

gcc –m32 –o craps craps.c -static

<strong>Running </strong><strong>craps</strong>

We cannot run our craps program on thoth.cs.pitt.edu because its kernel does not have the device driver loaded. However, we can test the program under QEMU once we have installed the new driver. We first need to download craps using scp as we did for the driver. However, we can just run it from our home directory without any installation necessary.

<strong>File Backups</strong>

One of the major contributions the university provides for the AFS filesystem is nightly backups. However, the /u/SysLab/ partition is <strong>not</strong> part of AFS space. Thus, any files you modify under your personal directory in /u/SysLab/ are not backed up. If there is a catastrophic disk failure, all of your work will be irrecoverably lost. As such, it is my recommendation that you:

<strong>Backup all the files you change under </strong><strong>/u/SysLab</strong><strong> to your </strong><strong>~/private/</strong><strong> directory frequently!</strong>

Loss of work not backed up is not grounds for an extension. You have been warned.

<strong>Hints and Notes</strong>

<ul>

 <li>printk() is the version of printf() you can use for debugging messages from the kernel.</li>

 <li>In the driver, you can use some standard C functions, but not all. They must be part of the kernel to work.</li>

 <li>In the craps program, you may use any of the C standard library functions.</li>

 <li>As root, typing poweroff in QEMU will shut it down cleanly.</li>

 <li>If the module crashes, it may become impossible to delete the file you created with mkdev in /dev. If that happens, just grab a new disk image and start over. It’s why we’re developing in a virtual machine as opposed to a real one.</li>

</ul>