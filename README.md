a. How much data will your publisher program send to the message broker in one run?
Program ini akan mengirim lima buah pesan ke message broker. Setiap pesan adalah sebuah objek UserCreatedEventMessage yang terdiri dari dua string: user_id dan user_name.

Masing-masing pesan memiliki ukuran data berbeda tergantung pada panjang string-nya. Kita bisa kira-kira menghitung total data sebagai berikut:

Setiap user_id memiliki panjang 1 karakter (misal "1"), dan user_name panjangnya sekitar 18 karakter (contoh: "2306217481-Amir").

Dalam format Borsh (yang digunakan untuk serialisasi), setiap string diserialisasi sebagai:
4 byte (panjang string, u32) + isi string (dalam byte, biasanya 1 byte per karakter UTF-8 ASCII).

Jadi perkiraan untuk satu pesan:

user_id: 4 + 1 = 5 byte

user_name: 4 + 18 = 22 byte

Total per pesan: 5 + 22 = 27 byte

Dengan 5 pesan:

5 × 27 byte = 135 byte total data dikirim ke broker dalam satu kali eksekusi.

Jadi sekitar 135 byte data dikirim oleh program publisher ini ke message broker setiap kali dijalankan.

b. The URL amqp://guest:guest@localhost:5672 is the same as in the subscriber program. What does it mean?
Ya, URL tersebut sama persis antara program publisher dan subscriber, dan ini memiliki makna penting.

amqp:// menunjukkan bahwa protokol yang digunakan adalah AMQP (Advanced Message Queuing Protocol).

guest:guest adalah kombinasi username dan password yang digunakan untuk autentikasi ke message broker.

guest (pertama): username

guest (kedua): password

localhost menunjukkan bahwa broker (misalnya RabbitMQ) berjalan secara lokal di mesin yang sama.

5672 adalah port default untuk koneksi AMQP.

Karena publisher dan subscriber menggunakan URL yang sama, ini berarti:

Keduanya terhubung ke message broker yang sama, yaitu RabbitMQ lokal.

Publisher mengirim pesan ke queue "user_created", dan subscriber (listener) mendengarkan queue yang sama.

Ini memungkinkan terjadinya pertukaran pesan langsung: pesan yang dipublish oleh publisher akan diterima oleh subscriber secara real-time (asinkron).

Running RabbitMQ:
![Image](https://github.com/user-attachments/assets/9214da05-4f1f-4a5d-92b8-f2785823e32c)

Screen capture showing consoles:
![Image](https://github.com/user-attachments/assets/0feb6463-6a6c-4c70-b9db-076e8b7133ce)

![Image](https://github.com/user-attachments/assets/8a2f54f7-3a01-4ab0-9858-97c9e50d1e0d)
Spike atau lonjakan yang terlihat pada grafik kedua ("Message rates") di RabbitMQ Management UI terjadi karena program publisher dijalankan. Dalam program tersebut, terdapat lima pesan yang dikirimkan ke broker RabbitMQ secara berurutan namun sangat cepat. Pesan-pesan ini dipublikasikan ke antrian dengan routing key "user_created" menggunakan protokol AMQP. Karena kelima pesan ini dikirim dalam waktu yang sangat singkat (hanya dalam satu eksekusi program), RabbitMQ mencatat peningkatan mendadak pada laju pengiriman pesan (publish rate). Hal inilah yang menyebabkan munculnya spike atau puncak tajam dalam grafik publish. Setelah pesan selesai dikirim, laju publish kembali turun ke nol, sehingga grafik kembali datar. Spike ini bisa muncul kembali setiap kali publisher dijalankan ulang karena pola pengiriman pesan yang cepat dan serempak tersebut.
