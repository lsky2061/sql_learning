# The Question

# Getting MySQL working

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

# Background and Data

# Data Cleaning

Using the CDC WONDER tab file, I had to remove the "notes" at the end to prevent the error ERROR 1261 (01000): Row 2898 doesn't contain data for all columns

For the WISQARS file, it contains serveral entries with a double asterisk after any number between 10 and 20. Several entries are just "--" in indicate a "indicates value (between one to nine deaths)" An example row:

    "Cut/Pierce","Homicide","18","2020","19**","4,410,589","0.43**","--","893"

This means that 19 18-year-olds were murdered by cutting or piercing in 2020 in the US. While the rest of this document is focused on data analysis, SQL methods, and my goal of answering a specific question, I want to make sure I remember something. While evaluating these data to find the truth and make our world a safer and better place, I think we always need to remember that each of these numbers represents a person's life cut short, a tragedy, and a grieving family. 

I opened the file in emacs and removed all of the asterisks. 

# Data Loading

First, I created a database to hold the two tables...

     create database cdc_2018;

## Loading WONDER Database
    
After several attempts and refinements, 

    CREATE TABLE wonder_2018_2022 ( notes VARCHAR(128) DEFAULT NULL, icd_10_113_cause VARCHAR(256) DEFAULT NULL, icd_10_113_code VARCHAR(16) DEFAULT NULL, age_char VARCHAR(16) DEFAULT NULL, age_num INTEGER DEFAULT NULL, year VARCHAR(16) DEFAULT NULL, year_num INTEGER DEFAULT NULL, deaths INTEGER DEFAULT NULL, population INTEGER DEFAULT NULL, crude_rate VARCHAR(16) DEFAULT NULL);

    LOAD DATA INFILE '/var/lib/mysql-files/Underlying Cause of Death, 2018-2022, Single Race_year_1_19_trunc.txt' INTO TABLE wonder_2018_2022 FIELDS TERMINATED BY '\t' OPTIONALLY ENCLOSED BY '"' IGNORE 1 ROWS;

Websites that were very helpful in the above commands

https://blog.skyvia.com/how-to-import-csv-file-into-mysql-table-in-4-different-ways/#Importing-CSV-into-MySQL-Automatically

https://stackoverflow.com/questions/32756710/count-specific-character-in-a-cell-in-openoffice-calc

https://stackoverflow.com/questions/29981417/mysql-load-data-infile-from-csv-fields-optionally-enclosed-by-quotes-numbers-h

https://stackoverflow.com/questions/32737478/how-should-i-resolve-secure-file-priv-in-mysql

## Loading WISQARS Database

To clean the WISQARS data for importing, I did the following

* Remove all asterisks. Some numbers are marked with asterisks to indicate "unstable value (<20 deaths)"
* All entries consisting of "--" indicate "suppressed value (between one to nine deaths)." To allow these to be imported as integers while still keeping them obvious as supressed values, I replaced all occurrences of "--" in the .csv file with -1 using emacs.
* As with WONDER, I removed the explanatory notes at the end.
* The population values are imported as text because SQL won't import them as integers since they contain commas. This is not relevant to the current analysis since w are comparing number of deaths. Calculating a rate would not yield more useful information since the denomniator (population) is the same.

# Data Analysis

## WONDER

Firearm categories
* Accidental discharge of firearms (W32-W34)
* Intentional self-harm (suicide) by discharge of firearms (X72-X74)
* Assault (homicide) by discharge of firearms (*U01.4,X93-X95)
* Discharge of firearms, undetermined intent (Y22-Y24)

## WISQARS

As the _Washington Post_ article notes, the WISQARS database "lists both deaths just from traffic-related crashes and an overall motor vehicle category that would include pedestrian and other deaths, such as death while in a stationary car."

Specifically, in WISQARS, one can use either the "Motor vehicle, traffic" mechanism or the "Motor Vehicle, Overall" mechanism, which consists of "Motor vehicle, traffic", "Pedal cyclist, other", "Pedestrian, other" and "Transport, other land".

    mysql> select year, SUM(deaths) from wisqars_2018_2022 WHERE mechanism = 'Motor vehicle, traffic' AND deaths>0 AND age<=19 AND age>=13 GROUP BY year ORDER BY year;
    +------+-------------+
    | year | SUM(deaths) |
    +------+-------------+
    | 2018 |        2486 |
    | 2019 |        2399 |
    | 2020 |        2804 |
    | 2021 |        3147 |
    | 2022 |        2898 |
    +------+-------------+

    mysql> select year, SUM(deaths) from wisqars_2018_2022 WHERE mechanism IN ('Motor vehicle, traffic', "Pedal cyclist, other", "Pedestrian, other", "Transport, other land") AND deaths>0 AND age<=19 AND age>=13 GROUP BY year ORDER BY year;
    +------+-------------+
    | year | SUM(deaths) |
    +------+-------------+
    | 2018 |        2557 |
    | 2019 |        2501 |
    | 2020 |        2908 |
    | 2021 |        3261 |
    | 2022 |        3029 |
    +------+-------------+


    mysql> select year, SUM(deaths) from wisqars_2018_2022 WHERE mechanism = 'Firearm' AND deaths>0 AND age<=19 AND age>=13 GROUP BY year ORDER BY year;
    +------+-------------+
    | year | SUM(deaths) |
    +------+-------------+
    | 2018 |        3039 |
    | 2019 |        3064 |
    | 2020 |        3879 |
    | 2021 |        4264 |
    | 2022 |        4167 |
    +------+-------------+


## Results





