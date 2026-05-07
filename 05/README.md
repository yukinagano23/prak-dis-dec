praktikum Modul 5 yang disesuaikan untuk pengguna Windows.

1. Install Docker Desktop.
   Unduh dan install Docker Desktop dari: https://docs.docker.com/desktop/setup/install/windows-install/
   Setelah install, pastikan Docker Desktop sudah berjalan (ada ikon paus di system tray). Docker Desktop di Windows sudah mencakup docker dan docker-compose sekaligus, sehingga tidak perlu install terpisah.
   Verifikasi instalasi di Command Prompt atau PowerShell:
   <img width="961" height="101" alt="image" src="https://github.com/user-attachments/assets/52cbe52b-dabe-4b67-ae27-8c8a8f124789" />


2. Install Python Environment Manager. Di Windows, disarankan menggunakan uv atau Miniconda. Panduan ini menggunakan uv. Install uv via PowerShell:
   <img width="963" height="196" alt="image" src="https://github.com/user-attachments/assets/2911a9df-e209-411b-bcc5-55b6a0374d75" />
   Setelah install, tutup dan buka kembali terminal, lalu verifikasi:
   <img width="963" height="107" alt="image" src="https://github.com/user-attachments/assets/3b7de711-7112-4fd0-af13-219b350547f5" />

3. Buat Workspace. Buka Command Prompt atau PowerShell, lalu:
   <img width="962" height="172" alt="image" src="https://github.com/user-attachments/assets/385843c5-5e8c-4538-8cc7-ac4cf97c27e2" />

4. Setup Virtual Environment Python dengan uv
   <img width="962" height="144" alt="image" src="https://github.com/user-attachments/assets/88f2cd8b-4174-48d2-88d7-e7c5a04c0633" />
   Setelah aktivasi, prompt akan berubah menjadi (workspace-05).


Bagian 1  Pembuatan Aplikasi Web Blacksheep

Install Blacksheep dan CLI-nya
<img width="963" height="329" alt="image" src="https://github.com/user-attachments/assets/2f03d5c3-8faf-49cc-b61e-132b5903a4f7" />
<img width="963" height="546" alt="image" src="https://github.com/user-attachments/assets/882902cd-1b1e-4799-8f3d-0de45f6fde3e" />

Buat Proyek Baru
<img width="959" height="254" alt="image" src="https://github.com/user-attachments/assets/a183dcac-8ccc-4567-8ba6-4a544652ee85" />
Masuk ke Direktori Proyek dan Install Dependensi
<img width="963" height="377" alt="image" src="https://github.com/user-attachments/assets/94f10dc6-a2f2-4d74-ba7b-ab552bf655f2" />
Jalankan Aplikasi untuk Uji Coba
<img width="962" height="354" alt="image" src="https://github.com/user-attachments/assets/a2dd62de-a71b-4238-8a37-081d8fd0e029" />
<img width="1582" height="823" alt="image" src="https://github.com/user-attachments/assets/28691527-5638-43de-8ab6-c0d3902f7938" />

Bagian 2 — Setup Docker untuk Load Balancing
Struktur direktori yang perlu dibuat adalah:
<img width="451" height="167" alt="image" src="https://github.com/user-attachments/assets/dfdb27ed-e310-418f-894c-d816fcf1243a" />
Buat Direktori nginx
<img width="962" height="148" alt="image" src="https://github.com/user-attachments/assets/74fb718c-831a-4588-a16e-06c67ba7982d" />
Buat File nginx/Dockerfile
<img width="963" height="161" alt="image" src="https://github.com/user-attachments/assets/504fce2f-0e85-4a4f-88f3-55f98d74ad97" />
Lalu Isikan:
<img width="749" height="160" alt="image" src="https://github.com/user-attachments/assets/59bc70ab-3b01-4ff0-9920-772c28a11d3a" />
Buat file nginx\nginx.conf dengan isi:
<img width="879" height="391" alt="image" src="https://github.com/user-attachments/assets/bf7c4aa7-5d9a-43e2-8ec7-94ea685a80c8" />
Buat file docker-compose.yml di direktori workspace-05 dengan isi:
<img width="755" height="291" alt="image" src="https://github.com/user-attachments/assets/2bb4f98d-aa96-4566-ba1b-40073387ca5d" />
Pastikan blacksheep_lb/Dockerfile yang sudah ada menggunakan Python 3.14.4-slim. Jika versi berbeda, edit bagian FROM python:... sesuaikan dengan versi Python yang digunakan.

Bagian 3 — Menjalankan Docker Compose

Jalankan dengan Scale 2 Instance.

Di PowerShell atau Command Prompt (dari direktori workspace-05):
<img width="961" height="567" alt="image" src="https://github.com/user-attachments/assets/53448a43-65b3-4111-ba62-52d80572eb09" />
<img width="961" height="186" alt="image" src="https://github.com/user-attachments/assets/585756d0-0058-4039-8f34-2ee7bdf6acd2" />
Verifikasi Container Berjalan.















