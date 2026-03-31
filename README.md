# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

# 1. In this tutorial, we used RwLock<> to synchronise the use of Vec of Notifications. Explain why it is necessary for this case, and explain why we do not use Mutex<> instead?

- Mengapa diperlukan: Penggunaan lock diperlukan untuk memastikan thread-safety. Di dalam web server (seperti Rocket), banyak request bisa datang dan diproses secara bersamaan (multi-threading). Jika beberapa thread mencoba membaca dan menulis ke Vector NOTIFICATIONS secara bersamaan tanpa sinkronisasi, akan terjadi data race atau memori korup.

- Mengapa RwLock<> dan bukan Mutex<>: Mutex (Mutual Exclusion) sangat ketat; ia hanya mengizinkan satu thread (baik membaca atau menulis) untuk mengakses data pada satu waktu. Sementara itu, RwLock (Reader-Writer Lock) mengizinkan banyak thread pembaca (reader) untuk mengakses data secara bersamaan selama tidak ada proses penulisan, tetapi jika ada proses penulisan, ia akan mengunci data secara eksklusif untuk satu writer saja. Dalam kasus notifikasi ini, kita berasumsi akan ada jauh lebih banyak proses read (melihat daftar notifikasi) daripada write (menambah notifikasi baru). Oleh karena itu, RwLock memberikan performa yang jauh lebih efisien untuk skenario read-heavy tanpa memblokir pembaca lain seperti yang dilakukan Mutex.

# 2. In this tutorial, we used lazy_static external library to define Vec and DashMap as a "static" variable. Compared to Java where we can mutate the content of a static variable via a static function, why did not Rust allow us to do so?
Rust memiliki aturan kepemilikan (ownership) dan peminjaman (borrowing) yang sangat ketat untuk menjamin keamanan memori (memory safety) pada saat proses kompilasi (compile-time).

- Di Java, kita bisa bebas memodifikasi variabel static global secara langsung dari banyak thread, yang sering kali memicu bug konkurensi (data race) jika programmer lupa memasang synchronized.

- Di Rust, variabel global/static yang bisa diubah (mutable) dianggap pada dasarnya tidak aman (inherently unsafe) karena sangat rawan data race di lingkungan multi-threaded. Rust akan memaksa kita membungkus operasi mutasi global statis di dalam blok unsafe {}, yang sangat dihindari.

- Untuk menyelesaikannya secara elegan dan safe, kita menggunakan lazy_static! untuk menunda inisialisasi pada saat program berjalan (karena tipe dinamis seperti Vec butuh memori heap alih-alih konstan saat kompilasi), lalu digabungkan dengan struktur data thread-safe seperti RwLock (atau DashMap di bagian sebelumnya) untuk meyakinkan compiler bahwa kita mengakses data global tersebut secara aman tanpa melanggar aturan sinkronisasi Rust.

#### Reflection Subscriber-2

# 1. Have you explored things outside of the steps in the tutorial, for example: src/lib.rs? If not,explain why you did not do so. If yes, explain things that you have learned from those other parts of code.

Saya menelusuri file src/lib.rs untuk memahami bagaimana aplikasi mengelola konfigurasi global serta klien HTTP menggunakan lazy_static!. Saya mempelajari bahwa APP_CONFIG dan REQWEST_CLIENT diinisialisasi satu kali agar dapat digunakan secara konsisten di seluruh layanan. Selain itu, saya juga memahami bagaimana penanganan error kustom diimplementasikan melalui fungsi compose_error_response, sehingga pesan error yang dikembalikan melalui Rocket menjadi lebih konsisten dan informatif.

# 2. Since you have completed the tutorial by now and have tried to test your notification system by spawning multiple instances of Receiver, explain how Observer pattern eases you to plug in more subscribers. How about spawning more than one instance of Main app, will it still be easy enough to add to the system?

Observer pattern mempermudah penambahan subscriber karena hubungan antara Publisher dan Receiver dibuat longgar (loose coupling). Publisher hanya bertugas menyimpan daftar subscriber dan mengirimkan notifikasi tanpa perlu mengetahui bagaimana masing-masing Receiver memproses data tersebut. Dengan begitu, menambahkan subscriber baru cukup dengan melakukan registrasi tanpa harus mengubah logika di sisi Publisher.

Namun, ketika terdapat lebih dari satu instance Main app (Publisher), setiap instance akan berjalan secara independen dan memiliki daftar subscriber masing-masing. Hal ini berarti penambahan subscriber masih tetap mudah, tetapi jika sebuah Receiver ingin menerima notifikasi dari semua Publisher, maka ia harus mendaftar ke setiap instance secara terpisah. Untuk skala yang lebih besar, biasanya diperlukan mekanisme tambahan seperti shared storage atau message broker agar manajemen subscriber bisa terpusat.

# 3. Have you tried to make your own Tests, or enhance documentation on your Postman collection? If you have tried those features, tell us whether it is useful for your work (it can be your tutorial work or your Group Project).

Saya menjalankan beberapa instance Receiver pada port 8001, 8002, dan 8003, lalu menggunakan Postman collection yang disediakan untuk melakukan subscribe ke berbagai product type. Selain itu, saya juga mencoba menambahkan request dan test sederhana di Postman untuk memastikan setiap endpoint (subscribe, publish, dan delete) memberikan respons yang sesuai.

Fitur dokumentasi dan collection di Postman sangat membantu karena semua request dapat disimpan dan dijalankan ulang dengan mudah tanpa harus mengetik ulang. Dalam pengerjaan tutorial maupun proyek kelompok, hal ini mempermudah pengujian beberapa instance Receiver sekaligus serta memastikan setiap notifikasi benar-benar terkirim ke subscriber yang tepat. Selain itu, proses debugging juga menjadi lebih cepat karena kita bisa langsung melihat hasil response dari tiap request secara terstruktur.
