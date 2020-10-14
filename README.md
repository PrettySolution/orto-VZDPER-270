# Migrate /home folder to new partition on server .100
https://docs.ortoscale.com/pages/viewpage.action?pageId=220758274

Plan for migration:

1. First we will add new HDD of 360GB (currently hdd have 300GB and we need to increase it by 20%)
1. On .100 server that HDD will have a name sdc. We will create partition sdc1.
1. Ensure that rsync is installed (ansible role: packages)
1. add authorized_key for root user on .100, so the AWX can password-less talk to .100 and original /home will not busy (ansible role: set-credentials) (and additionally, dont use password, see issue https://github.com/ansible/ansible/issues/56629)
1. for these steps responsible ansible role: Create new LV
    * Next we will create physical volume (pvcreate) for /dev/sdc1.
    * After that we will create volume group (vgcreate) for /dev/sdc1 name it vg-data.
    * After that we will create Logical volume (lvcreate) for vg-data and name it lv-data.
    * Now we will have a LVM vg-data/lv-data.


1. Next part is to pay attention of data inside /home folder. 
1. First we must unmount /home directory.
1. After that we will rename it to /home-old (this is doing for security reason, if something do not work we can back our original /home directory).
1. Next we will create a new /home directory (now we will have /home directory newly created and empty, and /home-old directory which contains all data)-
1. Next we will mount our newly created /home directory to the LVM vg-data/lv-data.
1. Next we will with rsync copy all data from /home-old to /home.
1. After copying finish we will umoun /home direcotry, and add it in /etc/fstab.
1. Reboot server.
1. And finally we will have /home directory with all data on new LVM 360GB partition, and also we will have /home-old folder (it can be used as temporarly backup for checking if everything working fine, and than delete it after confirmation).