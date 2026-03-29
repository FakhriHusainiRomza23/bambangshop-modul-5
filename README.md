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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Dalam konsep Observer Pattern, penggunaan interface atau trait sebenarnya bertujuan agar Subject (Publisher) dapat berkomunikasi dengan berbagai jenis Observer yang memiliki perilaku berbeda secara polimorfik. Namun, dalam kasus BambangShop saat ini, penggunaan sebuah Single Model Struct sudah dianggap cukup karena semua subscriber memiliki struktur data dan mekanisme komunikasi yang seragam, yaitu melalui pemanggilan HTTP endpoint yang tersimpan pada field url. Kita baru akan benar-benar membutuhkan trait jika di masa depan muncul kebutuhan untuk mendukung berbagai jenis subscriber dengan metode notifikasi yang berbeda-beda, misalnya satu subscriber melalui protokol HTTP, sementara yang lain melalui Email atau SMS, di mana masing-masing membutuhkan logika pengiriman yang spesifik.

2. Penggunaan DashMap jauh lebih diperlukan dibandingkan Vec (list) biasa dalam kasus ini karena efisiensi dan integritas data yang ditawarkannya. Mengingat id pada produk dan url pada subscriber dimaksudkan untuk menjadi pengenal unik, DashMap memungkinkan kita untuk melakukan operasi pencarian, penambahan, dan penghapusan data dengan kompleksitas waktu rata-rata O(1). Jika kita menggunakan Vec, setiap kali aplikasi ingin memastikan keunikan data atau menghapus subscriber tertentu, kita harus melakukan iterasi ke seluruh elemen (O(n)) yang akan memperlambat performa seiring bertambahnya jumlah data. Selain itu, DashMap secara otomatis menangani pemetaan kunci unik sehingga risiko terjadinya duplikasi data dapat diminimalisasi dibandingkan dengan mengelola logika manual pada sebuah list.

3. Berdasarkan pemahaman design pattern, DashMap dan Singleton bukanlah dua hal yang saling meniadakan, melainkan bekerja secara berdampingan dalam arsitektur aplikasi Rust ini. Penggunaan variabel statis SUBSCRIBERS yang dibungkus dengan lazy_static! pada dasarnya sudah merupakan implementasi dari Singleton Pattern, di mana kita memastikan hanya ada satu instance penyimpanan data di seluruh siklus hidup aplikasi. Namun, karena Rust memiliki aturan memory safety dan concurrency yang sangat ketat, Singleton saja tidak cukup untuk menangani akses dari banyak thread secara bersamaan. Di sinilah DashMap berperan sebagai thread-safe HashMap yang menyediakan mekanisme penguncian (locking) internal, sehingga kita bisa memiliki satu instance data (Singleton) yang aman untuk dibaca dan ditulis oleh banyak proses sekaligus tanpa menyebabkan data race atau crash.

#### Reflection Publisher-2
1. Pemisahan antara Service dan Repository dari Model didasarkan pada prinsip Single Responsibility Principle (SRP) dan Separation of Concerns. Dalam arsitektur MVC klasik, Model seringkali menjadi terlalu besar (fat model) karena harus menangani struktur data, validasi, logika bisnis, hingga akses ke penyimpanan data. Dengan memisahkan Repository, kita mengisolasi logika akses data (seperti kueri database atau manipulasi DashMap), sehingga jika cara penyimpanan data berubah, bagian lain tidak terdampak. Sementara itu, Service berfungsi untuk membungkus logika bisnis yang kompleks (seperti orkestrasi pengiriman notifikasi ke banyak subscriber), sehingga Model tetap bersih dan hanya fokus pada representasi data atau objek itu sendiri.

2. Jika kita hanya mengandalkan Model tanpa lapisan Service dan Repository, kode akan mengalami ketergantungan antar-objek yang sangat tinggi (high coupling) dan sulit dikelola. Bayangkan jika Model Product harus tahu cara memanggil Model Subscriber untuk mengambil daftar URL, lalu Model Product juga harus mengandung logika pengiriman Notification melalui HTTP client. Hal ini membuat Model Product tidak lagi hanya mewakili data produk, tapi juga merangkap sebagai pengirim pesan dan pengelola daftar langganan. Interaksi yang saling tumpang tindih ini akan meningkatkan kompleksitas kode secara drastis, membuat unit testing menjadi sangat sulit karena setiap Model terikat erat dengan fungsionalitas Model lainnya.

3. Postman sangat membantu dalam menguji aplikasi backend berbasis API seperti BambangShop karena memungkinkan kita mensimulasikan berbagai request HTTP (GET, POST, DELETE) tanpa harus membuat antarmuka pengguna (UI) terlebih dahulu. Dalam proyek ini, Postman membantu saya memastikan bahwa endpoint /subscribe dan /unsubscribe berjalan dengan benar serta mengembalikan kode status HTTP yang sesuai. Fitur yang paling menarik bagi saya adalah Collections, yang memungkinkan pengelompokan request sehingga pengujian bisa dilakukan secara terorganisir. Selain itu, fitur Environment Variables sangat berguna untuk menyimpan URL server (seperti localhost:8000) agar kita tidak perlu mengetik alamat lengkap secara berulang-ulang di setiap request. 


#### Reflection Publisher-3
1. Dalam implementasi BambangShop ini, kita menggunakan variasi Push Model. Hal ini terlihat jelas dari logika pada NotificationService dan Subscriber, di mana pihak Publisher secara aktif mengirimkan data (payload) yang lengkap langsung ke setiap subscriber segera setelah terjadi perubahan. Subscriber dalam model ini bersifat pasif dalam hal penerimaan data; mereka tidak perlu melakukan kueri atau meminta pembaruan secara manual, melainkan cukup menyediakan endpoint untuk menerima kiriman data yang dipicu oleh aksi di sisi Publisher.

2. Jika kita menggunakan Pull Model, Publisher hanya akan mengirimkan notifikasi pendek atau sinyal perubahan (misalnya hanya mengirimkan ID produk yang baru dibuat), dan kemudian subscriber yang akan memutuskan kapan mereka akan mengambil detail data tersebut melalui request tambahan. Keuntungan utama dari model ini adalah efisiensi transmisi data jika subscriber tidak selalu membutuhkan detail lengkap untuk setiap perubahan, serta memberikan kontrol lebih kepada subscriber agar tidak kewalahan menerima kiriman data saat sedang sibuk. Namun, kekurangannya adalah munculnya latensi tambahan karena harus terjadi komunikasi dua arah (notifikasi diikuti oleh request balik) serta beban kerja Publisher yang meningkat karena harus menyediakan endpoint API ekstra untuk melayani permintaan data dari para subscriber.

3. Apabila kita memutuskan untuk tidak menggunakan multi-threading (misalnya dengan menghapus thread::spawn), maka proses pengiriman notifikasi akan berjalan secara sinkronus atau blocking. Hal ini akan menyebabkan performa aplikasi menurun drastis karena server harus menunggu setiap pemanggilan HTTP ke subscriber selesai satu per satu sebelum bisa memberikan respon kembali ke pengguna. Jika salah satu subscriber memiliki koneksi yang lambat atau sedang mengalami gangguan, seluruh sistem BambangShop akan ikut tertahan (hang), sehingga pengguna yang sedang membuat atau menghapus produk akan merasakan delay yang sangat lama meskipun operasi pada database sebenarnya sudah selesai dilakukan.