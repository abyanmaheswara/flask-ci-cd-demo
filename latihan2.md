# 🚀 Panduan CI/CD dengan Docker & GitHub Actions untuk Pemula

Bayangkan setiap kali Anda melakukan *push* kode ke GitHub, secara otomatis aplikasi Anda akan di-*build*, di-*test*, dan siap di-*deploy*. Itulah kekuatan CI/CD! Dengan menggabungkan **Docker** dan **GitHub Actions**, Anda akan mengotomatiskan seluruh proses, mengakhiri era *deploy* manual yang rentan kesalahan dan melelahkan.

---

## 📚 Bab 1: Memahami CI/CD (Tanpa Pusing)

**CI/CD** (Continuous Integration & Continuous Deployment) adalah proses otomatisasi dari kode hingga produksi, dengan cara sebagai berikut:

- **Continuous Integration (CI)**: setiap kali Anda *push* kode, sistem akan langsung membangun (*build*) dan menguji (*test*) aplikasi secara otomatis. Ini membantu menemukan *bug* sejak dini.
- **Continuous Deployment (CD)**: setelah semua tes berhasil, sistem akan secara otomatis menempatkan aplikasi Anda ke server, siap digunakan.

**Tujuan CI/CD Pipeline yang akan kita buat**:
1. Menjalankan *test* secara otomatis untuk memastikan aplikasi berfungsi.
2. Membangun Docker *image* dari aplikasi Anda.
3. Mengirim (*push*) *image* tersebut ke **Docker Hub**.
4. Siap di-*deploy* di server manapun hanya dengan menarik (*pull*) *image* terbaru.

---

## 🛠️ Bab 2: Persiapan Akun dan Tools

| Akun/Tools | Tujuan | Cara Persiapan |
|:---|:---|:---|
| **GitHub** | Menyimpan kode & menjalankan pipeline | Daftar di [github.com/signup](https://github.com/signup) |
| **Docker Hub** | Menyimpan Docker *image* hasil build | Daftar gratis di [hub.docker.com/signup](https://hub.docker.com/signup) |
| **Docker Desktop** | Menjalankan container secara lokal | [Download](https://www.docker.com/products/docker-desktop/) dan install |
| **Git** | *Push* kode ke GitHub | Install dari [git-scm.com](https://git-scm.com/) |

---

## 💻 Bab 3: Membuat Aplikasi Contoh (Flask)

Kita akan menggunakan aplikasi web sederhana dengan Python Flask agar mudah diikuti.

### 3.1 Struktur Folder Proyek

Buat folder `flask-ci-cd-demo` dengan struktur berikut:

```
flask-ci-cd-demo/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── app.py
├── requirements.txt
├── test_app.py
└── Dockerfile
```

### 3.2 `app.py` – Aplikasi Web

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        "message": "Selamat datang di CI/CD Pipeline!",
        "status": "success"
    })

@app.route('/health')
def health():
    return jsonify({"status": "healthy"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 3.3 `requirements.txt` – Dependensi

```
flask==2.3.3
pytest==7.4.0
```

### 3.4 `test_app.py` – Unit Test

```python
import pytest
from app import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_home(client):
    response = client.get('/')
    assert response.status_code == 200
    assert response.json['status'] == 'success'

def test_health(client):
    response = client.get('/health')
    assert response.status_code == 200
    assert response.json['status'] == 'healthy'
```

---

## 🐳 Bab 4: Membuat Dockerfile (Resep Image)

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Copy requirements dan install dependensi
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy seluruh kode aplikasi
COPY . .

# Expose port 5000
EXPOSE 5000

# Jalankan aplikasi
CMD ["python", "app.py"]
```

---

## 🧪 Bab 5: Uji Coba Lokal (Sebelum CI/CD)

### 5.1 *Build* Image Secara Manual

```bash
cd flask-ci-cd-demo
docker build -t flask-cicd-demo .
```

### 5.2 Jalankan Container

```bash
docker run -d -p 5000:5000 --name flask-app flask-cicd-demo
```

Buka `http://localhost:5000` di browser, seharusnya Anda melihat JSON response.

### 5.3 Hentikan Container

```bash
docker stop flask-app && docker rm flask-app
```

Jika berhasil, aplikasi Anda siap untuk diotomatisasi!

---

## 🔑 Bab 6: Menyiapkan GitHub & Docker Hub Secrets

### 6.1 Buat Personal Access Token (PAT) di Docker Hub

1. Login ke [hub.docker.com](https://hub.docker.com/), buka **Account Settings** → **Security**.
2. Klik **New Access Token**, beri nama misal `github-actions-token`.
3. Pilih akses **Read & Write**, lalu **Generate** dan **copy tokennya** (simpan di Notepad).

### 6.2 Buat Repository GitHub

1. Buat repository baru di GitHub (misal `flask-cicd-demo`).
2. Ikuti petunjuk untuk menghubungkan folder lokal Anda:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/flask-cicd-demo.git
git push -u origin main
```

### 6.3 Tambahkan Secrets di GitHub

1. Buka repository GitHub → **Settings** → **Secrets and variables** → **Actions**.
2. Klik **New repository secret**, buat dua secret berikut:
   - `DOCKER_USERNAME` → isi dengan Docker Hub username Anda.
   - `DOCKERHUB_TOKEN` → isi dengan Access Token yang tadi disalin.

---

## ⚙️ Bab 7: Membuat GitHub Actions Workflow

### 7.1 Buat File Workflow

Buat folder `.github/workflows/` dan file `ci-cd.yml` di dalamnya:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Test Aplikasi
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout kode
      uses: actions/checkout@v4
      
    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.9'
        
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        
    - name: Jalankan unit test
      run: |
        pytest test_app.py -v

  build-and-push:
    name: Build & Push ke Docker Hub
    runs-on: ubuntu-latest
    needs: test  # hanya jalan kalau test sukses
    
    steps:
    - name: Checkout kode
      uses: actions/checkout@v4
      
    - name: Login ke Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
        
    - name: Setup Docker Buildx
      uses: docker/setup-buildx-action@v3
      
    - name: Build dan Push Image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/flask-cicd-demo:latest
```

### 7.2 Simpan dan Push

```bash
git add .github/workflows/ci-cd.yml
git commit -m "Add CI/CD pipeline"
git push origin main
```

---

## ▶️ Bab 8: Pipeline Berjalan (Magic Time!)

Setelah *push*, buka repository GitHub Anda → tab **Actions**.

Anda akan melihat workflow berjalan dengan dua *job*:
1. **Test Aplikasi** → menjalankan `pytest`.
2. **Build & Push ke Docker Hub** → hanya berjalan jika *test* sukses, lalu membangun Docker *image* dan mengirimkannya ke Docker Hub Anda.

🎉 **Selamat!** Sekarang setiap kali Anda *push* kode ke branch `main`, pipeline ini akan berjalan secara otomatis.

---

## ☁️ Bab 9: Deploy ke Server (Opsional)

Setelah *image* berada di Docker Hub, Anda bisa men-*deploy* ke server mana pun dengan satu perintah:

```bash
# Di server tujuan
docker run -d -p 5000:5000 --name flask-app ${{ secrets.DOCKER_USERNAME }}/flask-cicd-demo:latest
```

Untuk otomatisasi penuh, Anda dapat menambahkan *job* ketiga di workflow yang menggunakan SSH untuk masuk ke server dan menarik (*pull*) *image* terbaru.

---

Materi panduan Anda sudah sangat solid dan sistematis! Memisahkan soal dari jawaban adalah ide yang bagus agar mahasiswa benar-benar tertantang untuk berpikir sebelum mengintip kunci jawabannya.

Berikut adalah revisi untuk **Bab 10** serta tambahan **Latihan Praktikum Mandiri** untuk meningkatkan level kompetensi mahasiswa.

---

## 🧠 Bab 10: Uji Pemahaman (Quiz)

Coba jawab pertanyaan di bawah ini secara mandiri.

### **Bagian A: Soal**
1. Apa yang akan terjadi pada pipeline jika Anda melakukan *push* kode yang memiliki kesalahan logika sehingga gagal dalam *unit test*?
2. Di folder manakah file konfigurasi GitHub Actions harus diletakkan agar dapat dikenali oleh GitHub?
3. Pada file `ci-cd.yml`, apa fungsi dari baris kode `needs: test`?
4. Kapan sebaiknya kita menggunakan `Secrets` dibandingkan dengan `Variables` di GitHub Actions?
5. Mengapa kita disarankan menggunakan `docker/setup-buildx-action@v3` daripada perintah `docker build` biasa?
6. Jika pipeline Anda berwarna merah (gagal), langkah apa yang pertama kali Anda lakukan untuk mencari penyebabnya?
7. Apa konsekuensinya jika Anda mengganti nama *Secret* di GitHub tetapi tidak mengubah referensinya di file YAML?
8. Bagaimana cara menambahkan endpoint baru (misal: `/version`) ke dalam siklus CI/CD ini?
9. Jika ingin menyimpan image dengan tag versi spesifik (misal: `v1.0`) sekaligus `latest`, bagaimana cara penulisannya di file workflow?
10. Apa kegunaan dari `actions/checkout@v4` yang selalu muncul di awal setiap *job*?

---

## 🛠️ Latihan Praktikum Mandiri: Level Up!

Setelah berhasil menjalankan pipeline dasar, cobalah tantangan berikut untuk meningkatkan *skill* Anda dari pemula ke level menengah.

### **Tantangan 1: Keamanan (Security Scan)**
Jangan hanya tes fungsi, tes juga keamanannya!
* **Tugas:** Tambahkan langkah (*step*) baru di dalam job `test` menggunakan alat bernama **Trivy** untuk mendeteksi celah keamanan (vulnerability) pada file `Dockerfile` atau library yang digunakan.
* **Petunjuk:** Cari *action* `aquasecurity/trivy-action` di GitHub Marketplace.

### **Tantangan 2: Automatisasi Versi (Semantic Tagging)**
Tag `latest` terkadang membingungkan di produksi.
* **Tugas:** Ubah pipeline agar setiap kali Anda membuat **Git Tag** (misal: `v1.1`), GitHub Actions secara otomatis mem-build image dengan tag yang sama dengan tag Git tersebut.
* **Petunjuk:** Gunakan variabel `${{ github.ref_name }}` untuk mengambil nama tag atau branch.

### **Tantangan 3: Environment yang Berbeda**
* **Tugas:** Buat agar pipeline hanya melakukan *Push* ke Docker Hub jika Anda melakukan push ke branch `main`, namun jika Anda push ke branch `develop`, pipeline hanya menjalankan `test` saja tanpa mem-build image.
* **Petunjuk:** Gunakan kondisi `if: github.ref == 'refs/heads/main'` pada level *job*.

### **Tantangan 4: Slack/Discord Notification**
* **Tugas:** Tambahkan integrasi agar setiap kali pipeline selesai (berhasil atau gagal), sistem mengirimkan notifikasi ke pesan grup Discord atau Slack Anda.
* **Petunjuk:** Gunakan *Webhooks* dan *Action* pihak ketiga untuk notifikasi.

---
**Tips Karir:** Menguasai otomatisasi seperti ini adalah nilai tambah besar bagi seorang Backend Developer atau DevOps Engineer. Teruslah bereksperimen! 🚀