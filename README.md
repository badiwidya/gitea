<p align="center"><img src="img/gitea.png" width="200" alt="Gitea logo"></p>

# Gitea

<!--toc:start-->

- [Gitea](#gitea)
  - [Sekilas Tentang](#sekilas-tentang)
  - [Instalasi](#instalasi)
    - [Sistem Operasi](#sistem-operasi)
    - [Proses Instalasi](#proses-instalasi)
  - [Konfigurasi](#konfigurasi)
  - [Maintenance](#maintenance)
  - [Otomatisasi](#otomatisasi)
  - [Cara Pemakaian](#cara-pemakaian)
  - [Pembahasan](#pembahasan)
  - [Referensi](#referensi)
  <!--toc:end-->

## Sekilas Tentang

Gitea adalah sebuah platform pengembangan perangkat lunak lengkap (all-in-one) yang bersifat open-source dan dapat di-hosting secara mandiri (self-hosted). Fungsi utamanya adalah menyediakan layanan Git untuk kontrol versi (version control), yang dilengkapi dengan berbagai fitur pendukung kolaborasi tim, seperti pelacakan isu (issue tracking), ulasan kode (code review) melalui pull request, wiki untuk dokumentasi, hingga registri paket (package registry). Secara konsep dan antarmuka, Gitea sangat mirip dengan platform populer seperti GitHub dan GitLab, namun dengan fokus utama pada efisiensi, kecepatan, dan kemudahan instalasi.

Proyek Gitea lahir pada tahun 2016 sebagai sebuah fork (pengembangan cabang) dari proyek Gogs, dengan tujuan untuk membangun model pengembangan yang lebih terbuka dan digerakkan oleh komunitas. Ditulis menggunakan bahasa pemrograman Go, Gitea dapat dijalankan sebagai sebuah binary tunggal di berbagai sistem operasi (Linux, macOS, Windows) dan arsitektur prosesor (x86, amd64, ARM). Hal ini membuatnya sangat portabel dan tidak memiliki banyak dependensi eksternal, yang menyederhanakan proses instalasi dan pemeliharaan secara signifikan.

Salah satu filosofi desain utama Gitea adalah ringan dan cepat (lightweight and fast). Kebutuhan sumber daya sistemnya sangat rendah, di mana untuk tim atau proyek skala kecil, Gitea dapat berjalan dengan lancar hanya dengan 2 core CPU dan 1GB RAM. Bahkan, Gitea cukup efisien untuk dijalankan pada perangkat sekelas Raspberry Pi. Efisiensi ini menjadi keunggulan kompetitif utamanya dibandingkan solusi lain yang lebih kompleks dan haus sumber daya seperti GitLab.

**Tujuan Proyek**

Pemilihan Gitea sebagai aplikasi web untuk proyek mata kuliah KDJK ini didasari oleh beberapa pertimbangan, yaitu dengan kebutuhan tim pengembangan skala kecil. Tujuan utama projek ini untuk membangun infrastruktur version control yang lebih private, cepat, dan dikelola kelompok kami.
Berikut adalah alasan kami memilih Gitea:
1. **Data Sovereignty**,
   Dalam lingkungan pengembangan, biasanya untuk proyek tahap awal, menyimpan kode di server pribadi memberikan lapisan keamanan dan privasi tambahan. Dengan self-hosting Gitea, kami memiliki kebebasan untuk mengatur data repositori, bebas dari potensi keterbatasan layanan pihak ketiga. Konsep ini sesuai dengan materi jaringan di mana kita membangun dan mengelola server kita sendiri.
2. **Efisiensi Resource dan Cost**,
   Pada proyek studi ini, kami bekerja dengan sumber daya yang terbatas. Gitea, dengan sifatnya yang light, memungkinkan kami untuk menginstallnya pada Virtual Private Server (VPS) dengan spesifikasi yag rendah tanpa mengorbankan performa. Hal ini mengurangi biaya operasional dibandingkan jika kami harus menyewa server untuk menjalankan pilihan seperti GitLab. Sehingga menunjukkan bahwa jaringan yang reliable tidak selalu membutuhkan infrastruktur yang mahal.
3. **Kemudahan Instalasi**,
   Proses instalasi Gitea yang dipermudah dengan Docker membuat kami bisa menyiapkan server dalam waktu singkat. Fokus utama kami adalah pada pemahaman konsep jaringan dan layanan, bukan pada kerumitan administrasi sistemnya. Kami dapat langsung fokus pada konfigurasi jaringan seperti reverse proxy dengan Nginx dan manajemen layanan.
   
   
4. **Suplemen untuk Belajar Layer Aplikasi**,
   Mengelola Gitea memberikan pengalaman bagi kami dalam administrasi layanan jaringan dari Layer Aplikasi pada model OSI/TCP-IP. Kami menjadi administrator yang bertanggung jawab atas uptime, keamanan, dan pemeliharaan server, tidak sebagai pengguna saja. Pengalaman memberikan pembelajaran yan gsesuai dengan materi-materi yang sudah dibahas di kelas, seperti pada materi Applications and Layered Architectures, pertemuan kedua, di mana Gitea berperan sebagai Application Layer Service.

![TCP/IP](img/OSITCP.jpg)
*Gambar 2. Arsitektur TCP/IP*

   
   
## Instalasi

### Sistem Operasi

- Ubuntu Server 24.04

### Proses Instalasi

1. Update sistem dengan menjalankan perintah di bawah ini.

```bash
sudo apt-get update && sudo apt-get upgrade
```

2. Instal Docker.

```bash
# Tambahkan GPG Key Docker Official
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Tambahkan repository Docker ke dalam source list Apt
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Instal paket Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Aktifkan service docker
sudo systemctl enable --now docker.service
```

3. Instal Gitea dengan menggunakan Docker Compose.

Buat direktori baru di home dan buat docker compose di dalamnya.

```bash
mkdir -p $HOME/gitea && cd $HOME/gitea

# Bisa juga menggunakan text-editor selain nano (contoh vi)
nano docker-compose.yaml
```

Tempelkan konten `docker-compose.yaml` di bawah ini ke dalamnya.

```yaml
networks:
  gitea:
    external: false
volumes:
  gitea_data:
    driver: local
  postgres_data:
    driver: local
services:
  server:
    image: docker.gitea.com/gitea:1.24.6
    container_name: gitea
    restart: always
    networks:
      - gitea
    volumes:
      - gitea_data:/gitea
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5432
      - GITEA__database__NAME=${DB_NAME:-gitea}
      - GITEA__database__USER=${DB_USER:-gitea}
      - GITEA__database__PASSWORD=${DB_PASS:-gitea}
    ports:
      - "127.0.0.1:3000:3000"
    depends_on:
      - db
  db:
    image: postgres:17
    restart: always
    environment:
      - POSTGRES_USER=${DB_USER:-gitea}
      - POSTGRES_PASSWORD=${DB_PASS:-gitea}
      - POSTGRES_DB=${DB_NAME:-gitea}
    networks:
      - gitea
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

Simpan file dengan menekan tombol `CTRL+O` lalu `ENTER` dan keluar dari nano dengan menekan tombol `CTRL+X`.

> Secara default pengaturan **database** adalah sebagai berikut.
>
> - **Username**: gitea
> - **Password**: gitea
> - **Nama Database**: gitea
>
> Namun, ini bisa diganti dengan cara langsung mengganti value di `docker-compose.yaml` atau dengan menambahkan **shell environment variables**.
>
> ```bash
> export DB_USER=username_database
> export DB_PASS=password_database
> export DB_NAME=nama_database
> ```

Selanjutnya jalankan container dengan perintah berikut.

```bash
sudo docker compose up -d
```

4. Instal dan konfigurasi Nginx Web Server.

Instal nginx menggunakan apt.

```bash
sudo apt-get install nginx
```

Selanjutnya buat konfigurasi nginx baru.

```bash
sudo nano /etc/nginx/conf.d/gitea.conf
```

Tempel konten `/etc/nginx/conf.d/gitea.conf` di bawah ini.

```nginx
server {
    listen 80;

    server_name localhost 127.0.0.1;

    client_max_body_size 512M;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> catatan: `server_name localhost 127.0.0.1` bisa diganti jika memiliki nama domain (contoh `server_name gitea-lab.ddns.net`).

Lalu reload service nginx.

```bash
sudo systemctl reload nginx
```

5. Mengaktifkan HTTPS dengan sertifikat SSL/TLS (opsional).

Langkah ini bersifat opsional. Jika kita memiliki sertifikat SSL/TLS (misal dari [Let's Encrypt](https://letsencrypt.org)), HTTPS bisa diaktifkan pada situs Gitea ini.

Instal certbot.

```bash
sudo apt-get install certbot python3-certbot-nginx
```

Lalu jalankan perintah.

```bash
sudo certbot --nginx -d gitea-lab.ddns.net
```

> catatan: ganti `gitea-lab.ddns.net` dengan nama domain yang didaftarkan di [Let's Encrypt](https://letsencrypt.org).

## Konfigurasi
-----

## **3. Konfigurasi (Opsional)**

Setelah Gitea berhasil diinstal dan berjalan di dalam kontainer Docker, langkah selanjutnya adalah melakukan konfigurasi pada level server agar layanan dapat diakses dari luar jaringan secara aman dan profesional. Konfigurasi ini fokus pada bagaimana lalu lintas (*traffic*) jaringan diarahkan dan diamankan sebelum mencapai aplikasi. Pada proyek ini, kami mengimplementasikan dua konfigurasi utama, yaitu penggunaan Nginx sebagai *reverse proxy* dan pengamanan koneksi menggunakan SSL/TLS (HTTPS) dengan Certbot.

### **Reverse Proxy Nginx**

*Reverse proxy* adalah server perantara yang ditempatkan di depan server web utama (dalam kasus ini, kontainer Gitea). Fungsinya adalah menerima semua permintaan dari klien (pengguna) dan meneruskannya ke server Gitea. Penggunaan *reverse proxy* memberikan beberapa manfaat, seprti:

* **Load Balancing:** *reverse proxy* dapat mendistribusikan beban ke beberapa server aplikasi
* **Keamanan:** Menyembunyikan arsitektur jaringan internal. Klien hanya berinteraksi dengan Nginx, bukan langsung dengan Gitea
* **Fleksibilitas:** Memungkinkan hosting beberapa domain atau aplikasi pad asatu alamat IP yang sama
* **Terminasi SSL:** Mengelola enkripsi dan dekripsi HTTPS di satu tempat (Nginx), menyederhanakan konfigurasi aplikasi di belakangnya

Konfigurasi Nginx untuk Gitea didefinisikan dalam file `/etc/nginx/conf.d/gitea.conf`. Berikut adalah isi dari file konfigurasi tersebut:

```nginx
server {
    listen 80;
    server_name gitea-lab.ddns.net; # Ganti dengan domain atau IP server kita

    client_max_body_size 512M;

    location / {
        proxy_pass [http://127.0.0.1:3000](http://127.0.0.1:3000);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
````

**Penjelasan Konfigurasi:**

  * `listen 80;`: Memberi instruksi pada Nginx untuk "mendengarkan" semua koneksi masuk pada port 80, yang merupakan port standar untuk protokol HTTP
  * `server_name gitea-lab.ddns.net;`: Menentukan nama domain yang akan dilayani oleh blok `server` ini. Nginx menggunakan konfigurasi ini jika permintaan datang dengan *Host header* yang cocok dengan domain ini
  * `client_max_body_size 512M;`: Mengatur ukuran maksimum badan permintaan dari klien. Nilai ini dinaikkan menjadi 512 MB untuk memungkinkan pengunggahan repositori Git atau file besar lainnya
  * `location / { ... }`: Blok ini menangani semua permintaan yang masuk ke domain tersebut
  * `proxy_pass http://127.0.0.1:3000;`: Perintah ini meneruskan (*forwards*) semua permintaan yang diterima ke alamat `http://127.0.0.1:3000`, yaitu alamat lokal tempat kontainer Gitea mem-publikasikan layanannya
  * `proxy_set_header ...`: Perintah ini memodifikasi atau meneruskan *header* HTTP dari klien ke server Gitea, agar Gitea dapat mengetahui informasi asli dari klien (seperti alamat IP-nya), tidak dnegan melihat semua permintaan seolah olah berasal dari Nginx (`127.0.0.1`)

![nginx reverse proxy](img/nginx_reverseproxy.jpg)
*Gambar 4. Alur Kerja Nginx sebagai Reverse Proxy*

### **HTTPS dengan Certbot (Opsional)**

Untuk meningkatkan keamanan, disarankan untuk mengenkripsi komunikasi antara pengguna dan server menggunakan HTTPS. HTTPS mencegah pihak ketiga melakukan penyadapan (*eavesdropping*) atau manipulasi data (*man-in-the-middle attack*). Kami menggunakan **Let's Encrypt**, sebuah otoritas sertifikat (CA) gratis, dan **Certbot**, sebuah alat yang mengotomatiskan proses permintaan dan instalasi sertifikat SSL/TLS.

Berikut adalah langkah-langkah untuk mengaktifkan HTTPS:

1.  **Instal Certbot dan Plugin Nginx-nya:**
    Perintah ini menginstal Certbot beserta plugin yang dirancang khusus untuk bekerja dengan Nginx. Plugin ini dapat membaca konfigurasi Nginx kita dan memodifikasinya secara otomatis untuk mengaktifkan HTTPS.
    ```bash
    sudo apt-get install certbot python3-certbot-nginx -y
    ```
2.  **Minta dan Instal Sertifikat:**
    Perintah ini menjalankan Certbot dengan plugin Nginx (`--nginx`) dan menargetkan domain yang ditentukan (`-d gitea-lab.ddns.net`).
    ```bash
    sudo certbot --nginx -d gitea-lab.ddns.net
    ```
    Selama proses ini, Certbot akan:
      * Memverifikasi bahwa kita memiliki kontrol atas domain tersebut
      * Mengunduh sertifikat SSL/TLS dari Let's Encrypt
      * Mengedit file `/etc/nginx/conf.d/gitea.conf` secara otomatis untuk menambahkan konfigurasi HTTPS, termasuk mengarahkan semua lalu lintas HTTP ke HTTPS
      * Menyiapkan *cron job* atau *systemd timer* untuk memperbarui sertifikat secara otomatis sebelum masa berlakunya habis

Setelah proses ini selesai, Nginx akan dikonfigurasi ulang untuk melayani Gitea melalui port 443 (HTTPS) dengan enkripsi yang valid, dan situs dapat diakses dengan aman melalui `https://gitea-lab.ddns.net`.


## Maintenance

## Otomatisasi

## Cara Pemakaian

## Pembahasan

## Referensi

- Gitea Documentation - [https://docs.gitea.com/](https://docs.gitea.com/)
- Install Docker Engine on Ubuntu - [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
- nginx documentation - [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
