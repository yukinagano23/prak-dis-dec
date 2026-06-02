# Modul 11
## Sistem Terdesentralisasi, Blockchain, dan Web 3.0

---

### Bagian 1: Cara Kerja Blockchain
#### Langkah 1.1 — Hash SHA-256 (Tugas 1)
Jalankan Python interaktif:
```bash
python3
```
Kemudian ketik kode berikut satu per satu:
```bash
import hashlib

# Hash teks pertama
text = "UTDI"
hash_object = hashlib.sha256(text.encode('utf-8'))
hex_dig = hash_object.hexdigest()
print(f"SHA-256 Hash: {hex_dig}")

# Hash teks kedua
text = "Fakultas Teknologi Informasi"
hash_object = hashlib.sha256(text.encode('utf-8'))
hex_dig = hash_object.hexdigest()
print(f"SHA-256 Hash: {hex_dig}")

# Coba modifikasi kecil — hanya 1 karakter berbeda
text = "UTDI!"
hash_object = hashlib.sha256(text.encode('utf-8'))
hex_dig = hash_object.hexdigest()
print(f"SHA-256 Hash: {hex_dig}")

exit()
```
<img width="1025" height="638" alt="image" src="https://github.com/user-attachments/assets/91c99e63-4ad5-4281-8c37-00153e08ca1b" />

#### Jawaban Tugas 1 : Simpulan terkait hasil proses hash:
1. Deterministik: Input yang sama selalu menghasilkan hash yang sama persis. "UTDI" akan selalu menghasilkan hash e26d7d497c24e33ca2027102df133770c250847dd64280218fb7cb38a5127e64 tanpa pengecualian.
2. Fixed-length output: Berapa pun panjang input (1 karakter atau 1 juta karakter), output SHA-256 selalu berupa string 64 karakter hexadecimal (256 bit).
3. Avalanche effect (efek longsoran): Perubahan sekecil apapun pada input (bahkan 1 karakter) menghasilkan hash yang sama sekali berbeda. "UTDI" vs "UTDI!" menghasilkan hash yang tidak ada kemiripannya sama sekali.
4. One-way function (fungsi satu arah): Dari hash tidak bisa dikembalikan ke teks aslinya. Ini yang membuat SHA-256 aman untuk kriptografi.
5. Collision resistant: Sangat sulit (secara komputasi hampir mustahil) menemukan dua input berbeda yang menghasilkan hash yang sama.
6. Implikasi pada Blockchain: Sifat-sifat di atas menjadikan hash sebagai "sidik jari digital" yang ideal untuk mengunci dan memverifikasi integritas setiap blok dalam blockchain.

#### Langkah 1.2 — Membuat File CoreBlockchain.py
Keluar dari Python interaktif (sudah dilakukan di atas), lalu:
```bash
nano CoreBlockchain.py
```
Ketik/paste kode berikut:
```bash
import hashlib
import time

class CoreBlockchain:
    def __init__(self, idx, data, previous_hash):
        self.idx = idx
        self.timestamp = time.time()
        self.data = data
        self.previous_hash = previous_hash
        self.nonce = 0  # A nonce ("number used only once")
                        # is a random or semi-random number
                        # used in cryptography and blockchain
                        # to secure networks, authenticate transactions,
                        # and facilitate the creation of new blocks
        self.hash = self.count_hash()

    def count_hash(self):
        # combine all data in a block to be hashed.
        block_contents = (
            str(self.idx) +
            str(self.timestamp) +
            str(self.data) +
            str(self.previous_hash) +
            str(self.nonce)
        )
        return hashlib.sha256(block_contents.encode()).hexdigest()
```
Simpan dengan Ctrl+X → Y → Enter.
<img width="1025" height="475" alt="image" src="https://github.com/user-attachments/assets/7188de9c-b2d1-4073-a904-badaed2f3d47" />

#### Langkah 1.3 — Membuat File UtdiBlockchain.py
```bash
nano UtdiBlockchain.py
```
```bash
import CoreBlockchain

class UtdiBlockchain:
    def __init__(self):
        self.chain = []
        self.init_genesis_block()  # Genesis is the first block in Blockchain

    def init_genesis_block(self):
        # Genesis always has 0 for previous block.
        first_block = CoreBlockchain.CoreBlockchain(0, "Blok Genesis / Awal", "0")
        self.chain.append(first_block)

    def add_block(self, new_data):
        last_block = self.chain[-1]
        new_block = CoreBlockchain.CoreBlockchain(
            idx=len(self.chain),
            data=new_data,
            previous_hash=last_block.hash  # Lock new block with hash from
                                           # previous block
        )
        self.chain.append(new_block)
```
Simpan dengan Ctrl+X → Y → Enter.
<img width="1024" height="379" alt="image" src="https://github.com/user-attachments/assets/4e56f41c-f9f3-4309-8c80-328445fffb29" />

#### Langkah 1.4 — Membuat File blockchain_demo_01.py
```bash
nano blockchain_demo_01.py
```
```bash
import UtdiBlockchain

informatika_blockchain = UtdiBlockchain.UtdiBlockchain()

# Membuat 2 data:
informatika_blockchain.add_block("Bambang transfer ke Zaky sebesar 5 koin")
informatika_blockchain.add_block("Zaky transfer ke Didik sebesar 10 koin")

# Cetak isi Blockchain
for block in informatika_blockchain.chain:
    print(f"Blok #{block.idx}")
    print(f"Data: {block.data}")
    print(f"Hash Sebelumnya: {block.previous_hash}")
    print(f"Hash Sekarang : {block.hash}\n" + "-"*40)
```
<img width="1023" height="279" alt="image" src="https://github.com/user-attachments/assets/82a10375-d49a-4e42-816f-f5511c187e8f" />

Simpan, lalu jalankan:
```bash
python3 blockchain_demo_01.py
```
Output yang diharapkan (hash akan berbeda karena timestamp berbeda):
```bash
Blok #0
Data: Blok Genesis / Awal
Hash Sebelumnya: 0
Hash Sekarang : 6a61b3182669590bef5187a069c72bf664e9e7796f9f3a570b0f8903115f31d5
----------------------------------------
Blok #1
Data: Bambang transfer ke Zaky sebesar 5 koin
Hash Sebelumnya: 6a61b3182669590bef5187a069c72bf664e9e7796f9f3a570b0f8903115f31d5
Hash Sekarang : 2af0787b70b579ae85f2fba85e0c08de052a7fd7a0bc4527e09597202037d7e9
----------------------------------------
Blok #2
Data: Zaky transfer ke Didik sebesar 10 koin
Hash Sebelumnya: 2af0787b70b579ae85f2fba85e0c08de052a7fd7a0bc4527e09597202037d7e9
Hash Sekarang : 8a3c183faaf15fb6df5fa52676b6c7947c2402524681e00bc6a84ed4c12b9536
----------------------------------------
```
<img width="1024" height="291" alt="image" src="https://github.com/user-attachments/assets/d6def1cc-6a6e-485f-a273-eae82b1c2357" />

#### Jawaban Tugas 2: Penjelasan source code:
#### CoreBlockchain.py:
* __init__(self, idx, data, previous_hash) — Constructor yang dipanggil saat blok baru dibuat. Menerima 3 parameter: nomor urut blok (idx), isi data transaksi (data), dan hash blok sebelumnya (previous_hash).
* self.timestamp = time.time() — Mencatat waktu Unix (detik sejak 1 Jan 1970) saat blok dibuat. Ini memastikan setiap blok punya cap waktu yang unik.
* self.nonce = 0 — Angka "number used only once". Dalam implementasi sederhana ini bernilai 0, namun pada blockchain nyata nonce digunakan dalam proses Proof-of-Work untuk mining.
* self.hash = self.count_hash() — Langsung menghitung hash blok saat objek dibuat.
* count_hash() — Menggabungkan semua atribut blok (idx + timestamp + data + previous_hash + nonce) menjadi satu string, lalu di-hash menggunakan SHA-256. Ini adalah "sidik jari" unik dari blok tersebut.

#### UtdiBlockchain.py:
* __init__ — Menginisialisasi chain sebagai list kosong, lalu langsung membuat genesis block.
* init_genesis_block() — Membuat blok pertama (genesis) dengan idx=0, data bertuliskan "Blok Genesis / Awal", dan previous_hash="0" (karena tidak ada blok sebelumnya).
* add_block(new_data) — Mengambil blok terakhir di chain (self.chain[-1]), menggunakan hash-nya sebagai previous_hash untuk blok baru. Ini yang menciptakan "rantai" (chain) — setiap blok terkunci ke blok sebelumnya melalui hash.

#### blockchain_demo_01.py:
* Membuat instance blockchain, menambah 2 transaksi, lalu mencetak seluruh isi chain.
* Dari output terlihat bahwa Hash Sekarang pada blok #0 sama persis dengan Hash Sebelumnya pada blok #1 — inilah mekanisme penguncian yang membuat blockchain tamper-proof: jika data blok #0 diubah, hash-nya berubah, sehingga blok #1 dan seterusnya menjadi tidak valid.

---

### Bagian 2: Pengenalan Blockchain dan Ethereum
#### Langkah 2.1 — Install MetaMask di Browser
Di lingkungan WSL Debian, MetaMask dipasang di browser Windows (bukan di terminal WSL), karena MetaMask adalah ekstensi browser. Ikuti langkah berikut:
1. Buka Chrome di Windows
2. Kunjungi: https://metamask.io → klik Get MetaMask → pilih Chrome
3. Klik Add to Chrome → Add extension
<img width="1588" height="818" alt="image" src="https://github.com/user-attachments/assets/2b912e1d-cdc1-4839-ab3c-9b3a43d1515a" />
<img width="1584" height="819" alt="image" src="https://github.com/user-attachments/assets/fcde4905-6cbb-42ac-b517-8b6b8f3b3504" />

#### Langkah 2.2 — Membuat Wallet MetaMask Baru
1. Klik ekstensi MetaMask yang sudah terpasang
2. Klik Continue (pada halaman "Help improve MetaMask")
<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/ba2e9b00-6d90-4d6b-87a3-47dd52f75476" />

3. Pilih Create a new wallet (akan di suruh login, pilih login dengan google)
<img width="1600" height="816" alt="image" src="https://github.com/user-attachments/assets/cbab88ee-4f33-45a0-a586-2b09705b405d" />

4. Pilih Use Secret Recovery Phrase
5. Buat password minimal 8 karakter → centang persetujuan → klik Create password
6. PENTING: Catat dan simpan Secret Recovery Phrase (12 kata) di tempat yang sangat aman — jangan sampai diketahui orang lain dan jangan disimpan digital secara sembarangan
7. Jawab validasi recovery phrase sesuai urutan yang diminta
8. Klik Done — wallet siap digunakan

#### Langkah 2.3 — Menggunakan Testnet Sepolia
1. Buka MetaMask → klik menu (≡) → pilih Networks
2. Aktifkan toggle Show test networks
3. Pilih Sepolia dari daftar jaringan
<img width="350" height="320" alt="image" src="https://github.com/user-attachments/assets/a5a30554-2f9a-47e9-a97d-b302423cda74" />

#### Langkah 2.4 — Mendapatkan ETH Testnet dari Faucet
1. Buka browser → kunjungi https://faucets.chain.link/
<img width="1590" height="820" alt="image" src="https://github.com/user-attachments/assets/a84042cc-b27b-4f19-bece-935c124aae7d" />

2. Pilih Ethereum Sepolia (yang memberikan 25 LINK atau 0.5 ETH)
<img width="1582" height="821" alt="image" src="https://github.com/user-attachments/assets/e77d308a-bd95-4851-8665-0a06f9f05980" />

3. Klik Continue
<img width="1599" height="819" alt="image" src="https://github.com/user-attachments/assets/1bc688fb-2357-4064-885d-83e0d12bc2da" />

4. Klik Connect → pilih MetaMask → klik Connect di dialog MetaMask
<img width="1598" height="820" alt="image" src="https://github.com/user-attachments/assets/b0d7a288-c6e1-47d1-b587-0d2214581bbe" />
<img width="383" height="606" alt="image" src="https://github.com/user-attachments/assets/af295c5b-8ff1-43e9-ba7e-4d1a9a32abdb" />

5. Kembali ke halaman faucet → klik Get tokens
<img width="1594" height="817" alt="image" src="https://github.com/user-attachments/assets/ed178039-9d11-47ed-a33e-42a2a9fcb101" />

6. Konfirmasi Signature Request di MetaMask → klik Confirm
7. Tunggu hingga status menampilkan Success beserta Transaction Hash
<img width="957" height="545" alt="image" src="https://github.com/user-attachments/assets/7d8888da-6c42-4931-b373-ad2b32cde0ad" />

#### Jawaban Tugas 3
1. Penjelasan istilah-istilah Web3:

   **a. DApps (Decentralized Applications)** Aplikasi yang berjalan di atas jaringan blockchain terdesentralisasi, bukan di server terpusat milik satu perusahaan. Logika bisnis DApps ditulis dalam smart contract yang di-deploy ke blockchain. Karakteristik utamanya: open source, data tersimpan di blockchain (transparan), beroperasi tanpa otoritas pusat, dan tidak ada single point of failure. Contoh: Uniswap (DEX), Aave (lending), OpenSea (NFT marketplace).

    **b. NFT (Non-Fungible Token)** Token digital yang bersifat unik dan tidak dapat dipertukarkan satu-satu (non-fungible) dengan token lain — berbeda dengan ETH atau Bitcoin yang fungible (1 ETH = 1 ETH manapun). Setiap NFT memiliki identitas unik yang tercatat di blockchain, sehingga bisa membuktikan kepemilikan aset digital seperti karya seni, musik, item game, atau dokumen. NFT di Ethereum umumnya menggunakan standar ERC-721 atau ERC-1155.\

    **c. DEX (Decentralized Exchange)** Platform pertukaran aset kripto yang beroperasi tanpa perantara terpusat (berbeda dengan Binance atau Coinbase yang bersifat terpusat/CEX). DEX menggunakan smart contract dan mekanisme AMM (Automated Market Maker) untuk memfasilitasi transaksi langsung antar pengguna (peer-to-peer). Pengguna menyimpan kendali penuh atas private key mereka. Contoh populer: Uniswap, SushiSwap, Curve Finance.

   **d. TokenizationProses mengubah aset nyata (real-world assets)** maupun digital menjadi token digital di atas blockchain. Aset yang dapat di-tokenisasi sangat beragam: properti, saham, komoditas (emas), karya seni, hingga data. Tokenisasi memungkinkan kepemilikan fraksional (misalnya memiliki 0,001% gedung senilai miliaran), meningkatkan likuiditas aset yang sebelumnya tidak likuid, dan mempermudah transfer kepemilikan secara global tanpa birokrasi.

   **e. Stablecoins** Jenis cryptocurrency yang dirancang untuk memiliki nilai stabil dengan cara dipatok (pegged) ke aset referensi, umumnya mata uang fiat seperti USD. Tujuannya untuk mengatasi volatilitas tinggi yang menjadi ciri khas kripto seperti Bitcoin/Ethereum. Ada tiga jenis utama: (1) Fiat-backed — didukung cadangan mata uang nyata, contoh: USDT, USDC; (2) Crypto-backed — dijaminkan dengan kripto lain dengan over-collateralization, contoh: DAI; (3) Algorithmic — menjaga stabilitas melalui mekanisme algoritma supply-demand, contoh: FRAX.

3. Peranti Pengembangan untuk Membangun DApps di Ethereum:

   **a. Hardhat (Development Environment)** Hardhat adalah framework pengembangan Ethereum berbasis Node.js yang menjadi standar industri saat ini. Alasan memilihnya: menyediakan local Ethereum network untuk testing tanpa biaya gas, mendukung TypeScript, memiliki sistem plugin yang luas, debugging yang sangat baik (stack traces yang informatif saat transaksi gagal), dan dokumentasi lengkap. Perintah instalasi: npm install --save-dev hardhat.

   **b. Solidity (Bahasa Smart Contract)** Bahasa pemrograman resmi dan paling matang untuk menulis smart contract di Ethereum. Dipilih karena ekosistemnya paling besar, banyak library siap pakai (OpenZeppelin), dan dukungan komunitas terluas.

   **c. OpenZeppelin Contracts (Library Smart Contract)** Library smart contract yang sudah diaudit keamanannya dan siap pakai. Menyediakan implementasi standar ERC-20, ERC-721 (NFT), access control, dan lainnya. Sangat penting untuk keamanan karena menulis smart contract dari nol sangat berisiko terhadap exploit. Instalasi: npm install @openzeppelin/contracts.

   **d. ethers.js (Library Interaksi Blockchain)** Library JavaScript untuk berinteraksi dengan Ethereum dari sisi frontend/backend. Dipilih dibanding Web3.js karena ukurannya lebih kecil, API lebih modern dan intuitif, serta dokumentasi yang lebih baik.

   **e. React + Vite (Frontend)** Framework UI yang populer dengan ekosistem Web3 terluas. Banyak library Web3 UI dibuat untuk React (seperti wagmi, RainbowKit untuk wallet connection).

   **f. Remix IDE (untuk prototyping cepat)** IDE berbasis browser di remix.ethereum.org — sangat berguna untuk menulis dan menguji smart contract secara cepat tanpa setup apapun, cocok untuk tahap awal pengembangan dan pembelajaran.







