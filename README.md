# Migrate /home folder to new partition on server .100
https://docs.ortoscale.com/pages/viewpage.action?pageId=220758274

Plan for migration:
1. make sure you have backups, and recovery plan (vasyl.h doesn't have)
1. First we will add new HDD of 360GB (currently hdd have 300GB and we need to increase it by 20%)
1. On .100 server that HDD will have a name sdc. We will create partition sdc1.
1. Make sure no body is logined exept `ssh root@.100`
1. Ensure that rsync is installed (ansible role: packages)
1. add authorized_key for root user on .100, so the AWX can password-less talk to .100 and original /home will not busy (ansible role: set-credentials) (and additionally, dont use password, see issue https://github.com/ansible/ansible/issues/56629)
1. for these steps responsible ansible role: create-new-lv
    * Next we will create physical volume (pvcreate) for /dev/sdc1.
    * After that we will create volume group (vgcreate) for /dev/sdc1 name it vg-data.
    * After that we will create Logical volume (lvcreate) for vg-data and name it lv-data.
    * Now we will have a LVM vg-data/lv-data.
1. Stop services in service_list var (before unmounting home all DA + related services needs to be stopped) (ansible role: stop-services)
1. Ensure nobody of regular user logged in system run `who`. If you see root you are fine
1. Umnount /home and mount with ro option - read only (ansible role: home-ro-mode)
1. migrate data using rsync (ansible role: data-migration)
1. mount new LV as /home, mount original home as /home_orig, update /etc/fstab (ansible role: final-mount)
1. Start services in service_list var 
1. Schedule quotas review run `echo "action=rewrite&value=quota" >> /usr/local/directadmin/data/task.queue` 
https://www.directadmin.com/features.php?id=529
1. DO NOT REBOOT



### Side effects:
1. ssh-copy-id works weird after migration, shouldn't ask password anymore. 192.168.15.127 after home migration

### Vars
da_service_list: ["da-popb4smtp","directadmin","dovecot","exim","httpd","mysqld","pure-ftpd", "veeamservice"]