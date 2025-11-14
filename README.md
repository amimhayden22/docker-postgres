# Postgres (Docker)

Project ini menjalankan layanan PostgreSQL menggunakan image Docker `postgres:17` dan website Adminer untuk manajemen database.

**Ringkasan:**
- **Service utama:** PostgreSQL (image `postgres:17`)
- **Admin UI:** Adminer dengan Host `8081`
- **Volume persistensi data:** `pgdata`
- **Jaringan eksternal yang digunakan:** `nginx-proxy-network` (harus sudah ada)

**Prasyarat:**
- Docker (Engine) terpasang
- Docker Compose (atau `docker compose`) terpasang
- Terdapat jaringan Docker eksternal bernama `nginx-proxy-network` (atau ubah di `docker-compose.yaml`)

**Variabel lingkungan yang harus disediakan**
- `PG_PASSWORD` — password untuk user `postgres` di container.

Sebaiknya buat file `.env` dengan menyalin dari `.env.example` yang ada di repository:

```bash
cp .env.example .env
```

**Menjalankan project**

1. (Opsional) buat jaringan eksternal jika belum ada:

```bash
docker network create nginx-proxy-network
```

2. Pastikan `.env` berisi `PG_PASSWORD`, lalu jalankan:

```bash
docker compose up -d
```

3. Cek service berjalan:

```bash
docker compose ps
```

4. Masuk ke logs (jika perlu):

```bash
docker compose logs -f db
```

**Akses Adminer (UI web)**
- Buka: http://localhost:8081
- Isi credential: 
    - server = `db`
    - user = `postgres`
    - password = nilai `PG_PASSWORD` 
    - database = (kosong = gunakan `postgres`)

**Lokasi data & persistensi**
- Data PostgreSQL disimpan pada volume Docker `pgdata` yang dipetakan ke `/var/lib/postgresql/data` di container.

Perintah melihat volume:

```bash
docker volume ls
docker volume inspect my-docker-labs_postgres_pgdata || docker volume inspect pgdata
```

**Backup & Restore (singkat)**
- Backup dari host (menggunakan `pg_dump` via container):

```bash
docker compose exec db pg_dump -U postgres -F c -b -v -f /tmp/backup.dump your_database
docker cp $(docker compose ps -q db):/tmp/backup.dump ./backup.dump
```

- Restore contoh:

```bash
docker cp ./backup.dump $(docker compose ps -q db):/tmp/backup.dump
docker compose exec db pg_restore -U postgres -d your_database /tmp/backup.dump
```

**Troubleshooting**
- Jika container Postgres terus restart: periksa `docker compose logs db` untuk pesan error (mis. permission pada volume atau variabel lingkungan tidak ada).
- Jika Adminer tidak bisa tersambung: pastikan service `db` berjalan dan nama server di Adminer diisi `db`.
- Jika jaringan eksternal `nginx-proxy-network` tidak ada, buat dengan `docker network create nginx-proxy-network` atau edit `docker-compose.yaml` untuk menggunakan network lain.

**Menghentikan dan menghapus**

```bash
docker compose down
```

Jika ingin menghapus data juga:

```bash
docker compose down -v
```

**Materi**
- [https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/#How-to-run-Postgres-in-Docker](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/#How-to-run-Postgres-in-Docker)