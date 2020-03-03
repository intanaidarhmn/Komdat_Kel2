<p align="center">
  <a href="">
    <img src="https://github.com/intanaidarhmn/Komdat_Kel2/pict/ice hrm.png" alt="ice hrm" width=400 height=400>
  </a>

# Apa itu Ice Hrm?

  <h3>Ice Hrm</h3>

  <p>
  Ice Hrm adalah sistem manajemen karyawan IceHrm memungkinkan perusahaan untuk memusatkan
  informasi karyawan rahasia dan menetapkan izin akses kepada personel yang 
  berwenang untuk memastikan bahwa informasi karyawan aman dan dapat diakses.
  </p>
  
  <p>Sumber : https://icehrm.com/</p>
</p>

# Instalasi

Requirements : Untuk menginstall Ice HRM membutuhkan beberapa hal, yaitu :

1. Apache2
2. MariaDB (database)
3. PHP 7.2

# Konfigurasi

Bisa ngapain aja ga si

# Otomatisasi
## Instalasi Web Server Virtual

https://github.com/auriza/komdat-lab/blob/master/p01.md

1. Masuk ke 'Settings -> Network -> Advanced -> Port Forwarding' dan tambahkan dua aturan berikut.

2. Dengan demikian, jika kita mengakses port 8000 di host, maka akan diteruskan ke port 80 di guest (VM). Begitu juga dengan SSH, jika kita mengakses port 2200 di host, maka akan diteruskan ke port 22 di guest.

Setelah semuanya beres, jalankan VM dengan mode headless (tanpa tampilan).

: Aturan port forwarding

Buka terminal di komputer host, dan akses VM dengan username dan password student.

```text
# akses vm dari host
ssh student@localhost -p 2200

# set repo
sudo tee /etc/apt/sources.list << !
deb http://repo.apps.cs.ipb.ac.id/ubuntu bionic          main restricted universe multiverse
deb http://repo.apps.cs.ipb.ac.id/ubuntu bionic-updates  main restricted universe multiverse
deb http://repo.apps.cs.ipb.ac.id/ubuntu bionic-security main restricted universe multiverse
!

# instal apache, mysql, php
sudo apt update
sudo apt install apache2 php mysql-server
sudo apt install php-mysql php-gd php-mbstring php-xml php-curl
sudo service apache2 restart
```

Cek instalasi Apache dengan membuka laman http://localhost:8000.

## Instalasi Ice HRM
1. Install Apache2 HTTP Server

```text
sudo apt update
sudo apt install apache2
```

Kemudian, run commands di bawah ini untuk stop, start, dan enable Apache2 service
agar selalu terhubung dengan server boots.

```text
sudo systemctl stop apache2.service
sudo systemctl start apache2.service
sudo systemctl enable apache2.service
```

2. Install MariaDB

```text
sudo apt-get install mariadb-server mariadb-client
```

Run commands di bawah ini jika menggunakan Ubuntu 17.10 and 18.04 LTS
```text
sudo systemctl stop mariadb.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

Untuk mengamankan server MariaDB, di buat root password dan larangan untuk akses secara remote.
```text
sudo mysql_secure_installation
```

Jika muncul pertanyaan, jawab sebagai berikut:
```text
Enter current password for root (enter for none): Just press the Enter
Set root password? [Y/n]: Y
New password: Enter password
Re-enter new password: Repeat password
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y
```
Restart MariaDB server
```text
sudo systemctl restart mysql.service
```
3. Install PHP 7.2 dan modul-modul yang dibutuhkan

```text
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
```

Update dan upgrade ke PHP 7.2
```text
sudo apt update
```

Jalankan perintah seperti di bawah ini untuk install PHP 7.2 dan modul-modul yang terkait.
```text
sudo apt install php7.2 php7.2-common php7.2-mbstring php7.2-xmlrpc php7.2-soap php7.2-gd php7.2-xml php7.2-intl php7.2-mysql php7.2-cli php7.2 php7.2-ldap php7.2-zip php7.2-curl
```

Setelah berhasil install PHP, jalankan perintah di bawah untuk membuka FPM PHP default file.
```text
sudo nano /etc/php/7.2/apache2/php.ini
```
Kemudian di dalam file php.ini kita ubah baris-baris berikut:
```text
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
date.timezone = America/Chicago
```

4. Membuat ICE HRM Database

```text
sudo mysql -u root -p
```

Kemuian kita buat database dengan nama icehrm
```text
CREATE DATABASE icehrm;
```

Buat database user dengan nama icehrmuser dengan password
```text
CREATE USER 'icehrmuser'@'localhost' IDENTIFIED BY 'new_password_here';
```

Kemudian grant acces pada user icehrmuser untuk mengakses database icehrmuser.
```text
GRANT ALL ON icehrm.* TO 'icehrmuser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
```

Kemudian submit perintah dan exit dari SQL.
```text
FLUSH PRIVILEGES;
EXIT;
```

5. Download ICE HRM Latest Release

```text
cd /tmp && wget https://github.com/gamonoid/icehrm/releases/download/v23.0.0.OS/icehrm_v23.0.0.OS.zip
unzip icehrm_v23.0.0.OS.zip
sudo mv icehrm_v23.0.0.OS /var/www/html/icehrm
```

kemudian jalankan perintah di bawah ini untuk mengarut perizinan yang benar bagi Concrete5.
```text
sudo chown -R www-data:www-data /var/www/html/icehrm/
sudo chmod -R 755 /var/www/html/icehrm/
```

6. Konfigur Apache2

```text
sudo nano /etc/apache2/sites-available/icehrm.conf
```

Setelah itu copy dan paste perintah di bawah ini ke file icehrm.conf, kemudian simpan.
sesuaikan nama domain dengan nama domain masing-masing.
```text
<VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/html/icehrm
     ServerName example.com
     ServerAlias www.example.com

     <Directory /var/www/html/icehrm/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
Simpan kemudian exit.

7. Enable the ICE HRM

```text
sudo a2ensite icehrm.conf
sudo a2enmod rewrite
```

8. Restart Apache2

```text
sudo systemctl restart apache2.service
```

Kemudian pada browser ketik:
```text
localhost:8000/icehrm
```

Kemudian ikuti perintah yang muncul pada screen ketika Ice Hrm pertamakali dibuka. 

Anda dapat login dengan menggunakan username dan password di bawah ini:

username: admin
password: admin


# Cara Pemakaian

...

# Pembahasan

https://entrepreneursquad.id/6-tools-berbasis-teknologi-untuk-hrd/

Kelebihan dan kekurangan iceHRM

1. Penggunaan iceHRM menjadi mudah karena lebih modern interfacenya. 
2. Beberapa fitur unggulannya adalah leave management, tracking waktu dan kehadiran, informasi karyawan, serta upload dokumen sekaligus download report dalam format CSV.
3. Support yang mumpuni juga menjadikan iceHRM adalah opsi kuat untuk software kehadiran gratis/open source. Karena sasaran utama adalah usaha kecil dan menengah, jadi software ini mungkin bukan yang terbaik untuk berfungsi maksimal di perusahaan besar.



# Referensi


