# Getting Started with GitHub / Memulai dengan GitHub

Panduan cepat untuk tim yang baru menggunakan GitHub.

---

## 1. Core Concepts / Konsep Dasar

| English | Indonesian |
|---------|------------|
| **Repository** — Project folder with version history | **Repositori** — Folder proyek dengan riwayat versi |
| **Branch** — Independent line of development | **Branch** — Jalur pengembangan independen |
| **Commit** — Snapshot of changes | **Commit** — Snapshot perubahan |
| **Pull Request** — Propose & review changes | **Pull Request** — Ajukan & review perubahan |
| **Issue** — Track tasks, bugs, ideas | **Issue** — Lacak tugas, bug, ide |

---

## 2. First-Time Setup / Persiapan Pertama

1. **Create account / Buat akun** → [github.com/signup](https://github.com/signup)

2. **Set up authentication / Atur autentikasi** (choose one / pilih salah satu):
   - **SSH Key** (recommended) → [GitHub SSH Guide](https://docs.github.com/authentication/connecting-to-github-with-ssh)
   - **Personal Access Token** → [GitHub PAT Guide](https://docs.github.com/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

3. **Configure Git / Konfigurasi Git**:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your@email.com"
   ```

4. **Clone a repository / Clone repositori**:
   ```bash
   git clone https://github.com/BorneoBiodiverse/<repository-name>.git
   cd <repository-name>
   ```

---

## 3. Daily Workflow / Alur Kerja Harian

1. **Get latest changes / Ambil update terbaru**:
   ```bash
   git pull
   ```

2. **Create a new branch / Buat branch baru**:
   ```bash
   git checkout -b nama-branch-anda
   ```

3. **Make changes / Lakukan perubahan** (edit code, add files, etc.)

4. **Stage changes / Siapkan perubahan**:
   ```bash
   git add .
   ```

5. **Commit changes / Simpan perubahan**:
   ```bash
   git commit -m "feat: deskripsi singkat perubahan"
   ```

6. **Push to GitHub / Kirim ke GitHub**:
   ```bash
   git push origin nama-branch-anda
   ```

7. **Open Pull Request / Buka Pull Request** di GitHub web interface

---

## 4. Essential Commands / Perintah Dasar

| Command | Description | Deskripsi |
|---------|-------------|-----------|
| `git clone <url>` | Download repository | Unduh repositori |
| `git pull` | Get latest changes | Ambil perubahan terbaru |
| `git status` | Show working tree status | Tampilkan status file |
| `git add .` | Stage all changes | Siapkan semua perubahan |
| `git commit -m "msg"` | Create commit | Buat commit |
| `git push` | Push to remote | Unggah ke remote |
| `git checkout -b <name>` | Create & switch branch | Buat & pindah branch |
| `git log --oneline` | Show commit history | Tampilkan riwayat commit |

---

## 5. Resources / Sumber Belajar

| Resource | Description / Deskripsi |
|----------|-------------------------|
| [GitHub Skills](https://skills.github.com) | Free interactive courses / Kursus interaktif gratis |
| [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf) | One-page PDF reference / Referensi PDF satu halaman |
| [Markdown Guide](https://www.markdownguide.org) | Formatting syntax / Sintaks format |
| [GitHub Docs](https://docs.github.com) | Official documentation / Dokumentasi resmi |
| [Git Tutorial](https://git-scm.com/docs/gittutorial) | Official Git tutorial / Tutorial Git resmi |

---

## Quick Tips / Tips Cepat

- **Pull before push** — Selalu `git pull` sebelum mulai kerja
- **Small commits** — Commit perubahan kecil & sering
- **Descriptive messages** — Pesan commit jelas: `feat: add search filter` bukan `fix`
- **Branch per task** — Satu branch = satu tugas/fitur
- **Ask for review** — Mintakan review di Pull Request

---

*Part of KalimantanBio: Biodiversity Knowledge Platform*