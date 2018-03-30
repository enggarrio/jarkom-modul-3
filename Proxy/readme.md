# PROXY
## **SYARAT SEBELUM MASUK KE MATERI INTI**
1. Buka **seluruh UML**.
2. Jalankan **iptables ...** di **ROUTER**.
3. Export proxy http, https, dan ftp di **seluruh UML**.
4. Jalankan **apt-get update** di **seluruh UML**.
5. Install **apache2** di **PUCANG**

# Proxy Server
## 1. Pengertian, Fungsi, dan Manfaat
### 1.1 Pengertian
Proxy server adalah sebuah server atau program komputer yang berperan sebagai penghubung antara suatu komputer dengan jaringan internet. Atau dalam kata lain, proxy server adalah suatu jaringan yang menjadi perantara antara jaringan lokal dan jaringan internet.

Proxy server dapat berupa suatu sistem komputer ataupun sebuah aplikasi yang bertugas menjadi gateway atau pintu masuk yang menghubungan komputer kita dengan jaringan luar.

### 1.2 Fungsi
1. ***Connection sharing*** :
Proxy bertindak sebagai gateway yang menjadi pembatas antara jaringan lokal dengan jaringan luar. Gateway bertindak juga sebagai sebuah titik dimana sejumlah koneksi dari pengguna lokal dan koneksi jaringan luar juga terhubung kepadanya. Oleh sebab itu, koneksi dari jaringan lokal ke internet akan menggunakan sambungan yang dimiliki oleh gateway secara bersama-sama (connecion sharing).
2. ***Filtering*** :
Proxy bisa difungsikan untuk bekerja pada layar aplikasi dengan demikian maka dia bisa berfungsi sebagai firewalll paket filtering yang dapat digunakan untuk melindungi jaringan lokal terhadap gangguan maupun ancaman serangan dari jaringan luar. Fungsi filtering ini juga dapat diatur atau dikonfigurasi untuk menolak akses terhadap situs web tertentu dan pada waktu- waktu tertentu juga.
3. ***Caching*** :
Sebuah proxy server mempunyai mekanisme penyimpanan obyek-obyek yang telah diminta dari server-server yang ada di internet. Dengan mekanisme caching ini maka akan menyimpan objek-objek yang merupakan berbagai permintaan/request dari para pengguna yang di peroleh dari internet.

### 1.3 Manfaat
Proxy server memiliki manfaat-manfaat berikut ini:
- Membagi koneksi
- Menyembunyikan IP
- Memblokir situs yang tidak diinginkan
- Mengakses situs yang telah diblokir
- Mengatur bandwith

## 2. Implementasi
Untuk praktikum jarkom kali ini, software proxy server yang digunakan adalah **SQUID**. UML yang digunakan sebagai proxy server adalah **PUCANG**.

### 2.1 Instalasi Squid
**STEP 1** - Pada UML **PUCANG**, ketikkan:

    apt-get install squid3

![Pucang1](images/1.png)


**STEP 2** - Cek status squid3 dengan mengetikkan 

    service squid3 status

![Pucang2](images/2.png)

Jika muncul status **ok** maka instalasi telah berhasil.

### 2.2 Konfigurasi Dasar Squid
**STEP 1** - Backup terlebih dahulu file konfigurasi default yang disediakan squid. Ketikkan perintah berikut untuk melakukan backup: 

    mv /etc/squid3/squid.conf /etc/squid3/squid.conf.bak

![Pucang3](images/3.png)

Perintah di atas artinya mengubah ekstensi file **squid.conf** menjadi **squid.conf.bak** dan menyimpannya di directory yang sama (tidak pindah folder).

**STEP 2** - Buat konfigurasi baru dengan mengetikkan:

    nano /etc/squid3/squid.conf
    
![Pucang4](images/4.png)

**STEP 3** - Kemudian, pada file config yang baru, ketikkan:

    http_port 8080
    visible_hostname pucang

![Pucang5](images/5.png)

Konfigurasi di atas berarti:
- Menggunakan port 8080
- Nama yang akan terlihat pada status: pucang

**STEP 4** - Restart squid dengan cara mengetikkan perintah:

    service squid3 restart

![Pucang6](images/6.png)

**STEP 5** - Ubah pengaturan proxy browser. Gunakan **IP PUCANG** sebagai host, dan isikan port **8080**.

**STEP 6** - Cobalah untuk mengakses web **its.ac.id** (usahakan menggunakan mode **incognito/private**). Seharusnya yang muncul adalah sebagai berikut.

![Pucang7](images/7.png)

**STEP 7** - Supaya bisa mengakses web **its.ac.id**, buka kembali file konfigurasi squid yang sudah dibuat tadi

**STEP 8** - Tambahkan baris berikut.

    http_access allow all

![Pucang8](images/8.png)

**STEP 9** - **Simpan** file konfigurasi tersebut, lalu **restart** squid.

**STEP 10** - Refresh halaman web **its.ac.id**. Seharusnya halaman yang ditampilkan kembali normal.

Keterangan:
- **http_access allow all** perlu ditambahkan karena pengaturan default squid adalah **deny**

### 2.3 Membuat User Login
**STEP 1** - Buat user dan password baru ke dalam squid. Ketikkan:

    htpasswd -c /etc/squid3/passwd asisten
    
![Pucang9](images/9.png)

Ketikkan password yang diinginkan. Jika sudah maka akan muncul notifikasi:

![Pucang10](images/10.png)

**STEP 2** - Edit konfigurasi squid menjadi:

    http_port 8080
    visible_hostname pucang
    
    auth_param basic program /usr/lib/squid3/ncsa_auth /etc/squid3/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS

![Pucang11](images/11.png)

**STEP 3** - Restart squid

**STEP 4** - Ubah pengaturan proxy browser. Gunakan **IP PUCANG** sebagai host, dan isikan port **8080**.

**STEP 5** - Cobalah untuk mengakses web **elearning.if.its.ac.id** (usahakan menggunakan mode **incognito/private**). Seharusnya muncul pop-up untuk login.

![Pucang12](images/12.png)

**STEP 6** - Isikan username dan password.

**STEP 7** - E-learning berhasil dibuka.

### 2.4 Pembatasan Waktu Akses

Kita akan mencoba membatasi akses proxy pada hari dan jam tertentu. Asumsikan proxy dapat digunakan hanya pada hari **Senin** sampai **Jumat** pada jam **08.00-16.00**.

**STEP 1** - Buat file baru bernama **acl.conf** di folder **squid3**

    nano /etc/squid3/acl.conf

![Pucang13](images/13.png)

**STEP 2** - Tambahkan baris berikut

    acl KERJA time MTWHF 08:00-16:00

![Pucang14](images/14.png)

**STEP 3** - Simpan file acl.conf

**STEP 4** - Buka file **squid.conf**

    nano /etc/squid3/squid.conf

**STEP 5** - Ubah konfigurasinya menjadi:

    include /etc/squid3/acl.conf
    
    http_port 8080
    http_access allow KERJA
    http_access deny all
    visible_hostname pucang

![Pucang15](images/15.png)

**STEP 6** - Simpan file tersebut. Kemudian restart squid

**STEP 7** - Cobalah untuk mengakses web **its.ac.id** (usahakan menggunakan mode **incognito/private**). Seharusnya muncul halaman error jika mengakses diluar waktu yang telah ditentukan.

![Pucang16](images/16.png)

Keterangan:
