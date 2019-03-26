First, find all mount points to see whether your USB stick has already been mounted.
```
df -h
```
If not, then check where your drive is located:
```
sudo lsblk -m
```
or:
```
sudo lsblk -f
```
If your stick is not mounted, we need to proceed to mount it ourselves. Make a mountpoint and mount the USB drive, where xx is the drive in question.
```
sudo mkdir /mnt/sdxx
sudo mount /dev/sdxx /mnt/sdxx
``````
Create backup script. In this case, we are going to backup all mysql databases to a USB stick.
```
touch /usr/local/sbin/mysqlbackup.sh
cd /usr/local/sbin/
chmod 755 mysqlbackup.sh
nano mysqlbackup.sh
```
Paste the following code. Don't forget to edit the username, password for your mysql database and the USB drive mount point. 
For more detailed info about the script, visit: tinyurl.com/y8y3jqnr
```
#!/bin/bash
# Change this
MYSQL_USER="root"
MYSQL_PASSWORD="password"
BACKUPDIRECTORY="/mnt/sdxx"

# No need to change below this line.
DATE=$(date +"%Y%m%d")
MYSQL=/usr/bin/mysql
MYSQLDUMP=/usr/bin/mysqldump
SKIPDB="Database|information_schema|performance_schema|mysql"
RETENTION=14

mkdir -p $BACKUPDIRECTORY/$DATE

databases=`$MYSQL -u$MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW DATABASES;" | grep -Ev "($SKIPDB)"`
for db in $databases; do
echo $db
$MYSQLDUMP --force --opt --user=$MYSQL_USER -p$MYSQL_PASSWORD --skip-lock-tables --events --databases $db | gzip >  "$BACKUPDIRECTORY/$DATE/$db.sql.gz"
done

find $BACKUPDIRECTORY/* -mtime +$RETENTION -delete
```

Create a crontab.
```
nano /etc/crontab
```
Add this new crontab to it:
```
11 1 * * * root /usr/local/sbin/mysqlbackup.sh
```
It will run every night at 1:11 AM. 

Restart cron:
```
service cron restart
```
To test the script manually, do this and check your USB stick for any sign (hopefully it has a blinking LED):
```
cd /usr/local/sbin/
mysqlbackup.sh
```
Done. Enjoy your automatic backups!
