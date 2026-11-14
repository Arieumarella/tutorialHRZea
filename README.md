# Zea Tutorial

Website statis untuk panduan Zea HR, disajikan dengan Nginx dan diakses melalui IP lokal STB.

Tidak diperlukan Cloudflare, tunnel, token, atau API eksternal.

## Arsitektur

```text
Perangkat di jaringan yang sama
  -> http://IP-STB:8088
  -> Container Nginx
  -> html/index.html
```

9Router tidak digunakan sebagai reverse proxy. 9Router tetap berjalan terpisah di port `20128`.

## Struktur

```text
tutorialHRZea/
├── .gitignore
├── docker-compose.yml
├── README.md
└── html/
    └── index.html
```

## Prasyarat

- Docker dan Docker Compose sudah terpasang di STB.
- Port `8088` tidak dipakai service lain.
- STB dan perangkat lain berada di jaringan yang sama.

## Clone di STB

```bash
cd ~
git clone https://github.com/Arieumarella/tutorialHRZea.git
cd tutorialHRZea
```

## Cari IP STB

```bash
hostname -I
```

Atau:

```bash
ip route get 1.1.1.1 | awk '{print $7; exit}'
```

Contoh hasil:

```text
192.168.1.50
```

## Jalankan

```bash
cd ~/tutorialHRZea
docker compose up -d
docker compose ps
docker compose logs -f tutorial
```

## Verifikasi

Tes dari STB:

```bash
curl -I http://127.0.0.1:8088
```

Tes dari perangkat lain di jaringan yang sama:

```bash
curl -I http://IP-STB:8088
```

Keduanya harus mengembalikan `HTTP/2 200` atau `HTTP/1.1 200`.

Buka di browser:

```text
http://IP-STB:8088
```

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
| Melihat log Nginx | `docker compose logs -f tutorial` |
| Restart layanan | `docker compose restart` |
| Mematikan layanan | `docker compose down` |
| Menyalakan layanan | `docker compose up -d` |
| Update image | `docker compose pull && docker compose up -d` |

## Firewall

Jika halaman tidak bisa dibuka dari perangkat lain, cek firewall STB:

```bash
sudo ufw status
```

Jika UFW aktif:

```bash
sudo ufw allow 8088/tcp
```

Tidak perlu port forwarding di router. Akses hanya untuk jaringan lokal.

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Halaman tidak bisa dibuka | Container mati | Cek `docker compose ps` dan `docker compose logs tutorial` |
| Hanya bisa dari STB | Firewall memblokir port | Buka port `8088` di firewall STB |
| Port bentrok | Port `8088` dipakai service lain | Ganti port kiri pada `docker-compose.yml`, misalnya `8090:80` |
| Halaman tidak berubah | File salah lokasi | Pastikan file ada di `html/index.html` |
| IP berubah setelah reboot | DHCP memberi IP baru | Atur DHCP reservation di router |
| Image gagal pull | Arsitektur tidak didukung | Cek `uname -m`; image Nginx mendukung `amd64` dan `arm64` |

## Catatan

- 9Router tetap berjalan di `http://IP-STB:20128`.
- Repo ini hanya untuk tutorial dan konfigurasi deployment.
- File utama tutorial ada di `html/index.html`.
- Tidak ada token atau kredensial yang disimpan di repo ini.
