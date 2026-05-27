# Modul 10
## Data Terdistribusi

### 0. Pengantar
YugabyteDB adalah distributed SQL database yang digunakan untuk mempelajari konsep data terdistribusi. Kita akan menggunakan versi LTS (Long-Term Support) karena lebih stabil.

---

### 1. Download YugabyteDB
```bash
mkdir -p ~/master/dbms/yugabytedb
cd ~/master/dbms/yugabytedb

wget https://software.yugabyte.com/releases/2025.2.3.0/yugabyte-2025.2.3.0-b149-linux-x86_64.tar.gz
```
<img width="1250" height="333" alt="image" src="https://github.com/user-attachments/assets/3dcb80c2-f3ba-41e3-b794-12d7bfefdb4f" />

---

### 2. Ekstraksi
```bash
tar xvf yugabyte-2025.2.3.0-b149-linux-x86_64.tar.gz
```
<img width="1249" height="668" alt="image" src="https://github.com/user-attachments/assets/8b77d59e-7196-48a9-bb5e-b52cb45670dd" />

---

### 3. Pindahkan ke subdirektori software
```bash
mkdir -p ~/software/dbms
mv yugabyte-2025.2.3.0 ~/software/dbms/
ls -la ~/software/dbms/yugabyte-2025.2.3.0/
```
<img width="1244" height="331" alt="image" src="https://github.com/user-attachments/assets/5bbf01a1-4688-40d1-9564-a99b97209def" />

---

### 4. Buat symlink
```bash
cd ~/software/dbms
ln -s yugabyte-2025.2.3.0 yugabytedb
ls -la yugabyte*
```
<img width="1248" height="374" alt="image" src="https://github.com/user-attachments/assets/61b08c49-f4a9-445d-b3ab-e3fefcb89520" />
Hasilnya: yugabytedb -> yugabyte-2025.2.3.0

---

### 5. Jalankan post_install.sh
```bash
cd ~/software/dbms/yugabytedb/bin
./post_install.sh
```
<img width="1250" height="582" alt="image" src="https://github.com/user-attachments/assets/fc975a4d-67cb-414a-8ec3-b545f146078c" />
Output akhir: INSTALL PASSED

---

### 6. Set ulimit
Edit file /etc/security/limits.conf:

```bash
sudo nano /etc/security/limits.conf
```
Tambahkan baris-baris berikut di bagian bawah:
```bash
* - core unlimited
* - data unlimited
* - fsize unlimited
* - sigpending 119934
* - memlock 64
* - rss unlimited
* - nofile 1048576
* - msgqueue 819200
* - stack 8192
* - cpu unlimited
* - nproc 12000
* - locks unlimited
```
<img width="1246" height="705" alt="image" src="https://github.com/user-attachments/assets/59ca3f4c-c47e-4b62-a3a0-0de74124bd59" />

Simpan (Ctrl+O, Enter, Ctrl+X). Restart WSL dari PowerShell:
```bash
wsl --shutdown
```
Buka kembali Debian.

---

### 7. Buat file environment variables
```bash
cat > ~/env/bash/dbms/yugabytedb << 'EOF'
export PATH="$HOME/software/dbms/yugabytedb/bin:$HOME/software/dbms/yugabytedb/postgres/bin:$HOME/software/dbms/yugabytedb/tools:$PATH"
EOF
```
Buat direktorinya dulu jika belum ada:
```bash
mkdir -p ~/env/bash/dbms
```
Ulangi perintah cat di atas, lalu verifikasi:
```bash
cat ~/env/bash/dbms/yugabytedb
```
<img width="966" height="150" alt="image" src="https://github.com/user-attachments/assets/d1c1b1cc-a3bc-4736-a6bf-e833efde8d74" />

---

### 8. Source env setiap akan menggunakan YugabyteDB
```bash
source ~/env/bash/dbms/yugabytedb
```
Verifikasi:
```bash
yugabyted --help
```
<img width="1185" height="589" alt="image" src="https://github.com/user-attachments/assets/28e959fd-c8c7-48ea-bb23-ef9f141e413f" />

Agar tidak perlu source manual tiap sesi, tambahkan ke ~/.bashrc:
```bash
echo 'source ~/env/bash/dbms/yugabytedb' >> ~/.bashrc
```
<img width="1179" height="156" alt="image" src="https://github.com/user-attachments/assets/3c3d6a6c-7c58-4cdf-ace9-6cc77b5d4921" />

---

### 9. Membuat Kluster 3 Node
Pastikan IP alias aktif
```bash
sudo ip addr add 127.0.0.2/8 dev lo 2>/dev/null || true
sudo ip addr add 127.0.0.3/8 dev lo 2>/dev/null || true
```
#### Node 1
```bash
yugabyted start \
  --advertise_address=127.0.0.1 \
  --base_dir=${HOME}/var/node1 \
  --cloud_location=aws.us-east-2.us-east-2a
```
<img width="1186" height="589" alt="image" src="https://github.com/user-attachments/assets/293ebd25-ce01-43af-a144-0781aca2664a" />

Cek status:
```bash
yugabyted status --base_dir=${HOME}/var/node1
```
<img width="1183" height="287" alt="image" src="https://github.com/user-attachments/assets/b1ebb213-1031-4206-a511-e3f0256c3590" />

#### Node 2
```bash
yugabyted start \
  --advertise_address=127.0.0.2 \
  --base_dir=${HOME}/var/node2 \
  --cloud_location=aws.us-east-2.us-east-2b \
  --join 127.0.0.1
```
<img width="1184" height="669" alt="image" src="https://github.com/user-attachments/assets/3927cfcc-5aac-4765-b87c-43f8fe0aa7c2" />

Cek status:
```bash
yugabyted status --base_dir=${HOME}/var/node2
```
<img width="1182" height="286" alt="image" src="https://github.com/user-attachments/assets/2d394e5e-352f-482f-a12d-05e8150c838e" />

#### Node 3
```bash
yugabyted start \
  --advertise_address=127.0.0.3 \
  --base_dir=${HOME}/var/node3 \
  --cloud_location=aws.us-east-2.us-east-2c \
  --join 127.0.0.1
```
<img width="1181" height="662" alt="image" src="https://github.com/user-attachments/assets/03557406-5472-4eee-8a22-cda272de7242" />

Cek status:
```bash
yugabyted status --base_dir=${HOME}/var/node3
```
<img width="1181" height="283" alt="image" src="https://github.com/user-attachments/assets/224bd522-d06b-432e-8024-156eca3de92c" />
Status Replication Factor: 3 di Node 3 menandakan kluster sudah terbentuk dengan baik.

#### Konfigurasi data placement dengan fault tolerance per zone
```bash
yugabyted configure data_placement \
  --base_dir=${HOME}/var/node1 \
  --fault_tolerance=zone
```
output yang diharapkan:
```bash
Status           : Configuration successful.
Fault Tolerance  : Primary Cluster can survive at most any 1 availability zone failure
```
<img width="1186" height="203" alt="image" src="https://github.com/user-attachments/assets/b8e98816-e2a0-402b-b8af-425407386f28" />

#### Akses UI Web
Buka browser di Windows, akses:
```bash
http://localhost:15433
```
Tampilan Universe Dashboard akan menampilkan 3 nodes aktif dengan Replication Factor 3.
<img width="1596" height="820" alt="image" src="https://github.com/user-attachments/assets/bcab3b7e-0e4a-465e-8807-15ddaaede694" />

---

### 10. Sharding
Masuk ke YSQL shell (pilih salah satu node):
```bash
ysqlsh -h 127.0.0.1 -U yugabyte -d yugabyte
```
<img width="1182" height="140" alt="image" src="https://github.com/user-attachments/assets/f001a538-c2d7-46a8-8c3a-90e38c4f1122" />

#### Range Sharding — Tabel tanpa split (1 tablet)
```bash
CREATE TABLE user_range (id INT, name VARCHAR(50), PRIMARY KEY(id ASC));

INSERT INTO user_range VALUES
  (1,'user range 1'),(2,'user range 2'),(3,'user range 3'),
  (4,'user range 4'),(5,'user range 5'),(6,'user range 6');

SELECT * FROM user_range;
```
<img width="1184" height="395" alt="image" src="https://github.com/user-attachments/assets/781708f2-025e-4d36-a5bd-88dc0e334171" />

Cek jumlah tablet (hasilnya 1 tablet):
```bash
SELECT * FROM yb_table_properties('user_range'::regclass);
```
<img width="1186" height="176" alt="image" src="https://github.com/user-attachments/assets/6689c83a-3f3d-4e43-8487-9e0e276879a8" />

#### Range Sharding — Tabel dengan SPLIT (4 tablet)
```bash
CREATE TABLE users (
  user_id INT,
  username VARCHAR,
  PRIMARY KEY (user_id ASC)
) SPLIT AT VALUES ((3), (6), (9));

INSERT INTO users VALUES
  (1,'user 1'),(2,'user 2'),(3,'user 3'),
  (4,'user 4'),(5,'user 5'),(6,'user 6');
```
<img width="1184" height="237" alt="image" src="https://github.com/user-attachments/assets/f03db351-1818-4cb3-a8e4-c87f31a0a567" />

Cek jumlah tablet (hasilnya 4 tablet):
```bash
SELECT * FROM yb_table_properties('users'::regclass);
```
<img width="1183" height="122" alt="image" src="https://github.com/user-attachments/assets/e147f474-c6d1-43b7-8210-3452e2fe0467" />
Pembagian tabletnya:

* Tablet 1 → id ≤ 3
* Tablet 2 → id 4–6
* Tablet 3 → id 7–9
* Tablet 4 → id ≥ 10

---

### 11. Uji EXPLAIN pada Range Sharding

#### Query semua baris (Rows Scanned = 6):
```bash
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_range;
```
<img width="1182" height="394" alt="image" src="https://github.com/user-attachments/assets/0ae7b549-0556-4e92-a9d3-0070e2ade0e2" />

#### Query satu baris (Rows Scanned = 1, Index Scan):
```bash
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_range WHERE id=1;
```
<img width="1183" height="430" alt="image" src="https://github.com/user-attachments/assets/0829a772-a2be-4f77-921f-8e6ed34fab8e" />

#### Query range (Rows Scanned = 3, hanya membaca yang sesuai):
```bash
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_range WHERE id > 2 AND id < 6;
```
<img width="1184" height="427" alt="image" src="https://github.com/user-attachments/assets/45bd0d4d-584f-414f-86c4-63fe34ad673b" />

#### Simpulan range sharding: 
Sangat efisien untuk query berbasis range — hanya membaca baris yang benar-benar diperlukan.

#### Hash Sharding
```bash
CREATE TABLE user_hash (id INT, name VARCHAR, PRIMARY KEY(id HASH));

INSERT INTO user_hash VALUES
  (1,'user hash 1'),(2,'user hash 2'),(3,'user hash 3'),
  (4,'user hash 4'),(5,'user hash 5'),(6,'user hash 6');

SELECT * FROM user_hash;
```
<img width="1178" height="395" alt="image" src="https://github.com/user-attachments/assets/2d87399f-774e-415c-986a-087fa227231b" />

#### Uji EXPLAIN:
```bash
-- Query semua baris
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_hash;

-- Query satu baris (efisien)
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_hash WHERE id=1;

-- Query range (TIDAK efisien — scan semua baris)
EXPLAIN (ANALYZE, DIST, COSTS OFF) SELECT * FROM user_hash WHERE id>2 AND id<6;
```
<img width="1186" height="400" alt="image" src="https://github.com/user-attachments/assets/8bdc5778-18d6-4ee9-aadb-c4f4165f6c83" />
<img width="1180" height="409" alt="image" src="https://github.com/user-attachments/assets/8f0d30fc-3ebc-4b32-85f6-18e1a20a3d32" />
<img width="1181" height="410" alt="image" src="https://github.com/user-attachments/assets/8618e39f-cca0-407e-8bdb-dac27a37f2f2" />

#### Simpulan hash sharding: 
Efisien untuk point lookup (satu key), tetapi tidak efisien untuk query range karena harus scan semua tablet.

---

### 12. Shutdown YugabyteDB
Matikan semua node secara berurutan (mulai dari node tertinggi):
```bash
yugabyted stop --base_dir=${HOME}/var/node3
yugabyted stop --base_dir=${HOME}/var/node2
yugabyted stop --base_dir=${HOME}/var/node1
```
<img width="1186" height="79" alt="image" src="https://github.com/user-attachments/assets/c89d063c-7107-4349-834a-6c44c1bff935" />



















