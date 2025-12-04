# instalasi-E-prints-in-ubuntu

# instalasi Eprints 3.4.5 di ubuntu 

## 1. update sistem 
Update package list dan upgrade sistem terlebih dahulu di ubuntu kalian

`sudo apt update`
`sudo apt upgrade -y`

## 2. instalasi Apache web server
install apache

`sudo apt install apache -y`

setelah itu cek apakah apache sudah berjalan 
`sudo systemctl status apache2`

## 3. Instalasi depedensi yang dibutuhkan 

3. Instalasi Dependensi yang Dibutuhkan

3.1 Build Tools & Library

`sudo apt install wget curl build-essential libxml2-dev libgd-dev libapache2-mod-perl2 git -y`

3.2 Perl & Modul Utama

`sudo apt install perl libperl-dev cpanminus -y`

3.3 Dependensi Modul Perl

`sudo apt install libxml-libxml-perl libxml-sax-perl libxml-parser-perl \`
`libterm-readkey-perl libunicode-string-perl libmime-lite-perl `
`libmime-types-perl libdigest-sha-perl libdbi-perl libdbd-mysql-perl `
`libtext-unidecode-perl libjson-perl libdata-dumper-simple-perl `
`libsearch-xapian-perl libio-string-perl liblingua-stem-perl -y`

3.4 Modul Perl Tambahan via CPAN

`sudo cpanm CGI
sudo cpanm XML::LibXSLT
sudo cpanm Archive::Zip
sudo cpanm MIME::Base64`

⚠️ Kalau XML::LibXSLT error (versi conflict), install lewat apt:

sudo apt install libxml-libxslt-perl -y

4. Download & Instalasi EPrints

4.1 Masuk ke /opt

cd /opt

4.2 Download EPrints 3.4.5

sudo wget https://github.com/eprints/eprints3.4/archive/refs/tags/v3.4.5.tar.gz

4.3 Ekstrak

sudo tar -xzf v3.4.5.tar.gz

4.4 Rename Direktori

sudo mv eprints3.4-3.4.5 eprints3

4.5 Cek Instalasi

ls -l | grep eprints3

4.6 Masuk Direktori

cd eprints3


---

5. Buat User EPrints & Permission

5.1 Tambah User Sistem

sudo useradd -r -d /opt/eprints3 eprints

5.2 Set Ownership

sudo chown -R eprints:eprints /opt/eprints3

5.3 Cek Ownership

ls -l | grep eprints3

5.4 Cek epadmin

ls -la /opt/eprints3/bin/ | grep epadmin


---

6. Setup Root Password MariaDB/MySQL

6.1 Masuk MySQL

mysql

6.2 Jalankan SQL

> Ganti password root lo biar aman.



ALTER USER 'root'@'localhost' IDENTIFIED BY 'PasswordBaru123!';
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;

6.3 Test

mysql -u root -p


---

7. Buat Repository EPrints

7.1 Ganti User ke EPrints

sudo su - eprints

7.2 Masuk Direktori

cd /opt/eprints3

7.3 Jalankan Wizard

./bin/epadmin create pub

7.4 Isi Wizard Konfigurasi

(Tetep sama, cuma gue rapihin bahasa)

Pertanyaan Jawaban

hhhhhhhhhhhhhhhhhhhhhhhhhhh

## 8. Konfigurasi Apache

### 8.1 Edit apache2.conf

`sudo nano /etc/apache2/apache2.conf`

lalu tambahkan di paling bawah:

Include /opt/eprints3/cfg/apache.conf
ServerName localhost

Save: Ctrl + O → Enter → Ctrl + X

## 8.2 Generate Konfigurasi Apache

sudo su - eprints
cd /opt/eprints3
./bin/generate_apacheconf Eprints1

⚠️ Kalau muncul No files were changed:

./bin/generate_apacheconf --replace Eprints1

## 8.3 Aktifkan Site & Restart Apache

`exit`
`sudo cp /opt/eprints3/cfg/apache/Eprints1.conf /etc/apache2/sites-available/eprints.conf`
`sudo a2ensite eprints.conf`
`sudo systemctl restart apache2`

## 9. Testing EPrints

Akses:

👉 http://eprints.local

Kalau tampil “EPrints Done” berarti sukses.

Kalau halaman masih cache:
Ctrl + Shift + R

🔧 TROUBLESHOOTING

Troubleshooting 1: Error Database

Error:

Access denied for user 'www-data'@'localhost'

Solusi:

1. Cek User Database

sudo mysql -u root -p
CREATE USER IF NOT EXISTS 'Eprints1'@'localhost' IDENTIFIED BY 'eprints123';
GRANT ALL PRIVILEGES ON Eprints1.* TO 'Eprints1'@'localhost';
FLUSH PRIVILEGES;
EXIT;

2. Edit database.pl

sudo nano /opt/eprints3/archives/Eprints1/cfg/cfg.d/database.pl

Isinya harus kayak:

$c->{dbname} = 'Eprints1';
$c->{dbhost} = 'localhost';
$c->{dbuser} = 'Eprints1';
$c->{dbpass} = 'eprints123';
$c->{dbengine} = 'InnoDB';

3. Perbaiki Permission

sudo chown eprints:eprints /opt/eprints3/archives/Eprints1/cfg/cfg.d/database.pl
sudo chmod 640 /opt/eprints3/archives/Eprints1/cfg/cfg.d/database.pl

4. Generate Ulang Config

sudo su - eprints
cd /opt/eprints3
./bin/generate_apacheconf --replace Eprints1
exit

5. Copy ke Apache

sudo cp /opt/eprints3/cfg/apache/Eprints1.conf /etc/apache2/sites-available/eprints.conf
sudo a2ensite eprints.conf
sudo systemctl reload apache2

6. Restart Apache

sudo systemctl restart apache2

7. Test Lagi

👉 http://eprints.local

Troubleshooting 2: Error Permission (auto.js / folder nggak bisa ditulis)

Solusi:

### 1. Ganti Ownership Archive

sudo chown -R eprints:www-data /opt/eprints3/archives/Eprints1

### 2. Permission Folder

sudo find /opt/eprints3/archives/Eprints1 -type d -exec chmod 775 {} \;

### 3. Permission File

sudo find /opt/eprints3/archives/Eprints1 -type f -exec chmod 664 {} \;

### 4. Restart Apache

sudo systemctl restart apache2

### 5. Coba Buka

👉 http://eprints.local

6. Bersihkan Cache

Ctrl + Shift + R
