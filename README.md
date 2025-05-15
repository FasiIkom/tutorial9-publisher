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