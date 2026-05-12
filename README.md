# Publisher — Modul 9 Software Architectures

Repositori ini berisi kode **Publisher** untuk Tutorial A Modul 9 — Event-Driven Architecture menggunakan Rust dan RabbitMQ sebagai message broker.

---

## a. How Much Data Will Your Publisher Send to the Message Broker in One Run?

Dalam satu kali eksekusi (`cargo run`), publisher mengirimkan **5 event** ke message broker. Setiap event berupa objek `UserCreatedEventMessage` yang memiliki dua field, yaitu `user_id` (bertipe String) dan `user_name` (bertipe String). Kelima event tersebut masing-masing merepresentasikan pengguna berbeda: Amir, Budi, Cica, Dira, dan Emir. Semua pesan dikirim ke queue yang bernama `user_created` secara berurutan dalam satu sesi koneksi ke broker. Dengan demikian, total data yang dikirim dalam satu run adalah **5 objek `UserCreatedEventMessage`** yang masing-masing berisi pasangan `user_id` dan `user_name`.

---

## b. The URL `amqp://guest:guest@localhost:5672` Is the Same as in Subscriber. What Does It Mean?

URL `amqp://guest:guest@localhost:5672` adalah **connection string** yang digunakan untuk terhubung ke RabbitMQ. Bagian `guest` pertama adalah **username** dan `guest` kedua adalah **password** untuk autentikasi ke RabbitMQ. Sementara itu, `localhost` menunjukkan bahwa RabbitMQ berjalan di mesin yang sama, dan `5672` adalah **port default** protokol AMQP.

Fakta bahwa publisher dan subscriber menggunakan URL yang **sama persis** mengandung makna penting dalam event-driven architecture: keduanya terhubung ke **message broker yang sama**. Publisher tidak perlu tahu siapa yang akan membaca pesannya, dan subscriber tidak perlu tahu siapa yang mengirim. Mereka hanya perlu sepakat pada satu hal — broker mana yang digunakan. Inilah yang membuat arsitektur ini bersifat **loosely coupled**: publisher dan subscriber bisa berjalan secara independen, bahkan di waktu yang berbeda, selama keduanya terhubung ke broker yang sama.

---

## RabbitMQ Dashboard

Berikut adalah tampilan dashboard RabbitMQ yang berjalan di `localhost:15672`:

![RabbitMQ Dashboard](screenshot_rabbitmq.png)

RabbitMQ berhasil dijalankan menggunakan Docker dengan perintah `docker run`. Dashboard menampilkan informasi real-time mengenai status **connections**, **channels**, **exchanges**, dan **queues** yang sedang aktif. Dari dashboard ini kita bisa memantau jumlah pesan yang masuk, keluar, maupun yang masih mengantri di setiap queue. Dashboard ini sangat berguna untuk debugging dan monitoring performa message broker dalam sistem event-driven.

---

## Sending and Processing Event

![Screenshot Subscriber Terminal](screenshot_run_subs.png)
![Screenshot Publisher Terminal](screenshot_run_pub.png)

Ketika publisher dijalankan dengan `cargo run`, publisher langsung mengirimkan 5 event bertipe `UserCreatedEventMessage` ke message broker RabbitMQ melalui queue bernama `user_created`. Proses pengiriman berlangsung sangat cepat karena publisher tidak menunggu konfirmasi dari subscriber, ia cukup memastikan pesan sampai ke broker.

Pada sisi subscriber yang sedang berjalan di terminal lain, kelima event tersebut diterima secara otomatis dan diproses satu per satu. Setiap pesan yang diterima ditampilkan di konsol lengkap dengan isi `user_id` dan `user_name`-nya. Hal ini membuktikan bahwa komunikasi antara publisher dan subscriber berhasil dilakukan secara tidak langsung melalui RabbitMQ sebagai perantara, tanpa publisher dan subscriber perlu saling mengenal satu sama lain secara langsung.

---

## Monitoring Chart Based on Publisher

![RabbitMQ Chart](screenshot_chart.png)

Setelah publisher dijalankan beberapa kali secara berturut-turut, grafik **message rate** di RabbitMQ dashboard menunjukkan adanya **spike** (lonjakan tajam) setiap kali publisher dieksekusi. Lonjakan ini terjadi karena dalam waktu sangat singkat, 5 pesan langsung dikirim sekaligus ke broker sehingga message rate melonjak drastis sesaat sebelum kembali ke nol setelah subscriber selesai memproses semua pesan.

Semakin sering publisher dijalankan, semakin banyak spike yang terlihat pada grafik. Pola ini mencerminkan bagaimana sistem event-driven bekerja: publisher bebas mengirim kapan saja tanpa khawatir apakah subscriber sedang siap atau tidak, karena pesan akan tersimpan aman di antrian broker hingga subscriber mengambilnya.

---

# Bonus: Running on Cloud (CloudAMQP)

Sebagai bonus, seluruh eksperimen pada tutorial ini dijalankan ulang menggunakan **CloudAMQP** sebagai message broker berbasis cloud, menggantikan RabbitMQ yang sebelumnya berjalan di local machine via Docker.

---

### Apa itu CloudAMQP?

CloudAMQP adalah layanan **RabbitMQ as a Service** yang di-host di cloud. Dengan menggunakan CloudAMQP, publisher dan subscriber tidak lagi bergantung pada instance RabbitMQ yang harus dijalankan secara manual di local machine. Sebaliknya, keduanya terhubung ke broker yang sudah berjalan di cloud dan dapat diakses dari mana saja melalui internet menggunakan URL koneksi AMQP yang diberikan oleh CloudAMQP. Layanan ini tersedia secara gratis pada plan **"Little Lemur"** yang cukup untuk keperluan eksperimen dan pembelajaran.

---

### Perubahan yang Dilakukan

URL koneksi AMQP pada publisher diubah dari `localhost` ke URL yang disediakan oleh CloudAMQP:

```
// Sebelum (lokal):
amqp://guest:guest@localhost:5672

// Sesudah (cloud):
amqp://username:password@broker.cloudamqp.com/username
```

Perubahan ini hanya dilakukan pada satu baris di `src/main.rs`, membuktikan betapa mudahnya berpindah dari broker lokal ke broker cloud tanpa mengubah logika program sama sekali.

---

### RabbitMQ Dashboard (CloudAMQP)

![RabbitMQ Dashboard CloudAMQP](image-1.png)

Dashboard RabbitMQ yang diakses melalui portal CloudAMQP menampilkan informasi yang sama seperti dashboard lokal, namun kini broker berjalan sepenuhnya di cloud. Terlihat koneksi aktif dari publisher yang berjalan di GitHub Codespaces, membuktikan bahwa komunikasi lintas internet berhasil terjalin.

---

### Sending and Processing Event (Cloud)

![Screenshot Subscriber Terminal Cloud](image-3.png)
![Screenshot Publisher Terminal Cloud](image-4.png)

Setelah URL diganti ke CloudAMQP, publisher berhasil mengirimkan 5 event ke broker di cloud dan subscriber yang terhubung ke broker yang sama berhasil menerima serta memproses semua event tersebut. Ini membuktikan bahwa **event-driven architecture bekerja lintas mesin dan lintas jaringan** — publisher dan subscriber tidak harus berada di mesin yang sama, cukup terhubung ke broker yang sama di cloud.

---

### Monitoring Chart Based on Publisher (Cloud)

![RabbitMQ Chart CloudAMQP](image-5.png)

Grafik message rate di CloudAMQP dashboard juga menunjukkan pola spike yang sama seperti saat menggunakan broker lokal. Setiap kali publisher dijalankan, terjadi lonjakan singkat pada message rate yang kemudian kembali normal setelah subscriber selesai memproses semua pesan. Hal ini membuktikan bahwa perilaku sistem tidak berubah meskipun broker dipindahkan ke cloud.
