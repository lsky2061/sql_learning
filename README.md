First, I had to reinstall mysql-server. Then I followed the instructions below

    sudo mysql -u root # I had to use "sudo" since it was a new installation
    
    mysql> USE mysql;
    mysql> CREATE USER 'YOUR_SYSTEM_USER'@'localhost' IDENTIFIED BY 'YOUR_PASSWD';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'YOUR_SYSTEM_USER'@'localhost';
    mysql> UPDATE user SET plugin='auth_socket' WHERE User='YOUR_SYSTEM_USER';
    mysql> FLUSH PRIVILEGES;
    mysql> exit;

    sudo service mysql restart

In my case 'YOUR_SYSTEM_USER' = luke

https://stackoverflow.com/questions/39281594/error-1698-28000-access-denied-for-user-rootlocalhost

Using the CDC WONDER tab file, I had to remove the "notes" at the end to prevent the error ERROR 1261 (01000): Row 2898 doesn't contain data for all columns

After several attempts and refinements, 

    CREATE TABLE wonder_2018_2022 ( notes VARCHAR(128) DEFAULT NULL, icd_10_113_cause VARCHAR(256) DEFAULT NULL, icd_10_113_code VARCHAR(16) DEFAULT NULL, age_char VARCHAR(16) DEFAULT NULL, age_num INTEGER DEFAULT NULL, year VARCHAR(16) DEFAULT NULL, year_num INTEGER DEFAULT NULL, deaths INTEGER DEFAULT NULL, population INTEGER DEFAULT NULL, crude_rate VARCHAR(16) DEFAULT NULL);

    LOAD DATA INFILE '/var/lib/mysql-files/Underlying Cause of Death, 2018-2022, Single Race_year_1_19_trunc.txt' INTO TABLE wonder_2018_2022 FIELDS TERMINATED BY '\
t' OPTIONALLY ENCLOSED BY '"' IGNORE 1 ROWS;
