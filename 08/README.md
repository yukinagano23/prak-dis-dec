# Modul 8
## Arsitektur Microservices untuk Sistem Terdistribusi

### 0. Prasyarat: Install uv
Buka PowerShell (sebagai Administrator), lalu jalankan:
```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
<img width="841" height="199" alt="image" src="https://github.com/user-attachments/assets/725f1ff2-a4b1-448e-8184-cfb67b025675" />

Setelah selesai, tutup dan buka ulang PowerShell, lalu verifikasi:
```bash
uv --version
```
<img width="844" height="126" alt="image" src="https://github.com/user-attachments/assets/7fe93057-ec0b-4edc-9402-b1a9146c104e" />

---

### 1. Setup Python dengan uv
```bash
uv python list
uv python install 3.14
mkdir C:\microservices-sdm
cd C:\microservices-sdm
uv venv --python 3.14
.venv\Scripts\activate
```
<img width="841" height="310" alt="image" src="https://github.com/user-attachments/assets/f11e38ca-5216-45d2-b3ea-25ea47f5248d" />
<img width="842" height="85" alt="image" src="https://github.com/user-attachments/assets/f6685d7e-5087-4e7d-96f5-08f4adedef63" />
<img width="838" height="191" alt="image" src="https://github.com/user-attachments/assets/4f2f9451-aa02-4b09-bc7f-cec7fd9a6f36" />
<img width="843" height="56" alt="image" src="https://github.com/user-attachments/assets/fb402703-26f4-4f19-b5f6-6f60b13d661a" />
<img width="844" height="37" alt="image" src="https://github.com/user-attachments/assets/97781c99-2cc9-4c97-92d4-2e1f264e322c" />
Setelah aktif, prompt akan berubah jadi (src) atau nama folder di depan.

---

### 2. Install Paket yang Dibutuhkan
```bash
uv pip install "fastapi[standard]" sqlmodel
```
<img width="845" height="708" alt="image" src="https://github.com/user-attachments/assets/66752eca-9177-48a4-a3f3-6d35db445982" />
SQLite sudah built-in di Windows bersama Python, tidak perlu install terpisah.

---

### 3. Buat Database SQLite
Buka SQLite shell. Karena SQLite sudah tersedia via Python, gunakan cara ini:
```bash
python -c "
import sqlite3
conn = sqlite3.connect('departemen-sdm.db')
cur = conn.cursor()
cur.execute('''CREATE TABLE IF NOT EXISTS sdm (
    id INTEGER PRIMARY KEY,
    npp CHAR(6),
    nama VARCHAR(50)
)''')
cur.execute(\"INSERT INTO sdm (npp, nama) VALUES ('112233', 'Karyawan 1')\")
cur.execute(\"INSERT INTO sdm (npp, nama) VALUES ('223344', 'Karyawan 2')\")
cur.execute(\"INSERT INTO sdm (npp, nama) VALUES ('334455', 'Karyawan 3')\")
conn.commit()
print('Database berhasil dibuat!')
cur.execute('SELECT * FROM sdm')
print(cur.fetchall())
conn.close()
"
```
<img width="848" height="459" alt="image" src="https://github.com/user-attachments/assets/52cb6dc3-f78d-4135-8865-3508ee126f5b" />

#### Masalah: Konflik Tanda Kutip di PowerShell
PowerShell kesulitan memproses perintah multi-baris dengan tanda kutip campuran (' dan ") lewat python -c "...".
Solusi Terbaik: Buat File Python Terpisah
Daripada pakai python -c, lebih baik buat file .py langsung:
#### Buat file init_db.py:
```bash
notepad init_db.py
```
perintah di atas akan membuka notepad, setelah terbuka, ketikan perintah berikut ini:
```bash
import sqlite3

conn = sqlite3.connect('departemen-sdm.db')
cur = conn.cursor()

cur.execute('''CREATE TABLE IF NOT EXISTS sdm (
    id INTEGER PRIMARY KEY,
    npp CHAR(6),
    nama VARCHAR(50)
)''')

cur.execute("INSERT INTO sdm (npp, nama) VALUES ('112233', 'Karyawan 1')")
cur.execute("INSERT INTO sdm (npp, nama) VALUES ('223344', 'Karyawan 2')")
cur.execute("INSERT INTO sdm (npp, nama) VALUES ('334455', 'Karyawan 3')")

conn.commit()
print('Database berhasil dibuat!')

cur.execute('SELECT * FROM sdm')
print(cur.fetchall())

conn.close()
```
<img width="994" height="473" alt="image" src="https://github.com/user-attachments/assets/a72917e4-b641-478f-8ecf-d30f49983605" />
Lalu jalankan:

```bash
python init_db.py
```
<img width="842" height="98" alt="image" src="https://github.com/user-attachments/assets/b40bfb6c-0c00-40c7-ac6d-d1450541915d" />

---

### 4. Buat Source Code (service.py)
Buat file service.py di folder C:\microservices-sdm\:

ketikan:
```bash
notepad service.py
```
setelah notepad terbuka, masukan kode berikut:
```bash
# service.py
from typing import List, Optional
from fastapi import Depends, FastAPI, HTTPException
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Sdm(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    npp: str
    nama: str


engine = create_engine("sqlite:///departemen-sdm.db")


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.get("/sdm/", response_model=List[Sdm])
def read_sdm(session: Session = Depends(get_session)):
    statement = select(Sdm)
    results = session.exec(statement).all()
    return results
```
<img width="842" height="42" alt="image" src="https://github.com/user-attachments/assets/e84df0f0-b383-411d-8555-2590a12deede" />
<img width="991" height="476" alt="image" src="https://github.com/user-attachments/assets/08bc4bb1-5320-47b3-a150-565d5afe9275" />

---

### 5. Jalankan Service
```bash
uvicorn service:app --reload
```
Output yang diharapkan:
```bash
←[32mINFO←[0m:     Will watch for changes in these directories: ['C:\\microservices-sdm']
←[32mINFO←[0m:     Uvicorn running on ←[1mhttp://127.0.0.1:8000←[0m (Press CTRL+C to quit)
←[32mINFO←[0m:     Started reloader process [←[36m←[1m4340←[0m] using ←[36m←[1mWatchFiles←[0m
←[32mINFO←[0m:     Started server process [←[36m10296←[0m]
←[32mINFO←[0m:     Waiting for application startup.
←[32mINFO←[0m:     Application startup complete.
```
<img width="841" height="129" alt="image" src="https://github.com/user-attachments/assets/67b46108-6ecb-472e-993e-814d2255a398" />

--

### 6. Uji Endpoint
Buka jendela Powershell baru, ketikan perintah berikut:
```bash
cd C:\microservices-sdm
.venv\Scripts\activate
curl http://localhost:8000/sdm/
```
buka:http://localhost:8000/sdm/ Via curl di PowerShell:
```bash
curl http://localhost:8000/sdm/
```
<img width="840" height="371" alt="image" src="https://github.com/user-attachments/assets/5838ce44-1487-42da-a65d-f08a0e3ee8fe" />
Via curl di Command Prompt (cmd)

```bash
curl http://localhost:8000/sdm/
```
<img width="960" height="128" alt="image" src="https://github.com/user-attachments/assets/2ccbafcf-31c0-4974-822a-d819238b4230" />

#### Bonus: Swagger UI otomatis dari FastAPI
Buka di browser: 
```bash
http://localhost:8000/docs
```
<img width="1593" height="547" alt="image" src="https://github.com/user-attachments/assets/a6dd8a11-bc55-465a-8365-5e03efd1a715" />

---

## Tugas Modul 8 — Tabel Barang
### Langkah 1: Buat file init_barang.py untuk membuat tabel dan mengisi 5 data
```bash
notepad init_barang.py
```
setelah notepad terbukan, isikan:
```bash
import sqlite3

conn = sqlite3.connect('toko-barang.db')
cur = conn.cursor()

cur.execute('''CREATE TABLE IF NOT EXISTS barang (
    id INTEGER PRIMARY KEY,
    kode CHAR(6),
    nama VARCHAR(100),
    stok INT,
    tersedia BOOLEAN,
    harga FLOAT
)''')

cur.execute("INSERT INTO barang (kode, nama, stok, tersedia, harga) VALUES ('BRG001', 'Laptop ASUS', 10, 1, 8500000.50)")
cur.execute("INSERT INTO barang (kode, nama, stok, tersedia, harga) VALUES ('BRG002', 'Mouse Logitech', 25, 1, 175000.00)")
cur.execute("INSERT INTO barang (kode, nama, stok, tersedia, harga) VALUES ('BRG003', 'Keyboard Mechanical', 0, 0, 650000.75)")
cur.execute("INSERT INTO barang (kode, nama, stok, tersedia, harga) VALUES ('BRG004', 'Monitor 24 inch', 5, 1, 2300000.00)")
cur.execute("INSERT INTO barang (kode, nama, stok, tersedia, harga) VALUES ('BRG005', 'Headset Gaming', 0, 0, 450000.99)")

conn.commit()
print('Database toko-barang berhasil dibuat!')

cur.execute('SELECT * FROM barang')
for row in cur.fetchall():
    print(row)

conn.close()
```
<img width="994" height="618" alt="image" src="https://github.com/user-attachments/assets/044df502-9e7a-47f3-be4a-172666842c85" />

jalankan:
```bash
python init_barang.py
```
<img width="841" height="155" alt="image" src="https://github.com/user-attachments/assets/a399e236-68db-4541-834b-3e99f7b4613f" />

---

### Langkah 2: Buat file service_barang.py
```bash
notepad service_barang.py
```
setelah itu isikan:
```bash
# service_barang.py
from typing import List
from fastapi import Depends, FastAPI, HTTPException
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Barang(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    kode: str
    nama: str
    stok: int
    tersedia: bool
    harga: float


engine = create_engine("sqlite:///toko-barang.db")


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.get("/barang/", response_model=List[Barang])
def read_barang(session: Session = Depends(get_session)):
    statement = select(Barang)
    results = session.exec(statement).all()
    return results
```
<img width="994" height="608" alt="image" src="https://github.com/user-attachments/assets/a2a3236b-e98d-4779-bb64-59aae6a9501c" />

---

### Langkah 3: Jalankan Server
Di PowerShell 1:
```bash
uvicorn service_barang:app --reload
```
<img width="840" height="115" alt="image" src="https://github.com/user-attachments/assets/f542c0a6-1b73-4959-b911-4b5e68d057b8" />

---

### Langkah 4: Uji dengan curl
Di PowerShell 2 (buka jendela baru):
```bash
cd C:\microservices-sdm
.venv\Scripts\activate
curl http://localhost:8000/barang/
```
Output yang diharapkan:
<img width="842" height="387" alt="image" src="https://github.com/user-attachments/assets/661feafc-b2fa-4d16-bb18-2b1839653645" />
Bisa juga cek via browser di :
```bash
http://localhost:8000/barang/
```
<img width="1600" height="284" alt="image" src="https://github.com/user-attachments/assets/35f0d0cd-34cd-422b-8af9-b97414af7c36" />

atau Swagger UI di :
```bash
http://localhost:8000/docs.
```
<img width="1595" height="618" alt="image" src="https://github.com/user-attachments/assets/b967a679-1800-4dcd-9ece-59f2ad4cd92c" />



