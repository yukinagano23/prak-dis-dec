## Modul 7 Cloud Computing (di Windows)

### Pengantar
Cloud Computing menggunakan pendekatan XaaS atau sering juga disebut sebagainEverything as a Service. Dengan menggunakan pendekatan ini,provider dari Cloud Computing menyediakan berbagai sumber daya komputasi dan konsumen mendapatkan sumber daya tersebut dalam bentuk layanan. Meskipun saat ini ada banyak XaaS tetapim secara umum biasanya dibagi menjadi 3:

1. SaaS: Software as a Service
2. PaaS: Platform as a Service
3. IaaS: Infrastructure as a Service

### Tugas
1. Carilah berbagai contoh vendor/komunitas SaaS, PaaS, dan IaaS masing-masing 1 saja.
3. Uraikan apa yang menjadi service dari berbagai vendor untuk berbagai kategori XaaS tersebut.
4. Jelaskan secara umum arsitektur dari XaaS tersebut dalam bentuk visualisasi gambar.
#### Jawaban nomor 1:
1. SaaS : *Google Workspace* --> SaaS menyediakan aplikasi siap pakai melalui internet, misalnya Gmail, Google Docs, dan Google Drive.
2. PaaS (Platform as a Service): *Google App Engine* --> PaaS menyediakan platform untuk developer membuat dan menjalankan aplikasi tanpa harus mengelola server secara langsung.
3. IaaS (Infrastructure as a Service): *Amazon Web Service (AWS) EC2* ---> IaaS menyediakan infrastruktur seperti server virtual, jaringan, dan penyimpanan berbasis cloud.
#### Jawaban nomor 2:
| Kategori                               | Vendor                                                                         | Service yang Disediakan                                                                                                                                                                                                                                                                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **SaaS (Software as a Service)**       | [Google Workspace](https://workspace.google.com?utm_source=chatgpt.com)        | Menyediakan aplikasi berbasis cloud yang siap digunakan melalui internet, seperti Gmail untuk email, Google Docs untuk pengolahan dokumen, Google Sheets untuk spreadsheet, Google Meet untuk video conference, dan Google Drive untuk penyimpanan file online. Pengguna tidak perlu menginstal atau mengelola server sendiri. |
| **PaaS (Platform as a Service)**       | [Google App Engine](https://cloud.google.com/appengine?utm_source=chatgpt.com) | Menyediakan platform dan lingkungan pengembangan aplikasi berbasis cloud. Developer dapat membuat, menguji, menjalankan, dan mengelola aplikasi tanpa harus mengatur sistem operasi, server, atau infrastruktur jaringan secara manual. Mendukung berbagai bahasa pemrograman seperti Python, Java, dan Node.js.               |
| **IaaS (Infrastructure as a Service)** | [Amazon EC2 (AWS)](https://aws.amazon.com/ec2/?utm_source=chatgpt.com)         | Menyediakan infrastruktur virtual berupa server cloud (virtual machine), penyimpanan data, jaringan, firewall, dan resource komputasi lainnya. Pengguna dapat mengatur spesifikasi server sesuai kebutuhan dan memiliki kontrol penuh terhadap sistem operasi maupun aplikasi yang dijalankan.                                 |
#### Jawaban nomor 3:
<img width="1536" height="1024" alt="gambar" src="https://github.com/user-attachments/assets/72139936-03d8-4378-b00f-f3acd449ac19" />

Salah satu hal yang membentuk layanan-layanan tersebut adalah **Containerized App**.
Containerized App (berikutnya akan disingkat dengan **CA**) merupakan aplikasi yang
dimaksudkan untuk dijalankan oleh container (docker, podman, …). Pada contoh kali ini, kita
akan membangun aplikasi blog yang merupakan tutorial dari Flask menjadi CA.

Tutorial dari Flask bisa dilihat di URL:
[https://flask.palletsprojects.com/en/stable/tutorial/](https://flask.palletsprojects.com/en/stable/tutorial/)

Source code untuk tutorial bisa dilihat di:
[https://github.com/pallets/flask/tree/3.1.3/examples/tutorial](https://github.com/pallets/flask/tree/3.1.3/examples/tutorial)

Jika menggunakan git, kerjakan berikut ini. Asumsikan direktori kerja ada di $HOME/sr
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







