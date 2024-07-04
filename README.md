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
