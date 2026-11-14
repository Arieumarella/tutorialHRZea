# Zea Tutorial

Website statis untuk panduan Zea HR, disajikan dengan Nginx dan dipublikasikan melalui Cloudflare Tunnel.

## Arsitektur

```text
Internet
  -> Cloudflare
  -> Cloudflare Tunnel
  -> cloudflared container
  -> http://tutorial:80
  -> Nginx
  -> html/index.html
```

9Router tidak digunakan sebagai reverse proxy. 9Router tetap berjalan terpisah di port `20128`.

## Struktur

```text
tutorialHRZea/
├── .env.example
├── .gitignore
├── docker-compose.yml
├── README.md
└── html/
    └── index.html
```

## Prasyarat

- Docker dan Docker Compose sudah terpasang di STB.
- Domain sudah dikelola Cloudflare.
- Port `8088` tidak dipakai service lain.
- STB dapat membuat koneksi keluar ke internet.

## Clone di STB

```bash
cd ~
git clone https://github.com/Arieumarella/tutorialHRZea.git
cd tutorialHRZea
cp .env.example .env
nano .env
chmod 600 .env
```

Isi `.env` dengan token tunnel dari Cloudflare:

```bash
CLOUDFLARE_TUNNEL_TOKEN=token-dari-cloudflare
```

Jangan commit file `.env`. File tersebut sudah masuk `.gitignore`.

## Buat Cloudflare Tunnel

1. Buka Cloudflare Dashboard.
2. Masuk **Zero Trust**.
3. Buka **Networks -> Tunnels**.
4. Klik **Create a tunnel**.
5. Pilih **Cloudflared**.
6. Beri nama tunnel, misalnya `zea-tutorial`.
7. Copy token yang diberikan.
8. Simpan token ke `.env` di STB.
9. Buka tab **Public Hostname**.
10. Klik **Add a public hostname**.
11. Isi:
    - Subdomain: `tutorial`
    - Domain: domain Anda
    - Service Type: `HTTP`
    - URL: `tutorial:80`
12. Simpan.

Gunakan `tutorial:80`, bukan `localhost:8088`, karena Cloudflared berjalan di Docker network yang sama dengan Nginx.

## Jalankan

```bash
cd ~/tutorialHRZea
docker compose up -d
docker compose ps
docker compose logs -f cloudflared
```

Cari log berikut:

```text
Registered tunnel connection
```

## Verifikasi

Tes lokal di STB:

```bash
curl -I http://127.0.0.1:8088
```

Tes publik:

```bash
curl -I https://tutorial.domainanda.com
```

Keduanya harus mengembalikan `HTTP/2 200`.

## Update Tutorial

Dari Windows, salin file tutorial terbaru:

```powershell
Copy-Item "D:\PointOfSale\Aplikasi-mobile\tutorial.html" "D:\PointOfSale\zea-tutorial\html\index.html" -Force
Set-Location "D:\PointOfSale\zea-tutorial"
git add html/index.html
git commit -m "docs: update tutorial"
git push
```

Di STB:

```bash
cd ~/tutorialHRZea
git pull
```

Nginx langsung menyajikan file baru. Rebuild container tidak diperlukan.

## Perintah Operasional

| Kebutuhan | Perintah |
|---|---|
| Melihat status | `docker compose ps` |
| Melihat log tunnel | `docker compose logs -f cloudflared` |
| Melihat log Nginx | `docker compose logs -f tutorial` |
| Restart semua | `docker compose restart` |
| Mematikan layanan | `docker compose down` |
| Menyalakan layanan | `docker compose up -d` |
| Update image | `docker compose pull && docker compose up -d` |

## Keamanan

- Port `8088` hanya di-bind ke `127.0.0.1`, bukan ke semua interface.
- Jangan membuka dashboard 9Router `20128` ke internet tanpa Cloudflare Access.
- Token tunnel hanya disimpan di `.env` pada STB.
- Jika tutorial harus privat, aktifkan Cloudflare Access dan wajibkan email/OTP.
- Jangan commit `.env` atau token apa pun ke GitHub.

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `502 Bad Gateway` | Container tutorial mati | Cek `docker compose ps` dan `docker compose logs tutorial` |
| Cloudflared tidak connect | Token salah atau koneksi keluar diblokir | Cek `docker compose logs cloudflared` dan koneksi internet STB |
| DNS tidak ditemukan | Public hostname belum dibuat | Cek tab Public Hostname di Cloudflare |
| Port bentrok | Port `8088` dipakai service lain | Ganti port kiri pada `docker-compose.yml` |
| Halaman tidak berubah | File salah lokasi | Pastikan file ada di `html/index.html` |
| Image gagal pull | Arsitektur tidak didukung | Cek `uname -m`; image Nginx dan Cloudflared mendukung `amd64` dan `arm64` |

## Catatan

- 9Router tetap berjalan di `http://IP-STB:20128`.
- Repo ini hanya untuk tutorial dan konfigurasi deployment.
- File utama tutorial ada di `html/index.html`.
