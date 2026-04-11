# Flask CI/CD Demo Pipeline 🚀

![CI/CD Status](https://img.shields.io/github/actions/workflow/status/abyanmaheswara/flask-ci-cd-demo/ci-cd.yml?branch=main&style=for-the-badge)
![Docker Pulls](https://img.shields.io/docker/pulls/abyanmaheswara/flask-cicd-demo?style=for-the-badge)
![Python Version](https://img.shields.io/badge/python-3.9-blue?style=for-the-badge&logo=python)

Repositori ini berisi aplikasi Flask sederhana yang diintegrasikan dengan pipeline CI/CD menggunakan GitHub Actions. Project ini mencakup pengujian otomatis, pemindaian keamanan, pembuatan image Docker, hingga notifikasi Discord.

## 🌟 Fitur Utama

- **Aplikasi Flask**: API sederhana dengan endpoint health check.
- **Unit Testing**: Menggunakan `pytest` untuk memastikan kode berjalan dengan benar.
- **Security Check**: Pemindaian kerentanan file sistem menggunakan **Trivy**.
- **Automated Docker Build**: Otomatis membuat dan push image ke **Docker Hub** saat push ke branch `main` atau saat pembuatan `tags`.
- **Discord Notification**: Bot otomatis mengirim update status build ke channel Discord.

## 🛠️ Stack Teknologi

- **Backend**: Python 3.9, Flask
- **Testing**: Pytest
- **Containerization**: Docker
- **CI/CD**: GitHub Actions
- **Security**: Trivy
- **Notification**: Discord Webhook

## 🚀 Menjalankan Secara Lokal

### Prasyarat
- Python 3.9+
- Pip atau venv

### Langkah-langkah
1. Clone repositori:
   ```bash
   git clone https://github.com/abyanmaheswara/flask-ci-cd-demo.git
   cd flask-ci-cd-demo
   ```
2. Buat virtual environment dan aktifkan:
   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # Linux/Mac
   source .venv/bin/activate
   ```
3. Install dependensi:
   ```bash
   pip install -r requirements.txt
   ```
4. Jalankan aplikasi:
   ```bash
   python app.py
   ```
   Aplikasi akan berjalan di `http://localhost:5000`.

## 🐳 Docker

Untuk menjalankan aplikasi menggunakan Docker secara lokal:
```bash
docker build -t flask-cicd-demo .
docker run -p 5000:5000 flask-cicd-demo
```

## 🏗️ Alur CI/CD

Pipeline didefinisikan dalam `.github/workflows/ci-cd.yml` dengan tahapan sebagai berikut:

1. **Test Job**: 
   - Melakukan checkout kode.
   - Setup environment Python.
   - Menjalankan `pytest`.
   - Melakukan security scan dengan `Trivy`.
2. **Build & Push Job** (Hanya pada branch `main` atau `tags`):
   - Login ke Docker Hub.
   - Build image Docker.
   - Push image dengan tag `latest` dan tag versi sesuai release.
3. **Notify Job**:
   - Mengambil hasil dari tahapan sebelumnya.
   - Mengirim status sukses/gagal ke Discord melalui Webhook.

## 📝 Kesimpulan
Proyek ini mendemonstrasikan implementasi modern DevOps dengan fokus pada **Automation**, **Security**, dan **Visibility**. Dengan integrasi CI/CD, setiap perubahan kode divalidasi secara ketat sebelum dirilis ke lingkungan produksi.

---
Dibuat dengan ❤️ oleh [Abyan Maheswara](https://github.com/abyanmaheswara)
