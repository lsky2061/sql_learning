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

#Data Celaning

Using the CDC WONDER tab file, I had to remove the "notes" at the end to prevent the error ERROR 1261 (01000): Row 2898 doesn't contain data for all columns

For the WISQARS file, it contains serveral entries with a double asterisk after any number between 10 and 20. Several entries are just "--" in indicate a "indicates value (between one to nine deaths)" An example row:

    "Cut/Pierce","Homicide","18","2020","19**","4,410,589","0.43**","--","893"

This means that 19 18-year-olds were murdered by cutting or piercing in 2020 in the US. While the rest of this document is focused on data analysis, SQL methods, and my goal of answering a specific question, I want to make sure I remember something. While evaluating these data to find the truth and make our world a safer and better place, I think we always need to remember that each of these numbers represents a person's life cut short, a tragedy, and a grieving family. 

#Data Loading

First, I created a database to hold the two tables...

...more to come here.

After several attempts and refinements, 

    CREATE TABLE wonder_2018_2022 ( notes VARCHAR(128) DEFAULT NULL, icd_10_113_cause VARCHAR(256) DEFAULT NULL, icd_10_113_code VARCHAR(16) DEFAULT NULL, age_char VARCHAR(16) DEFAULT NULL, age_num INTEGER DEFAULT NULL, year VARCHAR(16) DEFAULT NULL, year_num INTEGER DEFAULT NULL, deaths INTEGER DEFAULT NULL, population INTEGER DEFAULT NULL, crude_rate VARCHAR(16) DEFAULT NULL);

    LOAD DATA INFILE '/var/lib/mysql-files/Underlying Cause of Death, 2018-2022, Single Race_year_1_19_trunc.txt' INTO TABLE wonder_2018_2022 FIELDS TERMINATED BY '\t' OPTIONALLY ENCLOSED BY '"' IGNORE 1 ROWS;

Websites that were very helpful in the above commands:

https://blog.skyvia.com/how-to-import-csv-file-into-mysql-table-in-4-different-ways/#Importing-CSV-into-MySQL-Automatically

https://stackoverflow.com/questions/32756710/count-specific-character-in-a-cell-in-openoffice-calc

https://stackoverflow.com/questions/29981417/mysql-load-data-infile-from-csv-fields-optionally-enclosed-by-quotes-numbers-h

https://stackoverflow.com/questions/32737478/how-should-i-resolve-secure-file-priv-in-mysql

In WISQARS, "Motor Vehicle, Overall" includes "Motor vehicle, traffic", "Pedal cyclist, other", "Pedestrian, other" and "Transport, other land".


