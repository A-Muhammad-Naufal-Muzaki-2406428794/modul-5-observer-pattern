# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

**1. Kebutuhan Interface (Trait) pada Observer Pattern di BambangShop**

Dalam kasus spesifik BambangShop saat ini, menggunakan **satu Model struct (`Subscriber`) saja sudah cukup**, dan kita tidak diwajibkan menggunakan Interface/Trait. 

Pada buku *Head First Design Pattern*, Observer (Subscriber) dibuat sebagai *interface* dengan asumsi bahwa di dalam satu aplikasi terdapat berbagai jenis *class* dengan implementasi yang berbeda-beda yang ingin menjadi *subscriber*. Agar Publisher bisa memanggil fungsi `update()` ke semua kelas tersebut, dibutuhkan sebuah *interface* sebagai kontrak.

Namun, dalam arsitektur BambangShop (Distributed Observer), *subscriber* kita bukanlah *class* internal, melainkan **aplikasi eksternal** yang berkomunikasi melalui HTTP Request. Selama aplikasi *receiver* mematuhi "kontrak" berupa struktur JSON (Model) dan URL *endpoint* (`/receive`) yang disepakati, Publisher tidak memedulikan detail implementasi aplikasi penerimanya. Oleh karena itu, mendefinisikan *payload* data (URL dan nama) ke dalam satu Model struct sudah sangat cukup.

**2. Penggunaan `Vec` (List) vs `DashMap` (Map/Dictionary) untuk Data Unik**

Menggunakan `DashMap` **jauh lebih efisien dan diperlukan** dibandingkan menggunakan `Vec` untuk kasus ini.

Atribut `url` pada *Subscriber* bertindak sebagai *Primary Key* (unik). 
* Jika menggunakan **`Vec` (List)**: Setiap kali kita ingin menghapus (*unsubscribe*) atau mengecek keberadaan seorang *subscriber*, program harus melakukan iterasi (Pencarian Linear dengan kompleksitas $O(N)$) satu per satu dari awal daftar. Ini akan menjadi *bottleneck* performa jika jumlah *subscriber* sudah mencapai ribuan.
* Jika menggunakan **`DashMap`**: Pencarian, penambahan, dan penghapusan data berdasarkan *Key* (`url`) dapat dilakukan secara langsung dengan kompleksitas waktu rata-rata $O(1)$. Selain itu, `DashMap` mencegah duplikasi *key* secara otomatis karena sifat dasarnya sebagai *Map/Dictionary*.

**3. `DashMap` vs Singleton Pattern untuk Keamanan Thread (Thread-Safety)**

Kita **tetap membutuhkan `DashMap`** (atau struktur data *thread-safe* sejenisnya), meskipun kita sudah menerapkan konsep *Singleton* menggunakan `lazy_static`.

*Singleton pattern* hanya menjamin bahwa **hanya ada satu instansiasi (objek) memori** dari daftar *subscribers* di seluruh siklus hidup aplikasi. Namun, *Singleton* **tidak** menjamin keamanan saat memori tersebut diakses atau dimodifikasi secara bersamaan (*Concurrency*).

Karena *framework* web (seperti Rocket) menangani banyak HTTP Request secara bersamaan di *thread* yang berbeda (*multithreading*), ada kemungkinan Request A mencoba menambahkan *subscriber* ke dalam *Map*, sementara Request B pada saat yang sama mencoba menghapus data dari *Map* tersebut. Jika kita menggunakan `HashMap` biasa dipadukan dengan *Singleton*, program Rust akan rentan terhadap *Data Race* dan menyebabkan aplikasi *crash* (*Panic*). Oleh karena itu, kita sangat membutuhkan `DashMap` karena ia memiliki mekanisme *Lock/Mutex* internal yang memastikan proses baca-tulis antar *thread* terjadi secara aman dan terkoordinasi.

#### Reflection Publisher-2

**1. Alasan Memisahkan "Service" dan "Repository" dari "Model"**

Meskipun pola dasar MVC (Model-View-Controller) menggabungkan penyimpanan data dan *business logic* ke dalam "Model", pendekatan arsitektur modern (seperti *Layered Architecture*) memisahkannya menjadi Model, Service, dan Repository dengan alasan utama **Single Responsibility Principle (SRP)** dan **Separation of Concerns**.

* **Model:** Hanya bertanggung jawab sebagai representasi struktur data (DTO/Data Transfer Object). Model murni hanya berisi atribut, tanpa tahu bagaimana data itu disimpan atau diproses.
* **Repository:** Hanya bertanggung jawab atas akses data ke memori atau *database* (CRUD). Jika suatu saat kita ingin mengganti `DashMap` dengan database sungguhan seperti PostgreSQL, kita hanya perlu mengubah kode di *Repository Layer* tanpa menyentuh *business logic*.
* **Service:** Hanya bertanggung jawab atas *business logic* (aturan bisnis), seperti validasi, manipulasi teks (misal: `to_uppercase()`), atau memanggil fungsi dari *Repository*.

Pemisahan ini membuat kode jauh lebih modular, lebih mudah dibaca, dan sangat mendukung *Test-Driven Development* (TDD) karena kita bisa melakukan *Mocking* pada Repository saat menguji Service.

**2. Dampak Jika Hanya Menggunakan Model (Tanpa Layering)**

Jika kita memaksakan semua logika (penyimpanan dan aturan bisnis) ke dalam Model, kita akan menciptakan **Spaghetti Code** dan **God Object Anti-Pattern**. Tingkat ketergantungan (*Coupling*) antar model akan menjadi sangat tinggi.

Bayangkan interaksinya: Model `Notification` harus membuat notifikasi baru. Jika tidak ada *Service/Repository*, model `Notification` harus menginisialisasi atau mengakses struktur data `DashMap` milik model `Subscriber` secara langsung untuk mencari siapa saja yang berlangganan `Product` tertentu. 
Akibatnya:
* Jika struktur penyimpanan di `Subscriber` berubah, kode di model `Notification` akan ikut rusak (*break*).
* Kode akan sangat sulit di-tes (karena *tight coupling*).
* Kompleksitas di setiap model akan membengkak karena satu *struct* harus mengurus validasi, HTTP request, penyimpanan data, dan relasi dengan entitas lain secara bersamaan.

**3. Pengalaman Menggunakan Postman**

Postman sangat membantu dalam menguji API (*backend*) secara mandiri tanpa harus menunggu aplikasi *frontend* atau *receiver* selesai dibangun. Postman bertindak sebagai klien simulasi yang bisa mengirim berbagai jenis *HTTP Request* (GET, POST, PUT, DELETE) dengan *header* dan *body* JSON yang bisa disesuaikan.

Beberapa fitur Postman yang menurut saya sangat menarik dan berguna untuk *Group Project* atau proyek *Software Engineering* ke depannya:
* **Collections & Sharing:** Memungkinkan tim *backend* untuk mengelompokkan semua *endpoint* API dan membagikannya ke tim *frontend* sebagai dokumentasi interaktif (seperti *collection* yang disediakan di tutorial ini).
* **Environment Variables:** Sangat berguna untuk menyimpan variabel seperti `{{base_url}}`. Kita bisa dengan mudah *switch* pengujian dari URL `localhost:8000` (saat *development*) ke URL *cloud/production* tanpa harus mengganti URL di setiap *request* satu per satu.
* **Automated Testing & Scripts:** Postman memiliki tab "Tests" di mana kita bisa menulis *script* JavaScript sederhana untuk memvalidasi *response* secara otomatis (misalnya memastikan status *code* 200 OK atau mengecek apakah JSON balikan memiliki atribut yang benar), yang sangat membantu proses *Quality Assurance* (QA).

#### Reflection Publisher-3

**1. Variasi Observer Pattern yang Digunakan (Push vs Pull)**

Pada tutorial kasus BambangShop ini, kita menggunakan variasi **Push Model**. 
Hal ini terlihat jelas pada fungsi `update` di Model Subscriber dan fungsi `notify` di NotificationService. Ketika terjadi sebuah *event* (misal: produk dibuat), aplikasi Publisher secara aktif "mendorong" (mengirimkan) keseluruhan data notifikasi (*payload* JSON) ke URL masing-masing Subscriber melalui HTTP POST request. Subscriber bersifat pasif dan hanya menunggu data tersebut datang.

**2. Keuntungan dan Kerugian Jika Menggunakan Pull Model**

Jika kita membalik logikanya dan menggunakan **Pull Model** (Publisher hanya memberi tahu "Ada perubahan produk lho!", lalu Subscriber harus melakukan HTTP GET untuk mengambil detail datanya), berikut adalah analisisnya:

* **Keuntungan Pull Model:**
  * **Ukuran Payload Lebih Kecil (Efisien bandwidth bagi Publisher):** Publisher tidak perlu memaketkan data yang besar ke semua subscriber. Ia hanya mengirimkan sinyal atau ID produk yang berubah.
  * **Kendali di Tangan Subscriber:** Subscriber bisa memilih kapan ia ingin mengambil (pull) data tersebut, atau bahkan mengabaikannya jika sedang sibuk/down. Subscriber juga bisa memfilter spesifik data apa yang ingin dia *request*.
* **Kerugian Pull Model:**
  * **Latensi Lebih Tinggi:** Proses menjadi dua kali kerja (*round-trip*). Publisher mengirim sinyal -> Subscriber memproses sinyal -> Subscriber melakukan request HTTP ke Publisher -> Publisher membalas dengan data.
  * **Beban Server Publisher Bisa Melonjak:** Jika Publisher mengumumkan pembaruan ke 1.000 subscriber, sepersekian detik kemudian server Publisher akan langsung dihantam oleh 1.000 request HTTP GET masuk secara bersamaan dari para subscriber yang berebut ingin mengambil data.

**3. Dampak Jika Tidak Menggunakan Multi-threading dalam Proses Notifikasi**

Jika kita menghapus `thread::spawn` dan melakukan pengiriman notifikasi secara sekuensial (sinkron/satu per satu) di *thread* utama, aplikasi akan mengalami **I/O Blocking** dan performanya akan sangat buruk.

* **Proses Menjadi Sangat Lambat:** Mengirim HTTP request melalui internet membutuhkan waktu (misal 100ms hingga beberapa detik). Jika ada 1.000 subscriber, aplikasi akan menunggu request pertama selesai sebelum mengirim yang kedua. Ini bisa memakan waktu bermenit-menit hanya untuk menyelesaikan satu fungsi `notify()`.
* **API Hang / Timeout:** Karena fungsi `notify()` dipanggil di dalam proses *create*, *delete*, atau *publish* produk, lambatnya pengiriman notifikasi akan menahan *return response* API. Akibatnya, saat Pak Bambang menekan tombol "Create Product" di aplikasi, layarnya akan *loading* terus-menerus (*freeze*) sampai semua notifikasi selesai dikirim.
* **Satu Error Menghambat Semua:** Jika URL milik Subscriber pertama sudah mati atau lambat merespons (*timeout*), antrean pengiriman ke 999 Subscriber lainnya akan ikut tertahan.