<p align="center">
  <a href="">
    <img src="https://a.fsdn.com/con/app/proj/icehrm/screenshots/Screen%20Shot%202016-02-09%20at%206.02.27%20AM.png/max/max/1" alt="ice hrm" width=400 height=400>
  </a>

# Apa itu IceHRM?
[`^ back to top ^`](#)
  <h3>IceHRM</h3>

  <p>
  IceHRM adalah sistem manajemen karyawan berbasis perangkat lunak
  Human Resources Management (HRM) yang memungkinkan perusahaan untuk memusatkan
  informasi rahasia karyawan, menetapkan izin akses kepada personel yang 
  berwenang untuk memastikan bahwa informasi karyawan aman dan dapat diakses
  dan mengelola kegiatan Sumber Daya Manusia dengan baik.
  </p>
  
</p>

# Instalasi
[`^ back to top ^`](#)

Requirements : Untuk menginstall Ice HRM membutuhkan beberapa hal, yaitu :

1. Apache2
2. MariaDB (database)
3. PHP 7.2

## Proses Instalasi 
#### Instalasi Web Server Virtual
*Sumber: https://github.com/auriza/komdat-lab/blob/master/p01.md*

1. Masuk ke 'Settings -> Network -> Advanced -> Port Forwarding' dan tambahkan dua aturan berikut.

2. Dengan demikian, jika kita mengakses port 8000 di host, maka akan diteruskan ke port 80 di guest (VM). Begitu juga dengan SSH, jika kita mengakses port 2200 di host, maka akan diteruskan ke port 22 di guest.

3. Setelah semuanya beres, jalankan VM dengan mode headless (tanpa tampilan).

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

4. Cek instalasi Apache dengan membuka laman http://localhost:8000.

#### Instalasi Ice HRM
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

6. Konfigurasikan Apache2

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

7. Aktifkan ICE HRM

```text
sudo a2ensite icehrm.conf
sudo a2enmod rewrite
```

8. Muat ulang Apache2

```text
sudo systemctl restart apache2.service
```

Kemudian pada browser ketik:
```text
localhost:8000/icehrm
```

Kemudian ikuti perintah yang muncul pada screen ketika IceHRM pertamakali dibuka. 

Anda dapat login dengan menggunakan username dan password di bawah ini:

username: admin
password: admin


# Konfigurasi

Jika telah mengunduh dan menginstal ICE HRM atau mendaftar ke layanan yang di hosting, Anda sekarang harus memiliki akses ke versi HRM ICE yang berfungsi. Ini memungkinkan Anda sebagai administrator sistem ini. Jadi sekarang saatnya untuk mulai mengkonfigurasi sistem baru Anda. Dalam tutorial ini **akan dikonfigurasikan** ICE HRM untuk perusahaan IT.

### 1. Menciptakan Struktur Perusahaan

Kita harus mulai dari sini, Anda akan membuat perusahaan yang memiliki dua departemen (Pengembangan dan QA). Nama perusahaan adalah ABC IT.

1. Untuk ini klik pertama pada item menu sisi kiri **"Admin" -> "Struktur Perusahaan"**, dan kemudian tambahkan struktur perusahaan baru menggunakan tombol tambah baru di sudut kanan atas.

2. Di sini Anda tidak perlu memilih *"Parent Structure"* (karena harus ada beberapa struktur yang tidak memiliki *parent structure*)

3. Sekarang pindah dan tambahkan dua departemen lain yang disebut Pengembangan dan QA. Pilih "ABC IT" sebagai *parent structure* dari kedua departemen.

<h1 align="center"><img src="https://0fc17599-a-49dc2a74-s-sites.googlegroups.com/a/gamonoid.com/ice-hrm/getting-started/company%20structure%20list.png?attachauth=ANoY7cqYE7dgnbYcetNn3gPzmiv3hbCi1Kvocvbe5iXtM1t8V2CXcBuv-8fKj_S0f2Lq_BA6CTxVaoVdS-7DC9cGJbwcqYyKby0qbA8smgN-76pdqydtkt4n_ihFLaYXfGme_Z9LPabGx1u3WfUo_8D7kvI8fZ_IKkdrsTQ2bS7WXiqNhXPgUTYoM5fibvMYV8NrE63orT2keEpL76HKLZqVpr_lRwe0jjEbQ2DUhFWmksseGH7fT7eqHTaYcoFJ8zKnvd3oDlXh&attredirects=0"></h1>

4. Dan Anda juga dapat melihat grafik struktur perusahaan. Ini membuat karyawan memiliki pandangan tingkat tinggi tentang bagaimana perusahaan dikelola.

<h1 align="center"><img src="https://0fc17599-a-49dc2a74-s-sites.googlegroups.com/a/gamonoid.com/ice-hrm/getting-started/company%20structure%20graph.png?attachauth=ANoY7crLRb2vgQNIVpU6-5Fgga2hMws7sZqnZB3wv90keGwzRJDlaCr2F-DIFiQUZVVE225CqyolmVXeZXCGPQaZksQvXOfJA8IGhhQiU1B2Pybeargll55HINxQ-OqXEEYqumvqDUDiZQNtfinku6G2KIPghsTIc7meNypeY__fEyAoHJNXxTiJ_YDZV1xzd4aV7k_shatzNGMXKuKqVmh7PgovA-ESxZK2EjluSg5fank-bQ2eMr0rDk5__VY1gOZ--34ri9ES&attredirects=0"></h1>

### 2. Membuat Judul Pekerjaan / Nilai Pembayaran / Status Pekerjaan

1. Klik menu "Admin-> Pekerjaan" dan pilih tab Judul Pekerjaan

2. Sekarang tambahkan dua Judul Pekerjaan (menggunakan tombol Tambah Baru di sudut kanan atas), yaitu Software Engineer dan QA Engineer

<h1 align="center"><img src="https://0fc17599-a-49dc2a74-s-sites.googlegroups.com/a/gamonoid.com/ice-hrm/getting-started/Job%20Titles.png?attachauth=ANoY7cqrzFYNOvIlnboLZtZ9MlZT9dqNZ25mqgAx7-t7r3f2-5malgN7_DXPKpVAmjs8jrYxNE7XH0CkCC-QAl7pSJjlUbSBu0J3kwa0bo6H-VUGTdnThxDsGVIFzCbfPh4SC74GXAqRARkKRGZ64swhlvnadNdy8s6pWaM0Mqk7gG4i3aim3N9G2cNiJQu4AUYhbpumzgPdzKGlDvlpNcw7X8YDjUzYZLmSLGtfIkaE6Ie-ibANyDo%3D&attredirects=0"></h1>

3. Sekarang Pilih tab Nilai Pembayaran dan tambahkan nilai pembayaran baru

<h1 align="center"><img src="https://0fc17599-a-49dc2a74-s-sites.googlegroups.com/a/gamonoid.com/ice-hrm/getting-started/pay%20grade.png?attachauth=ANoY7cq2TMOa41IzXicegFoRnvWNrUzj2CkZIWQYVb_3cE4DTQLuirdbq4j1Vsr08yfTAQM3573At4Wij8zVhf_Fve0q_1dU7FTBm2uQvwGtwEs8IKPV2pjTOM5lNj9S46MBl-uoy20gp9hcOE2liptOiYcp5m2Yq_XJR5__KclrnYJypYFkjFfLf-JL3QZCsDhkgRkL2SrMT7f-0eHS5ABcfMsYOY9ge57b_fnJWEbBj1YgAzvi_Kk%3D&attredirects=0"></h1>


# Otomatisasi






# Cara Pemakaian

...

# Pembahasan
Kelebihan dan kekurangan iceHRM

1. Penggunaan iceHRM menjadi mudah karena lebih modern interfacenya. 
2. Beberapa fitur unggulannya adalah leave management, tracking waktu dan kehadiran, informasi karyawan, serta upload dokumen sekaligus download report dalam format CSV.
3. Support yang mumpuni juga menjadikan iceHRM adalah opsi kuat untuk software kehadiran gratis/open source. Karena sasaran utama adalah usaha kecil dan menengah, jadi software ini mungkin bukan yang terbaik untuk berfungsi maksimal di perusahaan besar.

Sasaran dari aplikasi ini adalah untuk usaha kecil dan menengah, jadi aplikasi ini mungkin bukan yang terbaik untuk usaha menengah ke atas atau bagi perusahaan perusahaan yang besar.

# Referensi
1. https://icehrm.com/
2. https://github.com/gamonoid/icehrm
3. https://entrepreneursquad.id/6-tools-berbasis-teknologi-untuk-hrd/
