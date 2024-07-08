# The Question

> Are firearms or motor vehicle crashes the leading cause of deat of teenagers in the US?

A grim question to be sure, and one I heard debated recently. Since I am an avid believer in finding the truth and citing my sources, I tried searching for the answer online. This proved much more difficult than I thought because all of the articles I found referred to deaths of teenagers _and_ children, with age ranges of 0-17, 0-19, 1-17, and 1-19, but I could not find an answer to this question for the teenage age range (13-19). The articles I found include a very informative [_Washington Post_ Fact Check](https://www.washingtonpost.com/politics/2024/02/07/is-gun-violence-leading-cause-death-children/) that parses many of the claims and data analysis choices around this issue. It also references a [Johns Hopkins study](https://publichealth.jhu.edu/sites/default/files/2024-01/2023-june-cgvs-u-s-gun-violence-in-2021-v3.pdf) and an article from the [_New England Journal of Medicine_](https://www.nejm.org/doi/full/10.1056/NEJMc2201761). 


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

The CDC maintains two databases with the relevant information: WISQARS (used by ...) and WONDER (used by ...)

# Data Cleaning

## WONDER

Using the CDC WONDER tab file, I had to remove the "notes" at the end to prevent the error ERROR 1261 (01000): Row 2898 doesn't contain data for all columns

## WISQARS

For the WISQARS file, it contains serveral entries with a double asterisk after any number between 10 and 20. Several entries are just "--" in indicate a "indicates value (between one to nine deaths)" An example row:

    "Cut/Pierce","Homicide","18","2020","19**","4,410,589","0.43**","--","893"

This means that 19 18-year-olds were murdered by cutting or piercing in 2020 in the US. While the rest of this document is focused on data analysis, SQL methods, and my goal of answering a specific question, I want to make sure I remember something. While evaluating these data to find the truth and make our world a safer and better place, I think we always need to remember that each of these numbers represents a person's life cut short, a tragedy, and a grieving family. 

To clean the WISQARS data for importing, I did the following

* Remove all asterisks. Some numbers are marked with asterisks to indicate "unstable value (<20 deaths)"
* All entries consisting of "--" indicate "suppressed value (between one to nine deaths)." To allow these to be imported as integers while still keeping them obvious as supressed values, I replaced all occurrences of "--" in the .csv file with -1 using emacs.
* As with WONDER, I removed the explanatory notes at the end.
* The population values are imported as text because SQL won't import them as integers since they contain commas. This is not relevant to the current analysis since w are comparing number of deaths. Calculating a rate would not yield more useful information since the denomniator (population) is the same.

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



# Data Analysis

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

Fiream deaths are simpler as WISQARS had one mechanism for 'Firearm.'

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

It is important to note that many of these firearm deaths are suicides; this is indicated by the "intent" column.

    mysql> select year, intent, SUM(deaths) from wisqars_2018_2022 WHERE mechanism = 'firearm' AND deaths>0 AND age>=13 and age<=19 GROUP BY year,intent ORDER BY year,intent ;
    +------+--------------------+-------------+
    | year | intent             | SUM(deaths) |
    +------+--------------------+-------------+
    | 2018 | Homicide           |        1664 |
    | 2018 | Legal Intervention |          11 |
    | 2018 | Suicide            |        1253 |
    | 2018 | Undetermined       |          49 |
    | 2018 | Unintentional      |          62 |
    | 2019 | Homicide           |        1843 |
    | 2019 | Suicide            |        1129 |
    | 2019 | Undetermined       |          34 |
    | 2019 | Unintentional      |          58 |
    | 2020 | Homicide           |        2543 |
    | 2020 | Legal Intervention |          13 |
    | 2020 | Suicide            |        1239 |
    | 2020 | Undetermined       |          32 |
    | 2020 | Unintentional      |          52 |
    | 2021 | Homicide           |        2798 |
    | 2021 | Suicide            |        1358 |
    | 2021 | Undetermined       |          50 |
    | 2021 | Unintentional      |          58 |
    | 2022 | Homicide           |        2841 |
    | 2022 | Legal Intervention |          10 |
    | 2022 | Suicide            |        1210 |
    | 2022 | Undetermined       |          47 |
    | 2022 | Unintentional      |          59 |
    +------+--------------------+-------------+
## WONDER

The WONDER database has a similar separation of motor vehicle death causes, with "Transport accidents (V01-V99,Y85)" being the more inclusive and "Motor vehicle accidents (V02-V04,V09.0,V09.2,V12-V14,V19.0-V19.2,V19.4-V19.6,V20-V79,V80.3-V80.5,V81.0-V81.1,V82.0-V82.1,V83-V86,V87.0-V87.8,V88.0-V88.8,V89.0,V89.2)" the more specific.

    mysql> select year, SUM(deaths) from wonder_2018_2022 WHERE icd_10_113_cause='Motor vehicle accidents
    (V02-V04,V09.0,V09.2,V12-V14,V19.0-V19.2,V19.4-V19.6,V20-V79,V80.3-V80.5,V81.0-V81.1,V82.0-V82.1,V83-V86,V87.0-V87.8,V88.0-V88.8,V89.0,V89.2)' AND deaths>0 AND age_num<=19 AND age_num>=13 GROUP BY year OR
    DER BY year;
    +-------+-------------+
    | year  | SUM(deaths) |
    +-------+-------------+
    | 2018  |        2552 |
    | 2019  |        2509 |
    | 2020  |        2910 |
    | 2021  |        3249 |
    | 2022  |        3014 |
    +-------+-------------+

    
    mysql> select year, SUM(deaths) from wonder_2018_2022 WHERE icd_10_113_cause='Transport accidents (V01-V99,Y85)' AND deaths>0 AND age_num<=19 AND age_num>=13 GROUP BY year ORDER BY year;
    
    +-------+-------------+
    | year  | SUM(deaths) |
    +-------+-------------+
    | 2018  |        2640 |
    | 2019  |        2600 |
    | 2020  |        3021 |
    | 2021  |        3352 |
    | 2022  |        3113 |
    +-------+-------------+



Firearm categories
* Accidental discharge of firearms (W32-W34)
* Intentional self-harm (suicide) by discharge of firearms (X72-X74)
* Assault (homicide) by discharge of firearms (*U01.4,X93-X95)
* Discharge of firearms, undetermined intent (Y22-Y24)

    mysql> select year, SUM(deaths) from wonder_2018_2022 WHERE icd_10_113_cause IN ('Accidental discharge of firearms (W32-W34)', 'Intentional self-harm (suicide) by discharge of firearms (X72-X74)', 'Assault (homicide) by discharge of firearms (*U01.4,X93-X95)', 'Discharge of firearms, undetermined intent (Y22-Y24') AND deaths>0 AND age_num<=19 AND age_num>=13 GROUP BY year ORDER BY year;
  
    +-------+-------------+
    | year  | SUM(deaths) |
    +-------+-------------+
    | 2018  |        2979 |
    | 2019  |        3030 |
    | 2020  |        3834 |
    | 2021  |        4214 |
    | 2022  |        4110 |
    +-------+-------------+


## Results





