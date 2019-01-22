# scriptor
Backup, extract, and prepare script for database.


# How to use

## Requirements
- MariaDB
- Mariabackup
- Percona Xtrabackup
    > We need this for extracting the encrypted database backup.
- Mutt
    > We need this for email notification.
- A dedicated system and mysql user for performing the backup.

## Steps
**For this example, we will be using *backup* as our backup user**
1. Install the first 4 requirements above.
2. Create a MYSQL user dedicated for performing the backup.
```sh
mysql> CREATE USER 'backup'@'localhost' IDENTIFIED BY 'password'; 
```
3. Grant appropriate privileges.
```sh
mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT, CREATE TABLESPACE, PROCESS, SUPER, CREATE, INSERT, SELECT ON *.* TO 'backup'@'localhost';
```
4. Create a system user and group named **backup**.\
**Note:** If you are using Ubuntu distro, you can skip this step. On Ubuntu, a **backup** user and corresponding **backup** group is already available.
5. Configure the system backup user and assigning permissions.
    - Type the following commands to add the backup user to the **mysql** group and your **sudo** user to the **backup** group:
    ```sh
    sudo usermod -aG mysql backup
    sudo usermod -aG backup ${USER}
    ```
6. Creating backup assets.
    - Create a MySQL configuration file with the backup parameters
    ```sh
    sudo nano /etc/mysql/backup.cnf
    ```
    - Inside, start a **[client]** section and set the MySQL backup user and password user you defined within MySQL:
        >[client]\
        >user=backup\
        >password=password

    - Give ownership of the file to the **backup** user and then restrict the permissions so that no other users can access the file:
    ```sh
    sudo chown backup /etc/mysql/backup.cnf
    sudo chmod 600 /etc/mysql/backup.cnf
    ```
7. Create a backup root directory
```sh
sudo mkdir -p /backups/mysql
```
8. Assign ownership of the **/backups/mysql** directory to the **backup** user and group ownership to the **mysql** group:
```sh
sudo chown backup:mysql /backups/mysql
```
9. Create an encryption key to secure the backup files
```sh
printf '%s' "$(openssl rand -base64 24)" | sudo tee /backups/mysql/encryption_key && echo
```
10. Git clone this repository. It's free.
```sh
git clone https://github.com/rppf/scriptor.git
```
11. Move the scripts to **/usr/local/bin** and make them executable.

12. Then try to run the backup script
```sh
sudo -u backup /usr/local/bin/backup-mysql.sh
```

**If theres's an error, the log is located in your backup root directory.**