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

-----
## **4. Otomatisasi**

Salah satu nilai lebih dari proyek ini adalah implementasi alur kerja otomatis untuk menjaga konsistensi antara repositori utama yang ada di GitHub dengan server Gitea pribadi yang telah kita siapkan. Daripada melakukan proses *push* manual ke dua lokasi berbeda (*dual push*), kami membangun sebuah alur kerja *Continuous Integration/Continuous Deployment* (CI/CD) sederhana menggunakan **GitHub Actions**.

Tujuan dari otomatisasi ini adalah menjadikan repositori GitHub sebagai *single source of truth* (sumber kebenaran tunggal). Setiap kali ada perubahan yang digabungkan ke cabang `main` di GitHub, sebuah proses otomatis akan terpicu untuk mencerminkan (*mirroring*) perubahan tersebut ke server Gitea kita. Hal ini memastikan bahwa server Gitea selalu memiliki versi kode terbaru tanpa intervensi manual, mengurangi risiko kesalahan manusia dan meningkatkan efisiensi alur kerja tim.

### **Sinkronisasi GitHub ke Gitea dengan GitHub Actions**

Mekanisme otomatisasi ini diatur dalam sebuah file *workflow* yang terletak di `.github/workflows/sync-to-gitea.yml`. File ini menggunakan sintaks YAML untuk mendefinisikan serangkaian perintah yang akan dijalankan oleh *runner* (server virtual) yang disediakan oleh GitHub.

Berikut adalah isi lengkap dari file *workflow* tersebut:

```yaml
name: Sync repo to Gitea
on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set git identity
        run: |
          git config --global user.name "${GIT_NAME}"
          git config --global user.email "${GIT_EMAIL}"
        env:
          GIT_NAME: ${{ vars.GIT_NAME }}
          GIT_EMAIL: ${{ vars.GIT_EMAIL }}

      - name: Push to Gitea
        run: |
          git remote add gitea https://${GIT_NAME}:${GITEA_TOKEN}@gitea-lab.ddns.net/kdjk4/gitea.git
          git push --force gitea main
        env:
          GIT_NAME: ${{ vars.GIT_NAME }}
          GITEA_TOKEN: ${{ secrets.GITEA_TOKEN }}
````

**Penjelasan Alur Kerja (*Workflow*):**

1.  **`name: Sync repo to Gitea`**: Memberikan nama yang deskriptif untuk *workflow* ini, yang akan muncul di tab "Actions" pada repositori GitHub.

2.  **`on:`**: Mendefinisikan pemicu (*trigger*) yang akan menjalankan *workflow*.

      * `push: branches: ["main"]`: *Workflow* akan otomatis berjalan setiap kali ada `git push` ke cabang `main`.
      * `workflow_dispatch:`: Memungkinkan *workflow* dijalankan secara manual melalui antarmuka web GitHub.

3.  **`jobs:`**: Mendefinisikan satu atau lebih pekerjaan (*jobs*) yang akan dieksekusi.

      * **`sync:`**: Nama dari pekerjaan yang kita buat.
      * **`runs-on: ubuntu-latest`**: Menentukan bahwa pekerjaan ini akan berjalan di lingkungan server virtual Ubuntu versi terbaru yang disediakan oleh GitHub.

4.  **`steps:`**: Serangkaian langkah atau perintah yang akan dieksekusi secara berurutan di dalam *job*.

      * **`name: checkout`**: Langkah pertama adalah mengunduh salinan dari repositori GitHub ke dalam *runner* menggunakan *action* siap pakai `actions/checkout@v4`. Opsi `fetch-depth: 0` memastikan seluruh riwayat *commit* diunduh, yang penting untuk proses *mirroring*.

      * **`name: Set git identity`**: Mengkonfigurasi identitas Git (nama dan email) di dalam *runner*. Identitas ini diambil dari **Repository Variables** (`vars.GIT_NAME` dan `vars.GIT_EMAIL`) yang disimpan di pengaturan repositori GitHub.

      * **`name: Push to Gitea`**: Ini adalah langkah inti dari otomatisasi.

          * `git remote add gitea ...`: Menambahkan server Gitea kita sebagai *remote repository* baru dengan nama `gitea`. URL-nya mencakup kredensial otentikasi.
          * **Aspek Keamanan**: Perhatikan penggunaan `${GITEA_TOKEN}`. Ini adalah *access token* yang dibuat di Gitea dan disimpan sebagai **Repository Secret** (`secrets.GITEA_TOKEN`) di GitHub. Menggunakan *secrets* memastikan bahwa kredensial sensitif tidak terekspos dalam kode atau log.
          * `git push --force gitea main`: Melakukan *force push* dari cabang `main` lokal (yang merupakan salinan dari GitHub) ke *remote* `gitea`. Opsi `--force` digunakan untuk memastikan repositori Gitea menjadi cerminan identik dari repositori GitHub, menimpa riwayat yang mungkin berbeda.


![continuous integration](img/continuous_integration.jpg)
[Image of a CI/CD workflow diagram]

*Gambar 5. Alur Kerja Otomatisasi Sinkronisasi*

Dengan implementasi ini, kami dapat mengintegrasikan dua layanan Git yang berbeda (satu publik, satu privat) menjadi satu alur kerja yang kohesif dan otomatis, yang merupakan penerapan praktis dari konsep CI/CD dalam manajemen kode.



## Cara Pemakaian

## Pembahasan
## **6. Pembahasan**

Bagian ini menyajikan analisis mendalam mengenai aplikasi Gitea berdasarkan pengalaman langsung selama proses instalasi, konfigurasi, dan penggunaan, yang diperkaya dengan data dari studi kasus dan riset akademis. Pembahasan akan mencakup evaluasi kelebihan dan kekurangan Gitea secara kritis, serta perbandingan komparatif dengan platform sejenis untuk memberikan konteks yang lebih luas.

### **Kelebihan Gitea (Berdasarkan Pengalaman dan Analisis)**

#### **1. Ringan dan Cepat**

Salah satu keunggulan utama yang kami rasakan secara langsung adalah performa Gitea yang sangat responsif dengan kebutuhan sumber daya yang sedikit. Gitea menunjukkan konsumsi CPU dan memori yang jauh lebih rendah dibandingkan dengan platform DevOps lengkap seperti GitLab. Gitea yang ditulis dalam bahasa Go, yang menghasilkan *single binary* dengan dependensi minimal.

Dalam konteks proyek kami yang menggunakan VPS dengan sumber daya terbatas, Gitea dapat berjalan dengan lancar tanpa menyebabkan *bottleneck* pada sistem. Gitea sangat menguntungkan bagi organisasi dengan infrastruktur terbatas di mana efisiensi biaya dan alokasi sumber daya menjadi faktor pentingnya.

#### **2. Full Control dan Data Sovereignty **

Dengan meng-hosting Gitea pada server pribadi, kami memiliki kontrol penuh atas kode sumber dan data proyek. Keunggulan ini tidak didapatkan dari layanan berbasis *cloud public*. Semua data, mulai dari repositori, isu, hingga data pengguna, tersimpan di dalam infrastruktur yang kami kelola, di belakang *firewall* kami. Hal ini mengurangi risiko yang terkait dengan kebijakan privasi pihak ketiga, potensi kebocoran data, atau pemadaman layanan yang berada di luar kendali kami.

Penggunaan Docker dalam instalasi Gitea juga menambahkan lapisan keamanan melalui isolasi. Kontainer Gitea berjalan dalam lingkungan yang terpisah dari sistem operasi *host*, yang membatasi potensi kerusakan jika terjadi celah keamanan pada aplikasi. Meskipun kontainerisasi memperkenalkan tantangan keamanan tersendiri, seperti risiko dari *image* yang rentan atau miskonfigurasi jaringan, arsitektur *shared kernel* pada dasarnya dapat memicu kebocoran informasi jika tidak dikelola dengan benar (Gao et al., 2021). Namun, dengan mengikuti best practice, seperti menggunakan *official image*, tidak menjalankan kontainer sebagai *root*, dan mengelola *secrets* dengan benar, manfaat isolasi jauh lebih besar daripada risikonya untuk skala proyek kami.

#### **3. Instalasi dan Manajemen yang Disederhanakan 🛠️**

Proses instalasi menggunakan Docker dan Docker Compose sangat efisien. Dengan hanya satu file `docker-compose.yml`, kami dapat mendefinisikan dan menjalankan seluruh layanan Gitea beserta databasenya. Ini menyederhanakan proses yang biasanya bisa memakan waktu berjam-jam, seperti menginstal dependensi, mengkonfigurasi database, dan mengatur pengguna sistem secara manual.

Dengan menggunakan Docker pada Gitea, meskipun ada sedikit *overhead* pada performa I/O disk, manfaat dalam hal manajemen, portability, dan kemudahan *update* jauh lebih besar manfaatnya. Proses pembaruan Gitea menjadi sesederhana menarik *image* Docker versi terbaru dan menjalankan ulang kontainer, proses yang lebih bersih dan tidak rentan kesalahan dibandingkan pembaruan pada instalasi *native*.

#### **4. Antarmuka Familiar dan Intuitif 👨‍💻**

Antarmuka pengguna Gitea dibuat sangat mirip dengan GitHub. Bagi anggota tim kami yang sudah terbiasa dengan GitHub, transisi ke Gitea hampir tidak memerlukan learnig curve yang sulit. Tata letak untuk repositori, *pull request*, *issues*, dan pengaturan organisasi terasa sangat intuitif. Kemiripan ini mempercepat adopsi dan membuat tim dapat menggunakannya tanpa perlu pelatihan khusus terlebih dahulu. Faktor ini penting bagi tim kecil yang mengutamakan kecepatan dan agility dalam alur kerja pengembangan mereka.

![gitea benchmark](img/gitea_benchmark.png)

---
### **Kekurangan Gitea (Berdasarkan Pengalaman dan Analisis)**

Meskipun Gitea menawarkan banyak keunggulan, kami mengakui adanya *trade-off*. Filosofi desainnya yang ringan dan minimalis membawa beberapa keterbatasan jika dibandingkan dengan platform yang lebih besar.

#### **1. Beban Administratif dan Tanggung Jawab Keamanan Penuh**

Kebebasan dan kontrol penuh dari model *self-hosting* harus diikuti dnengan tanggung jawab yang besar. Dengan mengelola Gitea sendiri, tim kami juga berperan sebagai administrator sistem. Tanggung jawab ini mencakup seluruh siklus hidup pemeliharaan server, yang merupakan pekerjaan teknis yang terus menerus dan tidak bisa diabaikan.

* **Pemeliharaan Rutin:** Kami bertanggung jawab untuk melakukan pembaruan Gitea secara berkala, mem-patch sistem operasi, dan memastikan semua komponen (seperti Nginx dan Docker) tetap aman dan update.
* **Keamanan:** Gitea, seperti perangkat lunak lainnya, tidak luput dari penemuan celah keamanan (vulnerabilities). Pada tahun 2024-2025 saja, beberapa celah keamanan telah ditemukan, mulai dari *Cross-Site Scripting* (XSS) hingga isu pada implementasi SSH. Tanggung jawab untuk memantau pengumuman keamanan dan segera menerapkan *patch* berada di tangan kami sebagai administrator. Kelalaian dalam hal ini dapat membuka server terhadap risiko eksploitasi.
* **Strategi Backup dan Pemulihan:** Data adalah aset paling berharga. Kami harus merancang dan mengimplementasikan strategi *backup* yang reliable. Praktik terbaik untuk Gitea mengharuskan layanan dihentikan sementara saat proses *backup* untuk menjamin konsistensi data antara database dan repositori. Proses pemulihan (*restore*) juga merupakan prosedur yang rumit. Beban operasional ini ada pada layanan SaaS seperti GitHub atau GitLab.com, ditangani oleh penyedia layanan.

Beban ini seringkali ditanggung Usaha Kecil dan Menengah (UKM), yang biasanya memiliki sumber daya dan keahlian IT yang terbatas. Tantangan dalam mengelola skalabilitas dan sumber daya aplikasi pada infrastruktur *on-premises* dapat menjadi penghalang yang besar (Tonge et al., 2025).

#### **2. Keterbatasan Fitur Lanjutan dan Ekosistem**

Fokus Gitea pada efisiensi berarti Gitea tidak menawarkan rangkaian fitur DevOps terintegrasi selengkap GitLab. Bagi tim yang membutuhkan alur kerja DevOps yang canggih, ini bisa menjadi sebuah kekurangan.

* **CI/CD yang Lebih Sederhana:** Meskipun Gitea Actions dirancang agar kompatibel dengan GitHub Actions, terdapat beberapa keterbatasan teknis. Beberapa sintaks *workflow* penting seperti `concurrency` (untuk mengontrol eksekusi *job* secara bersamaan) dan `permissions` (untuk mengatur hak akses granular pada level *job*) belum didukung. Bagi tim yang sangat bergantung pada otomatisasi CI/CD yang kompleks, keterbatasan ini mungkin menjadi penghalang.
* **Fitur Enterprise Terbatas:** GitLab menawarkan fitur-fitur bawaan yang ditujukan untuk perusahaan besar, seperti *security scanning* (SAST, DAST), *compliance management*, dan *value stream analytics*. Fitur-fitur ini tidak ada di Gitea. Untuk mendapatkan fungsionalitas serupa, tim harus mengintegrasikan dan mengelola perangkat lunak pihak ketiga secara manual, yang menambah kompleksitas pada *tech stack*.
* **Ekosistem Plugin yang Masih Berkembang:** Ekosistem integrasi Gitea cukup matang untuk kebutuhan inti dan digerakkan oleh komunitas yang aktif. Namun, jumlahnya tidak sebanyak dan tidak seketat "marketplace" pada GitHub atau GitLab. Jika tim Anda bergantung pada integrasi yang sangat spesifik dengan perangkat lunak enterprise lain, kemungkinan besar Anda akan menemukan dukungan *out-of-the-box* yang lebih baik pada platform yang lebih besar.

Singkatnya, kekurangan Gitea disebabkan karena Gitea menempatkan lebih banyak tanggung jawab pada pengguna sebagai ganti dari kontrol dan efisiensi, serta fokus pada fitur inti *version control* daripada menyediakan platform DevOps yang serba ada.

![cloud vs selfhosted](img/cloud_vs_selfhosted.png)

---



## Referensi
* Tonge, A. S., Baniya, B. K., & GC, D. (2025). Efficient, scalable, and secure network monitoring platform: self-contained solution for future smes. Network, 5(3), 36. https://doi.org/10.3390/network5030036 
* Gao, X., Steenkamer, B. P., Gu, Z., Kayaalp, M., Pendarakis, D., & Wang, H. (2021). A study on the security implications of information leakages in container clouds. IEEE Transactions on Dependable and Secure Computing, 18(1), 174-191. https://doi.org/10.1109/tdsc.2018.2879605 
- Gitea Documentation - [https://docs.gitea.com/](https://docs.gitea.com/)
- Install Docker Engine on Ubuntu - [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
- nginx documentation - [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
