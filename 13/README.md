# Modul 13 
## Konsensus pada Blockchain

### 0. Pengantar
Solana adalah platform blockchain berkinerja tinggi yang mampu memproses puluhan ribu transaksi per detik (TPS) dengan biaya sangat murah. Berbeda dengan Bitcoin atau Ethereum yang memerlukan belasan detik hingga beberapa menit per blok, Solana menggabungkan dua mekanisme konsensus utama:
* Proof of History (PoH)  "jam digital kriptografis" yang mencatat urutan waktu setiap transaksi
* Proof of Stake (PoS)  menentukan validator berdasarkan jumlah token yang di-stake

Solana juga memiliki Solana Permissioned Environments (SPE), yaitu instans privat dari Solana Virtual Machine (SVM) untuk kebutuhan enterprise/organisasi.

---

### 1. Instalasi dan Persiapan
#### 1.1 Install Rust
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
<img width="841" height="755" alt="image" src="https://github.com/user-attachments/assets/f17ee5a6-73d7-4bb9-87e4-0fd88f3e00f5" />
<img width="844" height="443" alt="image" src="https://github.com/user-attachments/assets/9af30060-f7ad-40f4-b69f-48b76c88fb98" />

Saat muncul prompt pilihan, ketik 1 lalu tekan Enter (default installation). Setelah selesai, aktifkan Rust di sesi terminal saat ini:
```bash
source "$HOME/.cargo/env"
```
Verifikasi:
```bash
rustc --version
# Contoh output: rustc 1.96.0 (ac68faa20 2026-05-25)
```
<img width="840" height="126" alt="image" src="https://github.com/user-attachments/assets/26584258-64a0-4a08-b458-1f94be870b9d" />

#### 1.2 Install Node.js v24 LTS menggunakan fnm
```bash
curl -fsSL https://fnm.vercel.app/install | bash
```
<img width="844" height="277" alt="image" src="https://github.com/user-attachments/assets/50a46038-3f1f-4a31-91f3-cb1439e3d804" />

Setelah install, aktifkan fnm di sesi terminal saat ini:
```bash
source "$HOME/.bashrc"
```
Install Node.js versi LTS (24):
```bash
fnm install 24
fnm use 24
fnm default 24
```
Verifikasi:
```bash
node --version
# Contoh output: v24.17.0
```
<img width="845" height="156" alt="image" src="https://github.com/user-attachments/assets/8cfe768a-ca87-4a28-b14d-f432698640e1" />

Kemudian install yarn secara global:
```bash
npm install -g yarn
yarn --version
# Contoh output: 1.22.22
```
<img width="842" height="222" alt="image" src="https://github.com/user-attachments/assets/6587c88b-856b-41a1-8c6a-1ed0b2a85b56" />

#### 1.3 Install Solana CLI
Jalankan perintah installer resmi Solana:
```bash
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
```
<img width="845" height="239" alt="image" src="https://github.com/user-attachments/assets/884794a8-4380-476b-9340-f329464dd641" />

Setelah selesai, tambahkan Solana ke PATH. Buka file ~/.bashrc dan tambahkan baris berikut di bagian paling bawah:
```bash
echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
<img width="844" height="98" alt="image" src="https://github.com/user-attachments/assets/8a98ad00-d498-4692-9f87-8690064f37b9" />

Verifikasi:
```bash
solana --version
# Contoh output: solana-cli 4.0.2 (src:549805f3; feat:6ff76655, client:Agave)
```
<img width="843" height="83" alt="image" src="https://github.com/user-attachments/assets/c7d20e58-93b9-408a-b5c8-16894d131fa6" />

#### 1.4 Install Anchor Framework
Solana mengembangkan framework DApp bernama Anchor. Install menggunakan script resmi:
```bash
curl -sSfL https://raw.githubusercontent.com/solana-foundation/anchor/master/avm/install | sh
```
<img width="840" height="182" alt="image" src="https://github.com/user-attachments/assets/ff8b708d-d85c-4bb1-8d2a-8894b3e053be" />

Tambahkan AVM ke PATH:
```bash
echo 'export PATH="$HOME/.avm/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
Verifikasi instalasi AVM dan Anchor:
```bash
avm --version
anchor --version
# Contoh output: anchor-cli 1.0.2
```
<img width="842" height="181" alt="image" src="https://github.com/user-attachments/assets/c06c598c-aeaf-42a7-981a-0436ed5d9df4" />

#### 1.5 Ringkasan Verifikasi Semua Tools
Jalankan semua perintah berikut sekaligus untuk memastikan semua terinstall dengan benar:
```bash
rustc --version
solana --version
anchor --version
node --version
yarn --version
```
Contoh output yang diharapkan:
```bash
rustc 1.96.0 (ac68faa20 2026-05-25)
solana-cli 4.0.2 (src:549805f3; feat:6ff76655, client:Agave)
anchor-cli 1.0.2
v24.17.0
1.22.22
```
<img width="843" height="200" alt="image" src="https://github.com/user-attachments/assets/2522d298-c049-4a17-91b7-1cf395100f45" />

#### 1.6  Membuat Wallet Address Solana
Kita akan membuat 2 buah address wallet. Jalankan perintah berikut:
#### Address ke-1:
```bash
solana-keygen new --outfile ~/solana-address-01.json
```
Saat diminta BIP39 Passphrase, bisa tekan Enter untuk kosong, atau isi passphrase pilihan. Catat baik-baik:
* BIP39 Passphrase yang dibuat
* Seed phrase (12 kata) yang ditampilkan di bagian bawah

#### Address 2
```bash
solana-keygen new --outfile ~/solana-address-02.json
```
Lakukan hal yang sama, catat passphrase dan seed phrase-nya.

---

### 2. Konsensus pada Solana Blockchain
Kunci efisiensi Solana terletak pada kombinasi PoH + PoS:

| Komponen | Fungsi |
|---|---|
|**Proof of History (PoH)** | Mencatat urutan waktu transaksi secara kriptografis, bertindak sebagai "jam digital"|
| **Proof of Stake (PoS)** | Memilih validator sebagai leader berdasarkan jumlah stake, serta mekanisme voting|

Dengan PoH, validator tidak perlu saling berkomunikasi intensif hanya untuk menyepakati waktu transaksi, sehingga finalisasi jauh lebih cepat.

---

### 3. Memperoleh SOL dari Devnet
#### 3.1 Atur Solana CLI ke Devnet
```bash
solana config set --url devnet
```
<img width="840" height="124" alt="image" src="https://github.com/user-attachments/assets/30d3db2b-239e-4fab-a1d7-8a3ed3c6a9e7" />

Verifikasi konfigurasi:
```bash
cat ~/.config/solana/cli/config.yml
```
Output yang diharapkan:
```bash
json_rpc_url: https://api.devnet.solana.com
websocket_url: ''
keypair_path: /home/<user>/.config/solana/id.json
commitment: confirmed
```
<img width="839" height="212" alt="image" src="https://github.com/user-attachments/assets/efdcd88e-3ac4-4c19-8f21-d05873a718e1" />

#### 3.2 Set Wallet Default
Copy file address ke-1 sebagai wallet default Solana:
```bash
cp ~/solana-address-01.json ~/.config/solana/id.json
```
Verifikasi wallet default aktif:
```bash
solana address
solana balance
```
<img width="840" height="185" alt="image" src="https://github.com/user-attachments/assets/0f16bf43-2581-4cb7-b763-a6a4e4e17ff9" />

#### 3.3 Airdrop SOL dari Faucet
SOL Devnet bisa diminta melalui Solana CLI, tetapi sering gagal. Cara yang lebih andal adalah lewat web faucet:
* Buka browser, akses https://faucet.solana.com/
* Klik "Connect your GitHub" untuk login (diperlukan agar bisa request lebih banyak SOL)
* Masukkan address wallet ke-1 kamu di kolom Wallet Address
* Pilih jumlah 5 SOL
* Klik "Confirm Airdrop"

Setelah airdrop berhasil, cek saldo:
```bash
solana balance
# Output: 5 SOL
```
<img width="843" height="112" alt="image" src="https://github.com/user-attachments/assets/cc0c3e99-9e6b-422f-a92e-933ce45f9049" />

---

### 4. Alur Konsensus pada Solana Blockchain
Berikut alur lengkap konsensus Solana dari transaksi hingga finalized:
```bash
TRANSAKSI DIBUAT (Transfer SOL)
        │
        ▼
PROOF OF HISTORY (PoH)
  - Memberi urutan waktu kriptografis
  - Menentukan urutan transaksi
        │
        ▼
SLOT (kelompok waktu PoH)
        │
        ▼
PROOF OF STAKE (PoS)
  - Validator dipilih jadi LEADER
    berdasarkan jumlah stake
        │
        ▼
LEADER MEMBUAT BLOK
  (urut transaksi dalam slot)
        │
        ▼
VALIDATOR VOTING (PoS)
  - Validator lain memeriksa blok
  - Vote YES / NO
  - Bobot vote = jumlah stake
        │
        ▼
CONFIRMED — mayoritas setuju
        │
        ▼
FINALIZED — KONSENSUS TERCAPAI
  (tidak bisa diubah)
```
#### 4.1 Transfer SOL Antar Wallet
Kirim 0.5 SOL dari wallet ke-1 ke wallet ke-2. Ganti <ALAMAT_WALLET_02> dengan hasil perintah solana address -k ~/solana-address-02.json:
```bash
solana transfer <ALAMAT_WALLET_02> 0.5 --allow-unfunded-recipient
```
Output yang diharapkan:
```bash
Signature: 4f6mhPRhMuKhfas5UcC8BE4mKYbwrpe...
```
<img width="841" height="139" alt="image" src="https://github.com/user-attachments/assets/0b2b700c-733a-40b1-88d6-f717f7907b87" />

Cek saldo wallet ke-2:
```bash
solana balance <ALAMAT_WALLET_02>
# Output: 0.5 SOL
```
<img width="843" height="85" alt="image" src="https://github.com/user-attachments/assets/f3c50cd1-befc-450b-9e64-f27f8c9068f8" />

#### 4.2 Melihat Slot (Proof of History)
Slot adalah hasil pengurutan waktu transaksi oleh PoH:
```bash
solana slot
# Contoh output: 471127276
```
<img width="844" height="119" alt="image" src="https://github.com/user-attachments/assets/52440901-b1ef-4f0f-9a6b-61f94bfdf1d1" />

#### 4.3 Tampilkan Detail Blok dari Slot
Gunakan nomor slot yang didapat dari perintah sebelumnya:
```bash
solana block <NOMOR_SLOT>
# Contoh: solana block 471127276
```
Output akan menampilkan detail blok seperti:
* Slot dan Parent Slot
* Blockhash dan Previous Blockhash
* Block Time
* Daftar transaksi beserta status dan fee-nya

#### 4.4 Menampilkan Daftar Validators
```bash
solana validators
```
Perhatikan kolom Active Stake, semakin besar stake, semakin besar pengaruh validator dalam voting konsensus.

Tampilkan leader validator untuk slot tertentu (ganti dengan nomor slot):
```bash
solana leader-schedule | grep <NOMOR_SLOT>
```
<img width="845" height="111" alt="image" src="https://github.com/user-attachments/assets/84bc155e-d515-4326-a23e-f6c04ac267c2" />

#### 4.5 Melihat Vote Accounts via RPC API
```bash
curl https://api.devnet.solana.com \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "getVoteAccounts"
  }'
```
Dari hasil JSON tersebut, field penting yang membuktikan voting telah terjadi adalah lastVote — menunjukkan slot terakhir yang divote oleh masing-masing validator.

---

### 5. Periksa Hasil Akhir Proses
#### 5.1 Konfirmasi Status Transaksi via CLI
Gunakan signature yang didapat dari langkah transfer SOL:
```bash
solana confirm <SIGNATURE_TRANSAKSI>
# Output yang diharapkan: Finalized
```
<img width="846" height="135" alt="image" src="https://github.com/user-attachments/assets/6bd9388d-25ed-41e7-9e93-274fc910ef9b" />

Finalized berarti konsensus telah dicapai dan transaksi sudah ditulis permanen ke blok.

#### 5.2 Verifikasi melalui Solana Explorer (Web)
* Buka browser, akses https://explorer.solana.com
* Pastikan jaringan sudah diset ke Devnet (lihat pojok kanan atas — klik jika masih menunjukkan Mainnet, lalu pilih Devnet)
* Paste signature transaksi ke kolom Search
* Tekan Enter
<img width="1586" height="722" alt="image" src="https://github.com/user-attachments/assets/296104ee-f507-4000-8d23-61bb1f35c800" />

Finalized (MAX Confirmations) membuktikan bahwa konsensus PoH + PoS telah tercapai dan transaksi tidak dapat diubah lagi.
