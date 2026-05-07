## Modul 7 Cloud Computing (di Windows)

### Langkah 1: Clone Repository Flask
Buka PowerShell atau Windows Terminal, lalu jalankan perintah berikut satu per satu:
```bash
mkdir $HOME\src
cd $HOME\src
git clone https://github.com/pallets/flask
xcopy /E /I flask\examples\tutorial flask-app
cd flask-app
```
contoh jika di jalankan di CMD:
<img width="959" height="636" alt="image" src="https://github.com/user-attachments/assets/c9ccd8f0-403f-4351-ab60-085bd7da37ae" />
Catatan: Perintah cp -R di Linux diganti dengan xcopy /E /I di Windows.

---

### Langkah 2: Membuat Environment Python dengan uv

Install uv terlebih dahulu jika belum ada (via PowerShell):
```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
<img width="844" height="169" alt="image" src="https://github.com/user-attachments/assets/5389b209-9d50-4230-82eb-54d69b0b9c1e" />
Kemudian buat virtual environment dengan Python 3.13 (Python 3.14 masih beta, gunakan versi stabil):

```bash
uv venv --python 3.13
.venv\Scripts\activate
```

<img width="961" height="161" alt="image" src="https://github.com/user-attachments/assets/0fc80408-648d-448e-bee4-b115958a68c4" />

---

### Langkah 3: Install Paket yang Diperlukan

```bash
uv pip install -e .
```
<img width="963" height="288" alt="image" src="https://github.com/user-attachments/assets/a34b10a4-847f-4789-a043-58e34091dd85" />

---

### Langkah 4: Inisialisasi Database & Jalankan Aplikasi
```bash
flask --app flaskr init-db
flask --app flaskr run
```
<img width="966" height="218" alt="image" src="https://github.com/user-attachments/assets/c64c87b4-80a8-46d0-b83c-c8f8534d3705" />
Buka browser dan akses http://localhost:5000
<img width="1600" height="464" alt="image" src="https://github.com/user-attachments/assets/31e2604b-0634-479e-b392-52b0ab9e56de" />

---

### Langkah 5: Membuat Dockerfile

Buat file bernama Dockerfile (tanpa ekstensi) di dalam folder flask-app. Bisa menggunakan Notepad, VS Code, atau perintah berikut:
```bash
FROM python:3.13

RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -e .
RUN flask --app flaskr init-db

EXPOSE 5000
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]
```
Catatan: Ganti python:3.14 menjadi python:3.13 karena versi 3.14 masih dalam tahap development dan image-nya mungkin belum stabil di Docker Hub.
<img width="994" height="407" alt="image" src="https://github.com/user-attachments/assets/4fdb967a-1843-4354-93ab-568cc210faa7" />

---

### Langkah 6: Build Docker Image

Pastikan Docker Desktop sudah berjalan (cek di system tray — ikonnya harus aktif/hijau).
Di Linux perintahnya menggunakan sudo, tapi di Windows tidak perlu sudo karena Docker Desktop sudah menangani permission:
```bash
docker build -f Dockerfile -t flaskr:1.0.0 .
docker image ls
```
<img width="958" height="739" alt="image" src="https://github.com/user-attachments/assets/1bfb1b76-96df-430d-b775-2c72815d8911" />

---

### Langkah 7: Menjalankan Container Docker
```bash
docker run -p 5001:5000 flaskr:1.0.0
```
<img width="962" height="149" alt="image" src="https://github.com/user-attachments/assets/5a797a25-1213-4632-8262-3ebb8f4573fa" />
buka browser dan akses: 

```
http://localhost:5001
```

aplikasi Flask berjalan di dalam container.
<img width="1592" height="444" alt="image" src="https://github.com/user-attachments/assets/5af88004-52b8-48bf-93d4-9ef5f64624cb" />

---

### Langkah 8: Menggunakan Podman (Alternatif Docker)
Podman bisa diinstall di Windows melalui:

```
 https://podman.io/docs/installation#windows
```

Setelah install, perintahnya hampir identik dengan Docker:

```bash
podman build -f Dockerfile -t flaskr:1.0.0 .
podman image ls
podman run -p 5001:5000 flaskr:1.0.0
```
<img width="965" height="735" alt="image" src="https://github.com/user-attachments/assets/f0ef13a0-66f8-48cc-ae9e-3dbda3f0b4fe" />
<img width="964" height="720" alt="image" src="https://github.com/user-attachments/assets/4102c1de-3e90-4bff-abef-6a9e06399ec1" />
<img width="958" height="152" alt="image" src="https://github.com/user-attachments/assets/514084e0-df5e-4d3e-ab3c-95b89f854fce" />
<img width="1577" height="623" alt="image" src="https://github.com/user-attachments/assets/6b2456cf-2bf6-422e-a58d-9885496a8f3f" />







