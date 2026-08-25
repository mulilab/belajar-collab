# Workflow Kolaborasi GitHub yang Efektif

Panduan ini menjelaskan alur kolaborasi berbasis **GitHub Flow**: `main` selalu dijaga dalam kondisi stabil, setiap perubahan dibuat pada branch terpisah, dan kode hanya digabungkan melalui Pull Request (PR).

## Prinsip utama

1. Jangan bekerja langsung di `main`.
2. Satu branch menangani satu tujuan atau issue.
3. Buat perubahan kecil agar mudah ditinjau dan diuji.
4. Sinkronkan branch dengan `main` sebelum digabungkan.
5. Semua perubahan harus melalui PR, automated checks, dan code review.
6. Jangan pernah menyimpan password, token, API key, atau secret lain di repository.

> **Aturan wajib:** jangan melakukan `git push` langsung ke `main` dan jangan mengubah isi `main` secara langsung. Perubahan hanya boleh masuk ke `main` melalui Pull Request yang sudah direview dan seluruh required checks-nya lulus.

Alur ringkas:

```text
Issue -> Branch -> Commit -> Push -> Pull Request -> Review/CI -> Squash Merge -> Hapus Branch
```

## Konvensi yang digunakan

### Nama branch

Gunakan huruf kecil, tanda hubung, dan nomor issue jika tersedia.

| Jenis | Format | Contoh |
| --- | --- | --- |
| Fitur | `feat/<issue>-<deskripsi>` | `feat/42-profil-pengguna` |
| Perbaikan | `fix/<issue>-<deskripsi>` | `fix/57-validasi-email` |
| Dokumentasi | `docs/<issue>-<deskripsi>` | `docs/61-api-login` |
| Pemeliharaan | `chore/<issue>-<deskripsi>` | `chore/70-upgrade-dependency` |
| Perbaikan mendesak | `hotfix/<issue>-<deskripsi>` | `hotfix/88-payment-timeout` |

### Pesan commit

Gunakan format [Conventional Commits](https://www.conventionalcommits.org/):

```text
<type>(<scope opsional>): <ringkasan singkat>
```

Contoh:

```text
feat(profile): tambah halaman profil pengguna
fix(auth): cegah login dengan akun nonaktif
docs: jelaskan konfigurasi development
test(payment): tambah pengujian transaksi gagal
chore(deps): perbarui dependency validasi
```

Commit harus fokus pada satu perubahan logis. Hindari pesan seperti `update`, `fix bug`, atau `changes` karena tidak menjelaskan alasan perubahan.

## Workflow harian

### 1. Pilih atau buat issue

Sebelum mulai, pastikan pekerjaan memiliki tujuan dan acceptance criteria yang jelas. Assign issue kepada orang yang mengerjakannya untuk mencegah pekerjaan ganda.

### 2. Perbarui `main` dan buat branch

```bash
git switch main
git pull --ff-only origin main
git switch -c feat/42-profil-pengguna
```

`--ff-only` mencegah Git membuat merge commit tak terduga pada `main` lokal.

### 3. Kerjakan dan periksa perubahan

```bash
git status
git diff
```

Jalankan formatter, linter, dan test yang relevan dengan proyek sebelum melakukan commit.

```bash
git add <file-yang-diubah>
git diff --staged
git commit -m "feat(profile): tambah halaman profil pengguna"
```

Pilih file secara eksplisit saat menjalankan `git add`. Hindari memasukkan file sementara, hasil build, konfigurasi lokal, atau secret.

### 4. Sinkronkan branch dengan `main`

Lakukan ini sebelum membuka PR dan ulangi jika `main` berubah selama proses review.

```bash
git fetch origin
git rebase origin/main
```

Rebase menempatkan commit branch di atas versi `main` terbaru sehingga riwayat tetap linear. Jangan melakukan rebase pada branch bersama tanpa koordinasi karena commit hash akan berubah.

### 5. Push dan buka Pull Request

Untuk push pertama:

```bash
git push -u origin feat/42-profil-pengguna
```

Isi PR minimal mencakup:

- masalah yang diselesaikan dan tautan issue;
- ringkasan pendekatan yang digunakan;
- cara menguji perubahan;
- screenshot atau rekaman untuk perubahan UI;
- risiko, migration, atau perubahan konfigurasi jika ada.

Gunakan **Draft PR** jika pekerjaan belum siap ditinjau tetapi perlu terlihat oleh tim.

### 6. Review dan automated checks

Reviewer memeriksa kebenaran perilaku, keamanan, test, keterbacaan, dan dampak pada bagian lain. Komentar harus spesifik pada kode dan menjelaskan alasan perubahan yang diminta.

Author PR harus:

1. menanggapi atau memperbaiki setiap komentar;
2. menjalankan kembali test setelah perubahan;
3. memastikan seluruh checks berhasil;
4. menandai semua percakapan selesai setelah ada kesepakatan.

### 7. Gabungkan dan bersihkan branch

Gunakan **Squash and merge** agar satu PR menjadi satu commit yang mudah dilacak di `main`. Judul squash mengikuti format Conventional Commits.

Setelah PR berhasil digabungkan:

```bash
git switch main
git pull --ff-only origin main
git branch -d feat/42-profil-pengguna
```

Hapus branch remote melalui tombol **Delete branch** di GitHub atau jalankan:

```bash
git push origin --delete feat/42-profil-pengguna
```

## Skenario: membuat halaman login PHP sampai masuk ke `main`

Contoh ini menunjukkan bagaimana satu fitur login dikerjakan secara kolaboratif. Nama issue yang digunakan adalah `#101 - Buat halaman login PHP`.

### 1. Buat branch fitur

Mulai dari `main` terbaru agar pekerjaan tidak dibuat dari kode lama:

```bash
git switch main
git pull --ff-only origin main
git switch -c feat/101-login-php
```

### 2. Implementasikan halaman

Buat `login.php` sebagai halaman form. Contoh tampilan minimal:

```php
<?php
declare(strict_types=1);

session_start();
$errors = $_SESSION['login_errors'] ?? [];
unset($_SESSION['login_errors']);
?>
<!doctype html>
<html lang="id">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Login</title>
</head>
<body>
    <main>
        <h1>Masuk</h1>

        <?php if ($errors !== []): ?>
            <ul role="alert">
                <?php foreach ($errors as $error): ?>
                    <li><?= htmlspecialchars((string) $error, ENT_QUOTES, 'UTF-8') ?></li>
                <?php endforeach; ?>
            </ul>
        <?php endif; ?>

        <form method="post" action="/auth/login">
            <label for="email">Email</label>
            <input id="email" name="email" type="email" autocomplete="email" required>

            <label for="password">Password</label>
            <input id="password" name="password" type="password" autocomplete="current-password" required>

            <!-- Token CSRF harus dibuat dan divalidasi oleh server. -->
            <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($csrfToken ?? '', ENT_QUOTES, 'UTF-8') ?>">
            <button type="submit">Masuk</button>
        </form>
    </main>
</body>
</html>
```

Halaman tersebut hanya menangani tampilan form. Endpoint `/auth/login` tetap harus melakukan validasi server-side, query dengan prepared statement, verifikasi `password_hash` menggunakan `password_verify`, rotasi session setelah login, rate limiting, dan validasi CSRF. Jangan membandingkan password plaintext atau menaruh kredensial database di file halaman.

### 3. Uji dan commit

Periksa sintaks dan lakukan pengujian manual sebelum push:

```bash
php -l login.php
git diff -- login.php
git add login.php
git commit -m "feat(auth): tambah halaman login PHP"
```

Checklist pengujian login:

- [ ] Email kosong atau formatnya salah ditolak.
- [ ] Password kosong ditolak.
- [ ] Kredensial salah menghasilkan pesan umum tanpa membocorkan akun mana yang valid.
- [ ] Kredensial benar membuat session baru dan mengarahkan pengguna ke halaman tujuan.
- [ ] Token CSRF invalid ditolak.
- [ ] Input ditampilkan kembali dengan escaping dan tidak menghasilkan XSS.
- [ ] Halaman dapat digunakan melalui keyboard dan tampak baik pada layar kecil.

### 4. Push branch dan buka PR

```bash
git fetch origin
git rebase origin/main
git push -u origin feat/101-login-php
```

Buka PR dengan judul `feat(auth): tambah halaman login PHP`, tautkan issue `#101`, sertakan screenshot, hasil `php -l`, cara pengujian, dan catatan keamanan. Reviewer memeriksa validasi, session, CSRF, escaping, rate limiting, serta automated checks sebelum memberi approval.

### 5. Ajukan merge ke `main`

Alur yang diwajibkan adalah **Squash and merge** melalui PR GitHub. Author tidak boleh mengubah `main` atau melakukan push langsung ke `main`. Setelah PR disetujui, checks lulus, dan PR di-merge oleh GitHub:

```bash
git switch main
git pull --ff-only origin main
git branch -d feat/101-login-php
```

Jangan gunakan `git push origin main`, `git push --force origin main`, atau cara lain yang melewati PR. Branch protection harus menolak push langsung dan membatasi siapa yang dapat melakukan bypass untuk keadaan darurat yang terdokumentasi.

## Skenario konflik Git

Konflik terjadi ketika Git tidak dapat menentukan cara menggabungkan dua perubahan secara otomatis. Konflik bukan sekadar masalah teknis: penyelesai konflik harus memahami perilaku akhir yang diinginkan.

### Skenario 1: dua developer mengubah baris yang sama

Misalnya Developer A dan Developer B sama-sama mengubah konfigurasi berikut:

```js
const timeout = 30
```

Developer A mengubah nilainya menjadi `60` dan PR-nya lebih dahulu masuk ke `main`. Developer B mengubah nilai yang sama menjadi `45`. Ketika Developer B menyinkronkan branch:

```bash
git fetch origin
git rebase origin/main
```

Git dapat menghentikan rebase dan menampilkan penanda konflik:

```text
# marker pembuka: <<<<<<< HEAD
const timeout = 60
# marker pemisah: =======
const timeout = 45
# marker penutup: >>>>>>> feat/payment-timeout
```

Bagian di atas `=======` berasal dari tujuan rebase (`origin/main`), sedangkan bagian di bawahnya berasal dari commit yang sedang diterapkan kembali. Jangan hanya menghapus penanda; tentukan nilai yang benar berdasarkan kebutuhan fitur. Hasil akhirnya, misalnya:

```js
const timeout = 45
```

Lanjutkan penyelesaian:

```bash
git status
git add <file-yang-konflik>
git rebase --continue
```

Jika muncul konflik lain, ulangi proses edit, test, `git add`, dan `git rebase --continue`. Setelah rebase selesai, jalankan seluruh test terkait.

Jika branch sebelumnya sudah di-push, perbarui branch pribadi dengan:

```bash
git push --force-with-lease
```

Gunakan `--force-with-lease`, bukan `--force`, agar push ditolak jika remote memiliki perubahan baru yang belum dimiliki lokal. Jangan force-push ke `main` atau branch yang sedang dipakai bersama.

Jika penyelesaian salah atau situasinya belum jelas, batalkan dan kembali ke kondisi sebelum rebase:

```bash
git rebase --abort
```

### Skenario 2: file dihapus oleh satu developer dan diubah developer lain

Git akan melaporkan konflik `modify/delete`. Tim harus menentukan apakah file memang sudah tidak diperlukan.

- Jika keputusan akhirnya menghapus file, jalankan `git rm <file>`.
- Jika file harus dipertahankan, pulihkan atau buat isi final yang benar lalu jalankan `git add <file>`.
- Setelah itu, lanjutkan dengan `git rebase --continue` dan jalankan test.

Jangan mempertahankan file hanya untuk menghilangkan konflik. Periksa apakah fungsi file sudah dipindahkan atau digantikan pada `main`.

### Skenario 3: Git berhasil menggabungkan, tetapi perilaku aplikasi konflik

Dua perubahan pada baris berbeda dapat lolos tanpa conflict marker tetapi tetap menghasilkan bug. Contohnya, satu PR mengganti nama fungsi sementara PR lain menambah pemanggilan baru dengan nama lama.

Mitigasinya:

- jalankan test setelah sinkronisasi dengan `main`;
- periksa diff keseluruhan PR, bukan hanya file yang berkonflik;
- gunakan integration test untuk alur yang melibatkan beberapa modul;
- koordinasikan perubahan kontrak API, schema, dan fungsi publik.

### Skenario 4: konflik dependency atau lockfile

Jika manifest dependency dan lockfile berubah pada dua branch:

1. selesaikan manifest dependency berdasarkan versi yang memang dibutuhkan;
2. hapus penanda konflik pada lockfile atau regenerasikan lockfile dengan package manager proyek;
3. install dependency dengan mode reproducible/locked jika tersedia;
4. jalankan build, test, dan pemeriksaan keamanan dependency;
5. commit manifest dan lockfile yang konsisten.

Hindari menyusun lockfile besar secara manual karena relasi dependency dan checksum mudah rusak.

## Potensi masalah dan mitigasi

| Masalah | Dampak | Pencegahan atau solusi |
| --- | --- | --- |
| Branch hidup terlalu lama | Konflik menumpuk dan perubahan sulit digabungkan | Buat branch pendek, sinkronkan rutin, dan pecah fitur besar menjadi beberapa PR |
| PR terlalu besar | Review lambat dan bug mudah terlewat | Batasi satu tujuan per PR dan pisahkan refactor dari perubahan perilaku |
| Pekerjaan ganda | Waktu terbuang dan solusi saling bertabrakan | Gunakan issue, assignee, label, dan komunikasikan pekerjaan sebelum coding |
| Commit tidak jelas | Riwayat sulit dicari dan rollback sulit dipahami | Gunakan Conventional Commits dan commit yang fokus |
| Branch tertinggal dari `main` | Konflik muncul terlambat atau CI gagal setelah merge | Jalankan `git fetch` dan `git rebase origin/main` sebelum merge |
| Merge tanpa review | Bug, celah keamanan, atau standar kode terlewat | Wajibkan PR dan minimal satu approval melalui branch protection |
| CI gagal atau flaky | Perubahan tidak dapat dipercaya dan merge tertunda | Perbaiki penyebab, jangan sekadar menjalankan ulang sampai hijau, dan dokumentasikan flaky test |
| Force-push menimpa pekerjaan | Commit anggota lain dapat hilang | Hindari branch bersama; gunakan `--force-with-lease` hanya setelah rebase branch pribadi |
| Konflik lockfile | Build berbeda antar mesin atau dependency rusak | Regenerasikan dengan package manager dan commit bersama manifest dependency |
| Migration database tidak kompatibel | Deployment atau rollback gagal | Buat migration backward-compatible dan jelaskan urutan deployment di PR |
| Perubahan API tanpa koordinasi | Consumer atau service lain berhenti bekerja | Dokumentasikan kontrak, beri masa transisi, dan tambahkan contract/integration test |
| Secret ter-commit | Kredensial bocor meskipun commit kemudian dihapus | Gunakan environment variable dan secret scanning; segera revoke serta rotasi secret yang bocor |
| Perubahan lokal belum disimpan | Checkout atau rebase terhambat; pekerjaan berisiko hilang | Commit pekerjaan siap atau gunakan `git stash push -u` sebelum berpindah konteks |
| Urutan merge beberapa PR saling bergantung | PR kedua gagal setelah PR pertama digabungkan | Nyatakan dependency antar-PR, tentukan urutan merge, lalu rebase dan test ulang |
| GitHub atau layanan CI tidak tersedia | Review, checks, atau deployment tertunda | Jangan melewati proteksi; lanjutkan pekerjaan lokal dan merge setelah layanan pulih |

## Rekomendasi branch protection untuk `main`

Aktifkan aturan berikut di pengaturan repository GitHub:

- wajibkan Pull Request sebelum merge;
- wajibkan minimal satu approval;
- reset approval ketika ada perubahan penting baru jika risikonya tinggi;
- wajibkan status checks seperti test, lint, build, dan security scan;
- wajibkan seluruh percakapan review diselesaikan;
- blokir force-push dan penghapusan `main`;
- gunakan linear history jika tim memakai squash merge;
- batasi bypass hanya untuk kondisi darurat yang tercatat.

Untuk hotfix, tetap gunakan branch dan PR. Percepat review serta checks, tetapi jangan menghilangkan audit trail atau perlindungan keamanan.

## Checklist Pull Request

### Author

- [ ] PR hanya menangani satu tujuan atau issue.
- [ ] Branch sudah disinkronkan dengan `origin/main`.
- [ ] Self-review terhadap diff sudah dilakukan.
- [ ] Test, lint, dan build lokal berhasil.
- [ ] Test baru ditambahkan untuk perubahan perilaku atau bug fix.
- [ ] Dokumentasi, migration, dan konfigurasi diperbarui bila diperlukan.
- [ ] Tidak ada secret, debug log, atau file sementara.
- [ ] Langkah pengujian dan risiko dijelaskan di deskripsi PR.

### Reviewer

- [ ] Perubahan sesuai acceptance criteria.
- [ ] Logika, keamanan, dan penanganan error sudah diperiksa.
- [ ] Test mencakup kasus utama dan kegagalan penting.
- [ ] Dampak kompatibilitas, database, API, dan dependency sudah diperiksa.
- [ ] Semua komentar penting sudah diselesaikan sebelum approval.

### Sebelum merge

- [ ] Semua required checks berhasil.
- [ ] Approval yang diwajibkan sudah tersedia.
- [ ] Tidak ada konflik dengan `main`.
- [ ] Judul squash commit mengikuti Conventional Commits.
- [ ] Rencana deployment atau rollback tersedia untuk perubahan berisiko.

## Referensi cepat

```bash
# Mulai pekerjaan
git switch main
git pull --ff-only origin main
git switch -c fix/57-validasi-email

# Simpan perubahan
git status
git diff
git add <file>
git diff --staged
git commit -m "fix(auth): perbaiki validasi email"

# Sinkronkan dan push
git fetch origin
git rebase origin/main
git push -u origin fix/57-validasi-email

# Jika branch yang sudah di-push selesai di-rebase
git push --force-with-lease

# Setelah PR di-merge
git switch main
git pull --ff-only origin main
git branch -d fix/57-validasi-email
```

Workflow yang baik tidak menghilangkan seluruh konflik. Tujuannya adalah membuat konflik muncul lebih awal, memperkecil dampaknya, dan memastikan setiap keputusan perubahan dapat ditinjau serta dilacak.
