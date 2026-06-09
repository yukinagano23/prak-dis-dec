# Modul 12. Teknologi P2P (Peer-to-Peer)

### 0. Pengantar Teknologi P2P
Teknologi P2P terdiri atas sekumpulan nodes yang terhubung secara langsung tanpa adanya suatu server yang menjadi perantara. Jadi, node bisa berfungsi sebagai client sekaligus berfungsi sebagai server. Beberapa kemungkinan penggunaan teknologi ini adalah sebagai berikut:
1. Berbagi file / file sharing
2. Aplikasi Chat
3. Aplikasi games

#### Persiapan
Pastikan Python sudah terinstall. Buka Command Prompt (tekan Win + R, ketik cmd, lalu Enter) dan verifikasi:
```bash
python --version
```
Jika belum terinstall, unduh dari 
```bash
https://www.python.org/downloads/
```
dan centang "Add Python to PATH" saat instalasi.

---

### 1. Koneksi Antar Nodes
Program berikut merupakan program chat sederhana menggunakan Python. Dengan mempelajari program ini, bisa diketahui bagaimana cara koneksi antar nodes dilakukan serta bagaimana cara mengirimkan message antar node tersebut.
#### Langkah persiapan
Buka Notepad atau VS Code, buat file baru bernama simple_chat.py, lalu simpan di folder misalnya C:\P2P\. lalu isikan kode berikut:
```bash
import socket
import threading
import sys
import time

def terima_pesan(port_saya, siap_event):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    try:
        server_socket.bind(('0.0.0.0', port_saya))
        server_socket.listen(1)

        # Tunggu sampai semua input konfigurasi selesai
        siap_event.wait()

        print(f"\n[SERVER] Mendengarkan di port {port_saya}...")

        koneksi, alamat_peer = server_socket.accept()
        print(f"\n[SERVER] Terhubung dengan peer: {alamat_peer}")

        while True:
            data = koneksi.recv(1024)
            if not data:
                print("\n[SERVER] Peer memutuskan koneksi.")
                break
            print(f"\n[Peer]: {data.decode('utf-8')}")
            print(">> ", end="", flush=True)

    except Exception as e:
        print(f"[SERVER] Error: {e}")
    finally:
        server_socket.close()

def kirim_pesan(ip_tujuan, port_tujuan):
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    print(f"\n[CLIENT] Mencoba terhubung ke {ip_tujuan}:{port_tujuan}...")
    
    # Coba koneksi beberapa kali (peer mungkin belum siap)
    for percobaan in range(5):
        try:
            client_socket.connect((ip_tujuan, port_tujuan))
            print("[CLIENT] Sukses terhubung!")
            break
        except Exception:
            print(f"[CLIENT] Belum terhubung, mencoba ulang ({percobaan+1}/5)...")
            time.sleep(2)
    else:
        print("[CLIENT] Gagal terhubung setelah 5 percobaan.")
        return

    try:
        print("Silakan ketik pesan dan tekan Enter (Ketik 'keluar' untuk berhenti):")
        while True:
            print(">> ", end="", flush=True)
            pesan = input()
            if pesan.lower() == 'keluar':
                break
            client_socket.sendall(pesan.encode('utf-8'))
    except Exception as e:
        print(f"[CLIENT] Error: {e}")
    finally:
        client_socket.close()

if __name__ == "__main__":
    print("=== Praktikum Modul 12 - Sub 01 ===")
    port_lokal = int(input("Masukkan PORT LOKAL untuk server Anda (contoh: 5001): "))

    print("\n--- Konfigurasi Hubungan ke Peer Lain ---")
    ip_target = input("Masukkan IP TARGET (Peer tujuan, contoh: 127.0.0.1 atau localhost): ")
    port_target = int(input("Masukkan PORT TARGET (Port server peer tujuan): "))

    # Semua input sudah selesai, baru aktifkan server
    siap = threading.Event()
    thread_server = threading.Thread(target=terima_pesan, args=(port_lokal, siap))
    thread_server.daemon = True
    thread_server.start()

    siap.set()  # Beri sinyal: input sudah selesai, server boleh mulai

    kirim_pesan(ip_target, port_target)
    print("\nProgram Selesai.")
```


***catatan:*** Di Windows, print(sys.stderr, ...) mencetak objek stderr sebagai teks biasa, bukan mengarahkan output ke stderr. Untuk menghindari keanehan output, baris-baris tersebut sudah diubah menjadi print(f"...") biasa pada versi ini.

#### Cara menjalankan di Windows (simulasi 2 node di 1 komputer):
Buka dua jendela Command Prompt secara terpisah. Masuk ke folder file:
```bash
cd C:\P2P
```
<img width="1272" height="716" alt="image" src="https://github.com/user-attachments/assets/879699e7-49ee-4125-ad7b-ce0c03999033" />
<img width="1274" height="601" alt="image" src="https://github.com/user-attachments/assets/f971cc87-1e50-4cbd-922f-3bcb17a562c0" />
<img width="1272" height="219" alt="image" src="https://github.com/user-attachments/assets/0592ec68-c460-403e-86ae-b1c73529c08c" />

#### Jendela CMD ke-1 (Node A — server di port 5001, client ke port 5002):
```bash
python simple_chat.py
```
```bash
=== Praktikum Modul 12 - Sub 01 ===
Masukkan PORT LOKAL untuk server Anda (contoh: 5001): 5001

[SERVER] Mendengarkan di port 5001...

--- Konfigurasi Hubungan ke Peer Lain ---
Masukkan IP TARGET (Peer tujuan, contoh: 127.0.0.1 atau localhost): localhost
Masukkan PORT TARGET (Port server peer tujuan): 5002
```
<img width="957" height="288" alt="image" src="https://github.com/user-attachments/assets/f9c10a8b-f0f2-40eb-9b01-c3bc300bf059" />

#### Jendela CMD ke-2 (Node B — server di port 5002, client ke port 5001):
```bash
python simple_chat.py
```
```bash
=== Praktikum Modul 12 - Sub 01 ===
Masukkan PORT LOKAL untuk server Anda (contoh: 5001): 5002

[SERVER] Mendengarkan di port 5002...

--- Konfigurasi Hubungan ke Peer Lain ---
Masukkan IP TARGET (Peer tujuan, contoh: 127.0.0.1 atau localhost): localhost
Masukkan PORT TARGET (Port server peer tujuan): 5001
```
<img width="956" height="184" alt="image" src="https://github.com/user-attachments/assets/0abe62ac-a59a-4c45-97fc-52f3b824e1c0" />
Setelah keduanya terhubung, ketik pesan di salah satu jendela dan tekan Enter — pesan akan muncul di jendela lainnya. Ketik keluar untuk menghentikan program.

#### Contoh output saat berjalan:
```bash
=== Praktikum Modul 12 - Sub 01 ===
Masukkan PORT LOKAL untuk server Anda (contoh: 5001): 5001

--- Konfigurasi Hubungan ke Peer Lain ---
Masukkan IP TARGET (Peer tujuan, contoh: 127.0.0.1 atau localhost): localhost
Masukkan PORT TARGET (Port server peer tujuan): 5002

[CLIENT] Mencoba terhubung ke localhost:5002...
[SERVER] Mendengarkan di port 5001...

[CLIENT] Sukses terhubung!
Silakan ketik pesan dan tekan Enter (Ketik 'keluar' untuk berhenti):
>>
[SERVER] Terhubung dengan peer: ('127.0.0.1', 36960)

[Peer]: halo dari node B
>> halo balik dari node A
>>
```

#### Jawaban Tugas 1
1. **Hasil capture program:** Output di Jendela Node A:
   <img width="962" height="305" alt="image" src="https://github.com/user-attachments/assets/99605a66-69ce-40a6-b45b-03d122cbe775" />

2. **Jelaskan bagian dalam program tersebut yang digunakan untuk:**
   a. Membuka port yang akan menerima dan mengirim pesan:

      **Bagian yang Membuka Port PENERIMA (Server)**
      ```bash
      server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
      server_socket.bind(('0.0.0.0', port_saya))
      server_socket.listen(1)
      ```
      Penjelasan tiap baris:
      * socket.socket(...) → membuat objek socket baru menggunakan IPv4 dan protokol TCP
      * setsockopt(SO_REUSEADDR, 1) → mengizinkan port yang sama dipakai ulang jika program dijalankan ulang
      * bind(('0.0.0.0', port_saya)) → inilah yang "membuka" port — mendaftarkan socket ke port lokal yang Anda masukkan (misal 5001). 0.0.0.0 artinya menerima koneksi dari semua network interface
      * listen(1) → mulai mendengarkan koneksi masuk, angka 1 = maksimal 1 antrian koneksi

     **Bagian yang Membuka Port PENGIRIM (Client)**
     ```bash
     client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
     client_socket.connect((ip_tujuan, port_tujuan))
     ```
     Penjelasan tiap baris:
     * socket.socket(...) → membuat objek socket baru untuk sisi client
     * connect((ip_tujuan, port_tujuan)) → inilah yang "membuka" koneksi ke port tujuan — sistem operasi otomatis memilihkan port lokal yang kosong (misalnya 36912 seperti yang terlihat di screenshot Anda), lalu menyambungkan ke port target peer (misal 5002)

   b. Menerima pesan
      ```bash
      koneksi, alamat_peer = server_socket.accept()

      while True:
      data = koneksi.recv(1024)
      if not data:
        print("\n[SERVER] Peer memutuskan koneksi.")
        break
      print(f"\n[Peer]: {data.decode('utf-8')}")
      ```
      Penjelasan tiap baris:
      * server_socket.accept() → menunggu hingga ada peer yang menghubungkan diri. Saat peer berhasil masuk, fungsi ini mengembalikan dua nilai: koneksi (objek socket untuk berkomunikasi dengan peer tersebut) dan alamat_peer (IP + port si peer)
      * koneksi.recv(1024) → inilah yang menerima pesan — membaca data yang dikirim peer, maksimal 1024 bytes sekali baca. Fungsi ini memblokir (berhenti menunggu) sampai ada data yang masuk
      * if not data: break → jika data yang diterima kosong, berarti peer sudah memutuskan koneksi, maka loop dihentikan
      * data.decode('utf-8') → mengubah data bytes yang diterima menjadi teks yang bisa dibaca dan ditampilkan
     
   c. Mengirim pesan
      ```bash
      print(">> ", end="", flush=True)
      pesan = input()
      if pesan.lower() == 'keluar':
          break
      client_socket.sendall(pesan.encode('utf-8'))
      ```
     Penjelasan tiap baris:
      * input() → menangkap teks yang Anda ketik di keyboard dan simpan ke variabel pesan
      * if pesan.lower() == 'keluar': break → jika yang diketik adalah kata keluar, hentikan loop dan tutup koneksi
      * pesan.encode('utf-8') → mengubah teks menjadi bytes karena jaringan hanya bisa mengirim bytes, bukan teks langsung
      * client_socket.sendall(...) → inilah yang mengirim pesan — mengirimkan seluruh bytes ke peer tujuan melalui koneksi yang sudah terbuka. Perbedaan sendall() vs send() adalah sendall() memastikan semua data terkirim, sedangkan send() bisa saja hanya mengirim sebagian

---

### 2. DHT (Distributed Hash Table)
DHT merupakan mekanisme yang biasanya digunakan oleh teknologi P2P untuk pencarian data tanpa adanya server yang menyimpan semua data. Jika pernah menggunakan BitTorrent, DHT merupakan salah satu komponen untuk pencarian dan koneksi ke node lainnya. Berikut adalah simulasi DHT menggunakan Python.

Buat file dht.py di folder C:\P2P\ lalu isikkan kode berikut:
```bash
import hashlib

def hitung_hash_8bit(teks):
    sha1 = hashlib.sha1(teks.encode('utf-8')).hexdigest()
    return int(sha1[-2:], 16)

class NodeP2P:
    def __init__(self, nama_node):
        self.nama = nama_node
        self.id = hitung_hash_8bit(nama_node)
        self.penyimpanan_lokal = {}
        print(f"Node '{self.nama}' berhasil dibuat dengan ID: {self.id}")

class LingkaranDHT:
    def __init__(self):
        self.daftar_node = []

    def tambah_node(self, node):
        self.daftar_node.append(node)
        self.daftar_node.sort(key=lambda x: x.id)

    def cari_node_terdekat(self, key_data):
        for node in self.daftar_node:
            if node.id >= key_data:
                return node
        return self.daftar_node[0]

    def simpan_data(self, nama_file, isi_konten):
        key_data = hitung_hash_8bit(nama_file)
        node_target = self.cari_node_terdekat(key_data)
        node_target.penyimpanan_lokal[key_data] = (nama_file, isi_konten)
        print(f"[SIMPAN] File '{nama_file}' (Key ID: {key_data}) "
              f"disimpan di Node '{node_target.nama}' (Node ID: {node_target.id})")

    def cari_data(self, nama_file):
        key_data = hitung_hash_8bit(nama_file)
        node_target = self.cari_node_terdekat(key_data)

        print(f"\n[PENCARIAN] Mencari file '{nama_file}' dengan Key ID: {key_data}...")
        print(f"[ROUTING] Request diarahkan ke Node terdekat: "
              f"'{node_target.nama}' (Node ID: {node_target.id})")

        if key_data in node_target.penyimpanan_lokal:
            nama, konten = node_target.penyimpanan_lokal[key_data]
            print(f"[SUKSES] Data ditemukan! Isi konten: '{konten}'")
        else:
            print("[GAGAL] Data tidak ditemukan di jaringan.")

if __name__ == "__main__":
    print("=== Praktikum Modul 12 - Teknologi P2P - "
          "Simulasi Distributed Hash Table (DHT) ===")

    dht = LingkaranDHT()

    node_a = NodeP2P("Node A")
    node_b = NodeP2P("Node B")
    node_c = NodeP2P("Node C")

    dht.tambah_node(node_a)
    dht.tambah_node(node_b)
    dht.tambah_node(node_c)

    print("\nUrutan Node dalam Lingkaran DHT (Ring):")
    for n in dht.daftar_node:
        print(f"  -> Node ID: {n.id} ({n.nama})")

    dht.simpan_data("tugas_jaringan.pdf", "Konten: Laporan Praktikum Modul 1")
    dht.simpan_data("foto_makrab.jpg", "Konten: Data biner gambar makrab angkatan")
    dht.simpan_data("source_code.py", "Konten: print('Hello P2P')")

    dht.cari_data("tugas_jaringan.pdf")
    dht.cari_data("source_code.py")
    dht.cari_data("praktikum.py")
```
<img width="1114" height="736" alt="image" src="https://github.com/user-attachments/assets/1088ea0c-178f-4a72-916b-1ae2390dfa58" />
<img width="1112" height="649" alt="image" src="https://github.com/user-attachments/assets/25fd697c-ff14-40e0-b32b-35aaf0dcbe36" />

Cara menjalankan:
```bash
cd C:\P2P
python dht.py
```
output yang diharapkan:
```bash
=== Praktikum Modul 12 - Teknologi P2P - Simulasi Distributed Hash Table (DHT) ===
Node 'Node A' berhasil dibuat dengan ID: 129
Node 'Node B' berhasil dibuat dengan ID: 217
Node 'Node C' berhasil dibuat dengan ID: 92

Urutan Node dalam Lingkaran DHT (Ring):
  -> Node ID: 92 (Node C)
  -> Node ID: 129 (Node A)
  -> Node ID: 217 (Node B)
[SIMPAN] File 'tugas_jaringan.pdf' (Key ID: 115) disimpan di Node 'Node A' (Node ID: 129)
[SIMPAN] File 'foto_makrab.jpg' (Key ID: 124) disimpan di Node 'Node A' (Node ID: 129)
[SIMPAN] File 'source_code.py' (Key ID: 159) disimpan di Node 'Node B' (Node ID: 217)

[PENCARIAN] Mencari file 'tugas_jaringan.pdf' dengan Key ID: 115...
[ROUTING] Request diarahkan ke Node terdekat: 'Node A' (Node ID: 129)
[SUKSES] Data ditemukan! Isi konten: 'Konten: Laporan Praktikum Modul 1'

[PENCARIAN] Mencari file 'source_code.py' dengan Key ID: 159...
[ROUTING] Request diarahkan ke Node terdekat: 'Node B' (Node ID: 217)
[SUKSES] Data ditemukan! Isi konten: 'Konten: print('Hello P2P')'

[PENCARIAN] Mencari file 'praktikum.py' dengan Key ID: 68...
[ROUTING] Request diarahkan ke Node terdekat: 'Node C' (Node ID: 92)
[GAGAL] Data tidak ditemukan di jaringan.
```
***Catatan:*** Nilai ID node (hash) bersifat deterministik — selalu menghasilkan angka yang sama untuk nama yang sama. Urutan node di ring bisa berbeda tergantung hasil hash masing-masing nama.

#### Jawaban Tugas 2

1. **Jalankan program:**
   <img width="1204" height="482" alt="image" src="https://github.com/user-attachments/assets/2a232012-43a0-4efd-adf3-8f97c204ab80" />
2. **Penjelasan singkat apa yang dilakukan program:** Program dht.py adalah simulasi Distributed Hash Table (DHT) di jaringan P2P. Program membuat tiga node virtual (Node A, B, C), lalu masing-masing node diberi ID unik berdasarkan hasil hash SHA-1 dari namanya (diambil 8-bit terakhir, nilai 0–255). Node-node ini disusun dalam sebuah lingkaran/ring berdasarkan urutan ID. Saat menyimpan file, nama file di-hash menjadi sebuah Key ID, lalu file tersebut disimpan di node yang memiliki ID ≥ Key ID (node terdekat searah jarum jam di ring). Saat pencarian, proses yang sama diulang — hash nama file, cari node terdekat, lalu periksa apakah file ada di sana.
3. **Algoritma pencarian data menggunakan DHT:**
   <img width="1276" height="443" alt="image" src="https://github.com/user-attachments/assets/a49be8c4-6d3d-41aa-98b8-5520dfa40a9d" />
   Konsep kuncinya adalah konsistensi hashing: file selalu disimpan dan dicari di node yang sama (ditentukan oleh hash), sehingga setiap node hanya perlu menyimpan sebagian data, dan pencarian bisa dilakukan langsung ke node yang bertanggung jawab tanpa perlu bertanya ke semua node.

---

### 3. Torrent
Torrent adalah teknologi berbagi file peer-to-peer (P2P) yang memungkinkan pengguna mengunduh dan mengunggah file langsung antar perangkat tanpa melalui server pusat. Teknologi ini biasanya digunakan untuk mendistribusikan file berukuran sangat besar secara cepat dan efisien.

Salah satu resources di Web yang menyediakan torrent untuk berbagai ISO dari Linux dan/atau berbagai sistem operasi bebas serta software bebas lainnya adalah https://fosstorrents.com/.

#### Langkah persiapan — install library bcoding:
```bash
pip install bcoding
```
<img width="1205" height="186" alt="image" src="https://github.com/user-attachments/assets/4853053f-9f97-4120-b967-5e25d1eb89c8" />

#### Langkah mendapatkan file .torrent untuk percobaan:
1. Buka browser, pergi ke https://fosstorrents.com/
2. Cari misalnya FreeBSD atau distro Linux lainnya
   <img width="1578" height="818" alt="image" src="https://github.com/user-attachments/assets/b64c78ef-943e-4cea-b0d5-607b4bcd7654" />

3. Unduh file .torrent-nya (bukan file ISO-nya)
5. Simpan file .torrent tersebut di folder C:\P2P\

#### Buat file read_torrent.py di folder C:\P2P\:
```bash
import bcoding
import hashlib

def baca_metadata_torrent(path_file_torrent):
    print(f"=== ANALISIS METADATA TORRENT: {path_file_torrent} ===")

    try:
        with open(path_file_torrent, 'rb') as f:
            data_torrent = bcoding.bdecode(f)

        print(f"[TRACKER URL] : {data_torrent.get('announce')}")

        info = data_torrent.get('info')
        if info:
            print(f"[NAMA FILE]   : {info.get('name')}")
            print(f"[UKURAN FILE] : {info.get('length')} bytes")
            print(f"[UKURAN PIECE]: {info.get('piece length')} bytes")

            info_bencoded = bcoding.bencode(info)
            info_hash = hashlib.sha1(info_bencoded).hexdigest()
            print(f"[INFO HASH]   : {info_hash}")

            jumlah_pieces = len(info.get('pieces')) // 20
            print(f"[TOTAL PIECES]: {jumlah_pieces} potongan")

    except FileNotFoundError:
        print("Gagal: File .torrent tidak ditemukan. Pastikan path benar.")
    except Exception as e:
        print(f"Error saat membaca torrent: {e}")

if __name__ == "__main__":
    baca_metadata_torrent("FreeBSD-15.0-RELEASE-amd64-bootonly.iso.torrent")
```
<img width="1272" height="680" alt="image" src="https://github.com/user-attachments/assets/dd89eaa3-3e34-4d26-b040-b96a0882f68c" />

#### Cara menjalankan:
```bash
cd C:\P2P
python read_torrent.py
```

####  output:
```bash
=== ANALISIS METADATA TORRENT: FreeBSD-15.0-RELEASE-amd64-bootonly.iso.torrent ===
[TRACKER URL] : udp://fosstorrents.com:6969/announce
[NAMA FILE]   : FreeBSD-15.0-RELEASE-amd64-bootonly.iso
[UKURAN FILE] : 556255232 bytes
[UKURAN PIECE]: 262144 bytes
[INFO HASH]   : f9abad3a1b4956e7545d27e010af8faa416d0ae5
[TOTAL PIECES]: 2122 potongan
```

#### Jawaban Tugas 3
1. **Jalankan program:**
   <img width="1202" height="170" alt="image" src="https://github.com/user-attachments/assets/d840be61-dafb-428d-8df8-b8d0545ffa78" />
2. **Mengapa program memberi output seperti itu:**
   Program membaca file .torrent yang formatnya menggunakan Bencode (format encoding khas BitTorrent). Library bcoding.bdecode() mengurai format Bencode menjadi dictionary Python. Berikut penjelasan setiap output:
   * [TRACKER URL] → diambil dari key 'announce' di file torrent. Ini adalah URL server tracker yang berfungsi mempertemukan seeder dan leecher.
   * [NAMA FILE] → diambil dari info['name'], yaitu nama asli file yang akan diunduh.
   * [UKURAN FILE] → diambil dari info['length'] dalam satuan bytes.
   * [UKURAN PIECE] → diambil dari info['piece length']. File dibagi menjadi potongan-potongan (pieces) berukuran sama agar bisa diunduh dari banyak peer secara paralel.
   * [INFO HASH] → hasil SHA-1 dari bagian info yang di-encode ulang. Hash ini adalah identitas unik dari torrent, digunakan untuk verifikasi dan pencarian di jaringan DHT.
   * [TOTAL PIECES] → info['pieces'] berisi deretan hash SHA-1 (masing-masing 20 bytes) untuk setiap piece. Dibagi 20 menghasilkan jumlah total potongan file.
3. **Program read_torrent.py versi lebih fleksibel (nama file sebagai argumen):**
   <img width="1271" height="812" alt="image" src="https://github.com/user-attachments/assets/3dbe078f-39c7-4238-860d-cca086881afa" />
   
   **Cara menjalankan versi fleksibel:**
   ```bash
   python read_torrent.py FreeBSD-15.0-RELEASE-amd64-bootonly.iso.torrent
   ```
   <img width="1206" height="238" alt="image" src="https://github.com/user-attachments/assets/b7676f21-63b6-47bd-a11f-edabb37b6012" />

   **Perubahan utama dari versi awal:** Baris baca_metadata_torrent("FreeBSD-15.0-...") yang semula hardcoded diganti dengan sys.argv[1] — yaitu argumen pertama yang diberikan saat menjalankan program dari command line. Ditambahkan pula pengecekan if len(sys.argv) < 2 agar program memberikan pesan panduan yang jelas jika pengguna lupa menyertakan nama file.
   



