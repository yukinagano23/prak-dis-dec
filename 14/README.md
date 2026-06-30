# Modul 13
## Smart Contract pada Blockchain

#### Install Rust via rustup
Solana menggunakan Rust. Install menggunakan rustup (package manager resmi Rust).
```bash
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)"
# Tambahkan ke PATH
echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
solana --version
# Contoh output: solana-cli 2.x.x
```
<img width="966" height="265" alt="image" src="https://github.com/user-attachments/assets/2fd80e81-0c71-4440-82d4-19b6f0b30c35" />

#### Install Node.js, Yarn & Anchor 
Anchor Framework membutuhkan Node.js dan Yarn. Install menggunakan nvm.
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
npm install -g yarn
# Install Anchor CLI
cargo install --git https://github.com/coral-xyz/anchor avm --locked
avm install latest && avm use latest
anchor --version
# Contoh output: anchor-cli 1.1.2
```
<img width="958" height="543" alt="image" src="https://github.com/user-attachments/assets/beab0c44-ba24-4637-a3a3-1abb57b00814" />
<img width="964" height="343" alt="image" src="https://github.com/user-attachments/assets/b15f3bf9-5e19-4d45-920e-ca6fa096c60b" />
<img width="961" height="122" alt="image" src="https://github.com/user-attachments/assets/83eb64ed-4be0-4271-aeb9-dbf9b6537272" />

---

### Bagian 1 : Smart Contract Native Rust
#### 1. Buat proyek baru & tambahkan pustaka Solana
Buat proyek library Rust baru menggunakan cargo, lalu masuk ke direktori proyek dan tambahkan dependensi solana-program.
```bash
cargo new hello_solana --lib
cd hello_solana
cargo add solana-program
# Output: Adding solana-program v4.0.0 to dependencies
```
<img width="963" height="273" alt="image" src="https://github.com/user-attachments/assets/623eaa38-3a6a-4183-b7a3-f406776fd3c0" />

#### 2. Edit Cargo.toml, tambahkan konfigurasi [lib]
Buka file Cargo.toml dan tambahkan bagian [lib] dengan crate-type sebagai berikut. Gunakan nano, vim, atau editor lain.
```bash
nano Cargo.toml
```
Isi Cargo.toml setelah diedit:
```bash
[package]
name = "hello_solana"
version = "0.1.0"
edition = "2024"

[lib]
crate-type = ["cdylib", "lib"]

[dependencies]
solana-program = "4.0.0"
```
<img width="961" height="193" alt="image" src="https://github.com/user-attachments/assets/6280f46f-e887-409d-a05c-65b68ec12ec1" />

#### 3. Ganti isi src/lib.rs dengan source code smart contract
Hapus isi default src/lib.rs dan ganti dengan kode smart contract "Hello World" berikut.
```bash
nano src/lib.rs
```
Isi src/lib.rs:
```bash
use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult,
    msg, pubkey::Pubkey,
};

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, world!");
    Ok(())
}
```
<img width="962" height="277" alt="image" src="https://github.com/user-attachments/assets/16cc0261-96c4-4ede-a8fc-4f3002f6b6f9" />

#### 4. Build smart contract menggunakan cargo build-sbf
Perintah ini spesifik untuk Solana. Saat pertama kali dijalankan, cargo akan mengunduh dan menginstall semua tools yang diperlukan (bisa memakan waktu beberapa menit).
```bash
cargo build-sbf
# Proses kompilasi akan berjalan, output terakhir:
# Finished `release` profile [optimized] target(s) in 12.87s
```
<img width="960" height="369" alt="image" src="https://github.com/user-attachments/assets/e65c18ba-e559-433c-b1f1-2d5c9fd64363" />

#### 5. (Opsional) Hilangkan warning dengan menambahkan [lints.rust]
Jika ingin build tanpa warning, tambahkan bagian berikut di akhir Cargo.toml, lalu build ulang.
```bash
[lints.rust]
unexpected_cfgs = { level = "allow", check-cfg = [
    'cfg(feature, values("custom-heap", "custom-panic"))'
] }
```
<img width="961" height="272" alt="image" src="https://github.com/user-attachments/assets/c62a7460-3362-4471-b3ba-29b7f5832b83" />

lalu build ulang:
```bash
cargo build-sbf
# Output: Finished `release` profile [optimized] target(s) in 0.24s
```
<img width="959" height="73" alt="image" src="https://github.com/user-attachments/assets/8038e355-e52c-4381-bc9b-31de5d884c35" />

#### 6. Cek hasil build di direktori target/deploy
Hasil build ada 2 file di dalam target/deploy/.
```bash
tree target/deploy/
# Output:
# target/deploy/
# ├── hello_solana-keypair.json
# └── hello_solana.so
# 1 directory, 2 files
```
* **hello_solana.so** : hasil kompilasi smart contract siap deploy ke Solana
* **hello_solana-keypair.json** : ID program (kunci publik digunakan sebagai Program ID)
<img width="966" height="125" alt="image" src="https://github.com/user-attachments/assets/a31079c8-9926-4e4a-b948-830868c5a81a" />

Untuk melihat Program ID:
```bash
solana address -k target/deploy/hello_solana-keypair.json
# Contoh output: 5gr7mDSDUFcYsrKyaCFgovf1YoYv9WzGNP3bDaQNNu3d
```
<img width="963" height="55" alt="image" src="https://github.com/user-attachments/assets/cc794eb5-5116-4dee-9fc1-9dcc09e4692f" />

#### 7. Pengujian menggunakan crate litesvm
Tambahkan dependensi litesvm dan solana-sdk sebagai dev-dependencies.
```bash
cargo add litesvm --dev
cargo add solana-sdk --dev
```
<img width="962" height="505" alt="image" src="https://github.com/user-attachments/assets/40cc4b1d-3263-4d6f-a328-3630af854d61" />

Buat direktori tests dan file lib_test.rs di dalamnya:
```bash
mkdir tests
nano tests/lib_test.rs
```
Isi tests/lib_test.rs (sesuaikan path dengan lokasi file di komputer):
```bash
use litesvm::LiteSVM;
use solana_sdk::{
    signature::{read_keypair_file, Signer},
};

#[test]
fn test_hello_solana() {
    // Initialize the test environment
    let mut svm = LiteSVM::new();

    let solana_keypair_path =
        "target/deploy/hello_solana-keypair.json";
    let solana_so_path =
        "target/deploy/hello_solana.so";

    // Read keypair for correct program ID
    let program_keypair =
        read_keypair_file(solana_keypair_path).unwrap();
    let program_id = program_keypair.pubkey();

    // Deploy program ke test environment
    svm.add_program_from_file(program_id, solana_so_path)
        .expect("Failed to deploy program");

    // Always verify
    assert!(svm.get_account(&program_id).unwrap().executable);
}
```
<img width="960" height="448" alt="image" src="https://github.com/user-attachments/assets/71b8e355-749f-4228-8dad-021f573c5cac" />

Jalankan pengujian:
```bash
cargo test -- --show-output
# Output:
# running 1 test
# test test_hello_solana ... ok
# test result: ok. 1 passed; 0 failed
```
<img width="963" height="586" alt="image" src="https://github.com/user-attachments/assets/6202882d-b0e1-47dc-9ac8-d581b4c19ffb" />

#### 8. Deploy ke localhost (local validator)
Ubah konfigurasi jaringan Solana ke lokal terlebih dahulu.
```bash
solana config set -ul
# Output:
# Config File: /home/user/.config/solana/cli/config.yml
# RPC URL: http://localhost:8899
```
<img width="960" height="145" alt="image" src="https://github.com/user-attachments/assets/123a9b80-345a-4f36-96b5-fb090cc1b930" />

Jalankan validator lokal Solana di terminal/shell terpisah:
```bash
solana-test-validator
# Biarkan terminal ini tetap berjalan
```
<img width="964" height="356" alt="image" src="https://github.com/user-attachments/assets/279a8a9e-8571-4ce2-9fd9-2cf32747b179" />

Kembali ke terminal utama, lakukan deploy:
```bash
solana program deploy target/deploy/hello_solana.so
# Output:
# Program Id: TUuMYYZ844vVKAqukoEmUQw7Pmr9S4tr2A6HAMPMLgN
# Signature: 45kxyfwEfzVHDPkh8a3C4...
```
<img width="966" height="134" alt="image" src="https://github.com/user-attachments/assets/153c528c-94c5-4920-bbb1-513249291ddf" />

Untuk memverifikasi via browser, buka:
```bash
https://explorer.solana.com/?cluster=custom&customUrl=http%3A%2F%2Flocalhost%3A8899
```
Cari Program ID yang dihasilkan dari perintah deploy di atas.
<img width="1462" height="766" alt="image" src="https://github.com/user-attachments/assets/7c8ad4f3-343f-46ef-b897-4dad95c0001d" />

---

### Bagian 2 : Smart Contract dengan Anchor Framework
#### 1. Buat proyek baru dengan Anchor
Gunakan anchor init dengan flag --test-template mollusk agar pengujian menggunakan Rust (bukan TypeScript default).
```bash
anchor init --test-template mollusk sc-solana
cd sc-solana
ls -la
# Output: Anchor.toml  Cargo.toml  app  programs  target  ...
```
<img width="961" height="514" alt="image" src="https://github.com/user-attachments/assets/13cf5c16-bd8b-4201-8e9f-1f466dd1f459" />

#### 2. Build proyek Anchor
Jalankan anchor build dari dalam direktori proyek. Proses kompilasi pertama akan memakan waktu beberapa menit.
```bash
anchor build
# Proses kompilasi berlangsung...
# Finished `release` profile [optimized] target(s)
```
Cek hasil build:
```bash
tree target/deploy/
# Output:
# target/deploy/
# ├── sc_solana-keypair.json
# └── sc_solana.so
```
<img width="964" height="122" alt="image" src="https://github.com/user-attachments/assets/4a0640b9-9f17-4869-ad82-9db7cf1a8561" />

#### 3. Jalankan pengujian dengan anchor test
Pastikan local validator masih berjalan, kemudian jalankan perintah test.
```bash
anchor test
# Output:
# running 1 test
# test test_id ... ok
# test result: ok. 1 passed; 0 failed
#
# running 1 test
# test test_initialize ... ok
# test result: ok. 1 passed; 0 failed
```
<img width="960" height="746" alt="image" src="https://github.com/user-attachments/assets/da9b7bca-01c9-4d2a-8415-a54f5a168427" />

#### 4. Deploy ke Devnet
Edit Anchor.toml dan ubah cluster ke Devnet, serta pastikan skip_local_validator = true.
```bash
skip_local_validator = true

[toolchain]

[features]
resolution = true
skip-lint = false

[programs.localnet]
sc_solana = "3gqjic6wHpFxLQatzn7NMDxL7TwiDjtTMQyiC9PL67AS"

[provider]
cluster = "Devnet"
wallet = "~/.config/solana/id.json"

[scripts]
test = "cargo test-sbf"

[hooks]
```
<img width="957" height="370" alt="image" src="https://github.com/user-attachments/assets/a80daf7e-2892-4c1e-ba08-0eaf3a6789ff" />

Pastikan wallet memiliki SOL di Devnet (airdrop jika perlu):
```bash
solana config set --url devnet
solana airdrop 2
```





