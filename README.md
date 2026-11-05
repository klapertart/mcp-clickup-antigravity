# Tutorial Setup MCP ClickUp di Google Antigravity (Akun ClickUp Free)

Panduan langkah demi langkah untuk menghubungkan **Google Antigravity** ke **ClickUp** menggunakan **Model Context Protocol (MCP)**, dioptimalkan khusus untuk pengguna akun **ClickUp Free (Free Forever)** pada sistem operasi Windows.

---

## 1. Mengapa Menggunakan Local MCP + Personal API Token untuk Akun Free?

ClickUp menyediakan dua jalur integrasi yang memiliki perbedaan batas kuota (*rate limit*) signifikan pada akun **Free Forever**:

| Metode Integrasi | Jenis API | Kuota pada Akun ClickUp Free | Rekomendasi |
|---|---|---|---|
| **Local Stdio MCP (`@taazkareem/clickup-mcp-server`)** | ClickUp REST API v2 (Personal Token `pk_...`) | **100 request / menit** *(reset setiap menit, tanpa batas harian)* | **Sangat Direkomendasikan** |
| **Official Remote MCP (`https://mcp.clickup.com/mcp`)** | ClickUp Hosted AI MCP (OAuth 2.1) | **Hanya 100 call / 24 jam** *(kuota gabungan satu workspace)* | Tidak disarankan untuk akun Free (cepat terkena `429 RATE_LIMIT_EXCEEDED`) |

---

## 2. Prasyarat Sistem

Pastikan perangkat Windows Anda telah terinstal:
- **Node.js & npx** (cek melalui terminal PowerShell dengan perintah `node -v` dan `npx -v`).
- **Google Antigravity IDE**.

---

## 3. Langkah 1: Membuat Personal API Token di ClickUp

1. Login ke akun **ClickUp** Anda melalui browser (`https://app.clickup.com`).
2. Klik **Avatar / Profil** Anda (di pojok kiri bawah atau kanan atas) → pilih **Settings**.
3. Pada menu sidebar kiri, masuk ke menu **ClickUp API**.
4. Di bagian **Personal API Token**, klik tombol **Create token** (atau **Generate**).
5. Salin (*copy*) token yang dihasilkan (selalu diawali dengan prefiks `pk_...`, contoh: `pk_12345678_XXXXXXXXXXXXXXXXXXXXXXXXXXXX`).

> **Catatan Keamanan:** Jangan pernah membagikan atau melakukan *commit* token `pk_...` asli ke dalam repositori Git. Simpan hanya di file konfigurasi lokal mesin Anda.

---

## 4. Langkah 2: Mendapatkan `CLICKUP_TEAM_ID` (Workspace ID)

Ada dua cara mudah untuk mengetahui `CLICKUP_TEAM_ID` Anda:

### Cara A: Melihat dari URL Browser
Saat membuka halaman utama ClickUp di browser, perhatikan deretan angka pertama setelah domain `app.clickup.com/`:
```text
https://app.clickup.com/9018645826/v/l/...
                        ^^^^^^^^^^
                  Ini adalah CLICKUP_TEAM_ID Anda
```

### Cara B: Mengambil Otomatis via Terminal (Python / PowerShell)
Jika Anda sudah memegang Personal API Token (`pk_...`), jalankan perintah berikut di PowerShell untuk melihat daftar Workspace beserta `Team ID`-nya:

```powershell
python -c "import urllib.request, json; req = urllib.request.Request('https://api.clickup.com/api/v2/team', headers={'Authorization': 'pk_ISI_TOKEN_ANDA'}); print(json.loads(urllib.request.urlopen(req).read().decode()))"
```

---

## 5. Langkah 3: Konfigurasi `mcp_config.json` di Antigravity

Antigravity membaca konfigurasi MCP global dari direktori pengguna lokal. Agar kompatibel di semua mode eksekusi Antigravity, perbarui file konfigurasi berikut:
- `C:\Users\<username>\.gemini\config\mcp_config.json`
- `C:\Users\<username>\.gemini\antigravity\mcp_config.json`

Masukkan konfigurasi JSON berikut (menggunakan wrapper `cmd /c npx` agar proses Stdio berjalan stabil di Windows):

```json
{
  "mcpServers": {
    "clickup": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "-y",
        "@taazkareem/clickup-mcp-server@latest"
      ],
      "env": {
        "CLICKUP_API_KEY": "pk_ISI_DENGAN_PERSONAL_API_TOKEN_ANDA",
        "CLICKUP_TEAM_ID": "ISI_DENGAN_TEAM_ID_ANDA"
      }
    }
  }
}
```

---

## 6. Langkah 4: Aktivasi & Verifikasi di UI Antigravity

1. Buka panel chat **Antigravity**.
2. Klik ikon **Additional Options (`...`)** di pojok kanan atas panel chat → pilih **MCP Servers**.
3. Pastikan server **`clickup`** muncul di daftar, lalu klik **Refresh / Reconnect**.
   *(Pada pemanggilan pertama, `npx` akan mengunduh `@taazkareem/clickup-mcp-server@latest` secara otomatis selama beberapa detik hingga indikator berubah menjadi aktif).*
4. Uji koneksi melalui chat Antigravity dengan prompt sederhana, misalnya:
   > *"Cek daftar Space dan Folder yang ada di workspace ClickUp saya"*
   atau
   > *"Di Space CAKRA bagian Documentation ada dokumen apa saja?"*

---

## 7. Tips & Praktik Terbaik untuk Akun ClickUp Free

Ketika menggunakan agent AI untuk mengelola *task* atau membaca dokumentasi pada akun **ClickUp Free Forever**, perhatikan batasan platform berikut agar tidak mengalami *error*:

1. **Hindari Ketergantungan pada *Custom Fields*:**
   - Paket ClickUp Free membatasi penggunaan *Custom Fields* maksimal **60 kali** di seluruh workspace.
   - **Solusi:** Simpan atribut teknis (seperti ID Requirement `REQ-2026-001`, komponen `K-01`, nama microservice) menggunakan **Task Tags** (*unlimited* di paket Free), **Native Priority** (`Urgent`, `High`, `Normal`, `Low`), serta tabel Markdown di dalam **Task Description**.
2. **Efisiensi Struktur Space & List:**
   - Paket Free memiliki batas maksimal **5 Spaces** per Workspace dan **100 Lists** per Space. Gunakan pengelompokan **Folder** dan **List** di dalam satu Space utama (misal: Space `CAKRA`).
3. **Hemat Kuota Penyimpanan (Batas 100 MB):**
   - Hindari mengunggah lampiran file berukuran besar ke ClickUp. Cukup tautkan URL dokumen atau path repositori Git (`https://gitops.tritronik.com/...`) di dalam deskripsi Task atau ClickUp Doc.
4. **Akses ClickUp Docs (API v3):**
   - Selain mengelola *Task*, *List*, dan *Folder* (API v2), token `pk_...` yang sama juga dapat digunakan untuk menginspeksi daftar **ClickUp Docs** dan hierarki halaman (*pages/subpages*) melalui endpoint `/api/v3/workspaces/{team_id}/docs`.

