Nama : Yuki Nagano Mondjompi

NIM  : 235410045

Kelas: IF-1


3.1 SINGKRONISASI WAKTU > MENGGUNAKAN WINDOWS.
<img width="506" height="365" alt="gambar" src="https://github.com/user-attachments/assets/562c3a7d-ed76-4dab-bae3-8e17a39ea79d" />
1. Menghubungi NTP Server: Saat NetTime dijalankan, aplikasi akan mengirim permintaan ke server NTP (misalnya: pool.ntp.org). Server ini berfungsi sebagai sumber waktu yang akurat (biasanya terhubung dengan jam atom).
2. Pertukaran Timestamp: Terjadi pertukaran data waktu antara komputer dan server: waktu saat permintaan dikirim, waktu saat diterima server, waktu saat dikirim balik oleh server, dan waktu saat diterima kembali oleh komputer.
3. Menghitung Delay dan Offset: NetTime menghitung: Delay → waktu perjalanan data (network latency), Offset → selisih waktu antara komputer dan server.
4. Menentukan Koreksi Waktu: Berdasarkan hasil perhitungan, sistem menentukan apakah waktu komputer terlalu cepat atau terlalu lambat.
5. Penyesuaian Waktu Sistem: Jika selisih kecil → waktu disesuaikan secara perlahan (smooth adjustment), Jika selisih besar → waktu bisa langsung diubah (step adjustment).
6. Sinkronisasi Berkala (Periodic Sync): NetTime akan mengulangi proses ini secara otomatis dalam interval tertentu agar waktu tetap akurat.



3.2 VECTOR CLOCK.

<img width="714" height="304" alt="gambar" src="https://github.com/user-attachments/assets/e9c9702c-4140-4dac-9c7e-0663b445d0a5" />

1. Keluaran program menunjukkan bagaimana vector clock melacak urutan kejadian (causality) antar proses secara otomatis: awalnya semua proses memiliki waktu [0,0,0], lalu saat setiap proses melakukan event lokal atau mengirim/menerima pesan, nilainya bertambah dan diperbarui sesuai aturan (increment dan merge menggunakan nilai maksimum). Misalnya saat P0 mengirim ke P1, nilai clock P1 akan “menyerap” informasi dari P0 sehingga mencerminkan bahwa event di P0 terjadi sebelum P1. Hasil akhir menunjukkan bahwa VC0 happens before VC1 bernilai true, sedangkan sebaliknya false, yang berarti urutan kejadian terdeteksi dengan benar; sementara VC0 dan VC2 bisa jadi concurrent jika tidak ada hubungan langsung. Jika dibandingkan dengan perhitungan manual, hasilnya sama, tetapi program ini mengotomatiskan proses update (increment, penggabungan max, dan pengecekan relasi), sehingga lebih cepat, minim kesalahan manusia, dan lebih mudah digunakan untuk sistem yang kompleks dibanding menghitung vector clock satu per satu secara manual.
2. a. modul Python untuk class VectorClock: Simpan kode berikut dalam file bernama vector_clock.py:

   <img width="519" height="553" alt="gambar" src="https://github.com/user-attachments/assets/91fe551f-8a87-47ee-9b6b-8d4a5c783253" />

   b. contoh cara menggunakan modul tersebut: Buat file baru bernama main.py, isikan kode berikut kedalam file tsb.

   <img width="413" height="587" alt="gambar" src="https://github.com/user-attachments/assets/02d6a46e-7497-43e6-8429-0c52f76ea5ca" />

   c. Cara menjalankan: pastikan kedua file dalam folder yang sama, lalu jalankan: python main.py. Contoh output:

   <img width="779" height="447" alt="gambar" src="https://github.com/user-attachments/assets/85074253-3c74-40d5-8a05-35baeea9d1e6" />


3.3 PROBLEM TANPA SINKRONISASI
1. Jalankan program tersebut sampai anda mendapatkan keluaran yang berbeda.

   <img width="795" height="231" alt="gambar" src="https://github.com/user-attachments/assets/733e7a3e-7622-413c-b72a-703bc0d53029" />

2. Capture keluaran program tersebut dan jelaskan mengapa bisa berbeda.

   <img width="793" height="221" alt="gambar" src="https://github.com/user-attachments/assets/077ffce0-8d8a-438d-8a67-368eed1220b3" />

   penjelasan kenapa bisa berbeda:

   Keluaran program bisa berbeda-beda setiap kali dijalankan karena menggunakan multithreading, di mana beberapa thread berjalan secara bersamaan (concurrent) dan dijadwalkan oleh sistem operasi secara dinamis. Urutan eksekusi thread tidak dijamin selalu sama karena dipengaruhi oleh faktor seperti scheduler CPU, beban sistem, dan timing eksekusi masing-masing thread. Meskipun semua task memiliki delay yang sama (sleep 5 detik), waktu mulai dan selesai tiap thread bisa sedikit berbeda, sehingga urutan pesan “starting” dan “finished” yang muncul di output dapat berubah-ubah di setiap eksekusi.

3.3.1 Data Race / Race Conditions
1. Jalankan program tersebut. (race-conditions-01.py)

   <img width="694" height="116" alt="gambar" src="https://github.com/user-attachments/assets/49ea5182-1244-4366-98cd-50722ac868b0" />

2. jelaskan menggunakan visualisasi (gambar dengan pensil/ballpoint kemudian difoto), mengapa terjadi data race /race conditions.

   <img width="1239" height="1500" alt="gambar" src="https://github.com/user-attachments/assets/0a041251-d28a-4476-8a9d-bf9b8c2c8a8f" />

Berikut adalah contoh program untuk membuat supaya race conditions tersebut tidak terjadi(race-conditions-02.py):
1. jalankan program tersebut

   <img width="677" height="118" alt="gambar" src="https://github.com/user-attachments/assets/0d58349a-8f01-421e-b05e-4b37efb507c8" />

2. Amati keluarannya. Jelaskan mengapa race-conditions tidak terjadi?

   Pada program ini race condition tidak terjadi karena digunakan mekanisme lock (threading.Lock) yang membatasi akses ke variabel bersama (balance). Saat satu thread masuk ke blok with counter_lock:, thread tersebut “mengunci” resource sehingga thread lain harus menunggu sampai lock dilepas. Artinya, proses cek saldo (if balance >= amount), delay (sleep), dan pengurangan saldo (balance -= amount) dilakukan secara atomik (tidak terinterupsi) oleh thread lain. Akibatnya, hanya satu thread yang bisa memproses penarikan pada satu waktu; jika thread pertama sudah mengurangi saldo (misalnya dari 100 jadi 20), maka thread kedua saat masuk akan melihat saldo sudah tidak cukup dan tidak melakukan penarikan. Inilah yang mencegah kondisi balapan (race condition) yang sebelumnya bisa terjadi ketika dua thread membaca dan memodifikasi saldo secara bersamaan tanpa sinkronisasi.

3.3.2 Deadlock

1. Jalankan program tersebut.
   
   <img width="626" height="145" alt="gambar" src="https://github.com/user-attachments/assets/96b0e727-d6a7-4c28-94c0-7c1c370eabfd" />
  
2. Jelaskan menggunakan visualisasi (gambar dengan pensil/ballpoint kemudian difoto), mengapa terjadi deadlock

   <img width="1259" height="1500" alt="gambar" src="https://github.com/user-attachments/assets/44fc0669-fe93-47de-8a17-37b3d24ca101" />

Berikut adalah contoh program untuk membuat supaya deadlock tersebut tidak terjadi (deadlock-02.py)

1. Jalankan program tersebut.

   <img width="627" height="167" alt="gambar" src="https://github.com/user-attachments/assets/6def72af-dcdb-477a-b9ac-5120bffb1cc7" />

2. Amati keluarannya. Jelaskan mengapa deadlock tidak terjadi?

   Deadlock tidak terjadi pada program ini karena setiap thread tidak menahan dua lock secara bersamaan dan menggunakan mekanisme timeout    saat acquire lock. Setelah thread berhasil mendapatkan lock pertama (misalnya lock_a pada Thread 1), lock tersebut akan langsung          dilepas (release) sebelum mencoba mengambil lock kedua (lock_b), sehingga tidak ada kondisi saling menunggu (circular wait). Selain       itu, penggunaan acquire(timeout=5) memastikan bahwa jika suatu lock tidak bisa didapatkan dalam waktu tertentu, thread tidak akan         menunggu selamanya. Kombinasi pelepasan lock lebih awal dan adanya timeout inilah yang mencegah kondisi deadlock, karena resource         tidak saling “dikunci dan ditunggu” secara bersamaan oleh kedua thread.

3.4 ALGORITMA RAFT

1. Jalankan program tersebut.(raft.py)

   <img width="888" height="615" alt="gambar" src="https://github.com/user-attachments/assets/1327dcd7-2378-431b-bb08-ca2e43ea3320" />
   <img width="886" height="619" alt="gambar" src="https://github.com/user-attachments/assets/67af891e-907b-48a9-9721-227851076335" />
   <img width="887" height="351" alt="gambar" src="https://github.com/user-attachments/assets/60334f6b-b14f-4cad-8e63-9dabd6d797c2" />
   <img width="884" height="358" alt="gambar" src="https://github.com/user-attachments/assets/1da93f8e-8a5d-486a-98cc-2f0d9cbc3b41" />

2. Perhatikan keluaran program yang dijalankan tersebut. Dari keluaran program tersebut, jelaskan secara sederhana algoritma Raft untuk memilih koordinator (LEADER) menggunakan visualisasi (digambar dengan manual pensil/ballpoint dan kemudian difoto) .

   <img width="1185" height="1500" alt="gambar" src="https://github.com/user-attachments/assets/618a29a0-ace1-439f-9b5c-511c704ae368" />

3. Buatlah program tersebut menjadi modul Python dan kemudian buatlah contoh simulasinya menggunakan modul Python yang sudah anda buat tersebut.
   
a. Modul: raft_module.py: Simpan kode berikut sebagai raft_module.py

   <img width="875" height="433" alt="gambar" src="https://github.com/user-attachments/assets/a46fc612-2b0f-498d-b6e5-95df433b820a" />
   <img width="876" height="398" alt="gambar" src="https://github.com/user-attachments/assets/0458603e-b2f0-4674-a248-ca0ee53ab6b1" />
   <img width="875" height="287" alt="gambar" src="https://github.com/user-attachments/assets/b99e7e0e-4179-408c-947f-f2cb5bf43f8f" />

b. contoh imulasi: buat file baru dengan nama main_raft_module.py lalu isikan kode berikut:

   <img width="1002" height="610" alt="gambar" src="https://github.com/user-attachments/assets/0b0c26f0-abde-4e8d-bc3d-28ae20ae1325" />
   <img width="1005" height="168" alt="gambar" src="https://github.com/user-attachments/assets/2f7025b4-9a8c-441b-8927-2780c6e130a5" />

c. output / keluaran:

   <img width="886" height="542" alt="gambar" src="https://github.com/user-attachments/assets/18a589a3-21c8-49c6-911b-e3f8f7b7a7ff" />
   <img width="887" height="616" alt="gambar" src="https://github.com/user-attachments/assets/875e6f7d-9f4a-4359-88ef-fddb16a59a7e" />
   <img width="888" height="272" alt="gambar" src="https://github.com/user-attachments/assets/75248dc7-c839-4579-bfeb-b400a035042f" />



   



















