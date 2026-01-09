# Ujian Akhir Semester – Membuat Aplikasi Sederhana (katalog buku)

- **Nama**  : _Anggriani Hermawan_  
- **NIM**   : _312410175_  
- **Kelas** : _TI.24.A2_

---

## 1. Persiapan Project

### 1.1. Membuat folder project

- Membuat folder `KATALOG_BUKU` di dalam `htdocs` XAMPP.   
- Menambahkan file baru 
<img width="127" height="208" alt="image" src="https://github.com/user-attachments/assets/769dffee-fc78-41e0-b86f-738f38f87daa" />

### 1.2. config.php

Fungsi:
- Koneksi database `katalog_buku` (localhost/root)
- Auto create table buku (id, judul, pengarang, penerbit, tahun, stok)
- Insert 4 data sample `PHP/Laravel/Bootstrap/Statistics`
- Error handling dengan `PDO::ERRMODE_EXCEPTION`
- UTF8MB4 support untuk emoji Indonesia

Usage: require `config.php`; → $pdo siap pakai di semua file
```
<?php
$host = 'localhost';
$dbname = 'katalog_buku';
$username = 'root';
$password = '';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8mb4", $username, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
    
    // Auto create table
    $pdo->exec("CREATE TABLE IF NOT EXISTS buku (
        id INT PRIMARY KEY AUTO_INCREMENT,
        judul VARCHAR(255) NOT NULL,
        pengarang VARCHAR(255),
        penerbit VARCHAR(255),
        tahun_terbit YEAR,
        stok INT DEFAULT 0,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )");
} catch(PDOException $e) {
    die("❌ Database Error: " . $e->getMessage());
}
?>
```

---

## 2. Routing 

### 2.1. .htaccess - Clean URL Routing Katalog Buku

Fungsi:
- Ubah localhost/index.php?page=2 → /katalog/page/2
- /katalog/search/PHP → ?search=PHP (SEO friendly)
- /edit/5 → edit.php?id=5 | /hapus/10 → hapus.php?id=10
- Lindungi config.php → 403 Forbidden
- Custom 404 error page
 
```
RewriteEngine On
RewriteRule ^$ dashboard.php [L]
RewriteRule ^katalog/?$ index.php [L,QSA]
RewriteRule ^katalog/page/([0-9]+)/?$ index.php?page=$1 [L,QSA]
RewriteRule ^katalog/search/(.+)/page/([0-9]+)/?$ index.php?search=$1&page=$2 [L,QSA]
RewriteRule ^katalog/search/(.+)/?$ index.php?search=$1 [L,QSA]
RewriteRule ^tambah/?$ tambah.php [L,QSA]
RewriteRule ^edit/([0-9]+)/?$ edit.php?id=$1 [L,QSA]
RewriteRule ^hapus/([0-9]+)/?$ hapus.php?id=$1 [L,QSA]
RewriteRule ^dashboard/?$ dashboard.php [L,QSA]
RewriteRule ^logout/?$ logout.php [L,QSA]
RewriteRule ^config\.php$ - [F,L]
ErrorDocument 404 "<h1>404 - Halaman Tidak Ditemukan</h1>"
```

---

## 3. login.php - Halaman Login Admin Katalog Buku

Fitur:
- Form login responsive Bootstrap 5 (admin/admin123)
- Session management ($_SESSION['user_id'])
- Validasi input + error message
- Design modern gradient + icons
- Auto redirect ke index.php setelah login

Flow: login.php → POST → session → index.php
Tech: PHP native session + Bootstrap 5
```
<?php
require 'config.php';
session_start();

$error = '';

if ($_POST) {
    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';
    
    if ($username && $password) {
        // Login admin sederhana (ganti sesuai kebutuhan)
        if ($username === 'admin' && $password === 'admin123') {
            $_SESSION['user_id'] = 1;
            $_SESSION['username'] = $username;
            header('Location: index.php'); // nama file katalogmu
            exit;
        } else {
            $error = 'Username atau password salah!';
        }
    } else {
        $error = 'Isi semua field!';
    }
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login - Katalog Buku</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">
    <style>
        .login-card {
            max-width: 400px;
            margin: 100px auto;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }
        .bg-gradient {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
    </style>
</head>
<body class="bg-light">
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-6 col-lg-4">
                <div class="card login-card shadow-lg">
                    <div class="card-body p-5">
                        <div class="text-center mb-4">
                            <i class="bi bi-book display-1 text-primary"></i>
                            <h2 class="mt-3">Katalog Buku</h2>
                            <p class="text-muted">Silakan login</p>
                        </div>
                        
                        <?php if ($error): ?>
                        <div class="alert alert-danger">
                            <i class="bi bi-exclamation-triangle"></i> <?= $error ?>
                        </div>
                        <?php endif; ?>
                        
                        <form method="POST">
                            <div class="mb-4">
                                <label class="form-label fw-bold">
                                    <i class="bi bi-person"></i> Username
                                </label>
                                <input type="text" name="username" class="form-control" 
                                       value="<?= htmlspecialchars($_POST['username'] ?? '') ?>" required>
                            </div>
                            
                            <div class="mb-4">
                                <label class="form-label fw-bold">
                                    <i class="bi bi-lock"></i> Password
                                </label>
                                <input type="password" name="password" class="form-control" required>
                            </div>
                            
                            <button type="submit" class="btn btn-primary w-100 py-2 fw-bold">
                                <i class="bi bi-box-arrow-in-right"></i> Masuk
                            </button>
                        </form>
                        
                        <div class="text-center mt-4">
                            <small class="text-muted">
                                Demo: <strong>admin</strong> / <strong>admin123</strong>
                            </small>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2026-01-09 125855" src="https://github.com/user-attachments/assets/3407ac74-18f3-4061-9442-a070874ed445" />

---

## 4. dashboard.php - Halaman Dashboard Admin Katalog Buku

Fitur:
- Quick Actions: Katalog / Tambah Buku / Laporan / Logout
- Login required (session user_id)
- Bootstrap 5 cards + icons responsive
- Modal tambah buku (sama seperti index.php)

Tech: PHP PDO queries + Bootstrap 5
Usage: Akses setelah login → overview lengkap 1 glance!
```
<?php 
require 'config.php'; 
session_start();

// Cek login
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}

// Statistik dashboard
$stmt = $pdo->query("SELECT COUNT(*) FROM buku");
$total_buku = $stmt->fetchColumn();

$stmt = $pdo->query("SELECT COUNT(*) FROM buku WHERE stok > 0");
$stok_tersedia = $stmt->fetchColumn();

$stmt = $pdo->query("SELECT COUNT(*) FROM buku WHERE stok = 0");
$habis = $stmt->fetchColumn();
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard - Katalog Buku</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">
</head>
<body class="bg-light">
    <div class="container mt-5">
        <div class="row">
            <div class="col-md-12 text-center mb-5">
                <h1 class="display-4 text-primary"><i class="bi bi-house-door"></i> Dashboard</h1>
                <p class="lead">Selamat datang di Sistem Katalog Buku</p>
            </div>
        </div>

        <!-- Quick Actions -->
        <div class="row">
            <div class="col-md-12">
                <div class="card shadow">
                    <div class="card-header bg-primary text-white">
                        <h4><i class="bi bi-speedometer2"></i> Quick Actions</h4>
                    </div>
                    <div class="card-body">
                        <div class="row">
                            <div class="col-md-3 mb-3">
                                <a href="index.php" class="btn btn-primary btn-lg w-100 h-100">
                                    <i class="bi bi-book fs-1 d-block mb-2"></i>
                                    <strong>Katalog Buku</strong>
                                </a>
                            </div>
                            <div class="col-md-3 mb-3">
                                <a href="#tambahModal" class="btn btn-success btn-lg w-100 h-100" data-bs-toggle="modal">
                                    <i class="bi bi-plus-circle fs-1 d-block mb-2"></i>
                                    <strong>Tambah Buku</strong>
                                </a>
                            </div>
                            <div class="col-md-3 mb-3">
                                <a href="laporan.php" class="btn btn-info btn-lg w-100 h-100">
                                    <i class="bi bi-file-earmark-text fs-1 d-block mb-2"></i>
                                    <strong>Laporan</strong>
                                </a>
                            </div>
                            <div class="col-md-3 mb-3">
                                <a href="logout.php" class="btn btn-danger btn-lg w-100 h-100" onclick="return confirm('Yakin logout?')">
                                    <i class="bi bi-box-arrow-right fs-1 d-block mb-2"></i>
                                    <strong>Logout</strong>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal Tambah Buku (sama seperti index.php) -->
    <div class="modal fade" id="tambahModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-success text-white">
                    <h5 class="modal-title"><i class="bi bi-plus-circle"></i> Tambah Buku Baru</h5>
                    <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
                </div>
                <form method="POST" action="tambah.php">
                    <div class="modal-body">
                        <input type="hidden" name="action" value="tambah">
                        <div class="mb-3">
                            <label class="form-label">Judul Buku</label>
                            <input type="text" name="judul" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Pengarang</label>
                            <input type="text" name="pengarang" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Penerbit</label>
                            <input type="text" name="penerbit" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Tahun Terbit</label>
                            <input type="number" name="tahun_terbit" class="form-control" min="1900" max="2026" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Stok</label>
                            <input type="number" name="stok" class="form-control" min="0" value="0" required>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                        <button type="submit" class="btn btn-success">Simpan Buku</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d3247eb7-845c-4bc4-a804-555caa9cdb05" />

---

## 5. index.php - Halaman Utama Katalog Buku

Fitur:
- CRUD Buku (Modal Bootstrap 5)
- Search judul/pengarang (real-time)
- Pagination (5 buku/halaman)
- Nomor urut virtual (#1,#2,... page 1-2)
- Edit inline (data auto-fill modal)
- Hapus konfirmasi
- Responsive + badge stok (🟢/🔴)

Tech: PHP PDO + Bootstrap 5 + JS Vanilla
Usage: `require config.php → $pdo` → semua fitur work!
```
<?php 
require 'config.php'; 
session_start();

// Cek login
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Katalog Buku Sederhana</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">
</head>
<body class="bg-light">
    <div class="container mt-4">
        <!-- Header dengan Logout -->
        <div class="row mb-4">
            <div class="col-md-8">
                <div class="card shadow">
                    <div class="card-header bg-primary text-white">
                        <h2 class="mb-0"><i class="bi bi-book"></i> Katalog Buku</h2>
                    </div>
                </div>
            </div>
            <div class="col-md-4 text-end">
                <div class="btn-group" role="group">
                    <a href="dashboard.php" class="btn btn-outline-primary">
                        <i class="bi bi-house"></i> Dashboard
                    </a>
                    <a href="logout.php" class="btn btn-danger" onclick="return confirm('Yakin logout?')">
                        <i class="bi bi-box-arrow-right"></i> Logout
                    </a>
                </div>
            </div>
        </div>

        <div class="card shadow">
            <div class="card-body">
                <!-- Search & Pagination -->
                <?php
                $search = $_GET['search'] ?? '';
                $page = $_GET['page'] ?? 1;
                $limit = 5;
                $offset = ($page - 1) * $limit;

                $where = $search ? "WHERE judul LIKE :search OR pengarang LIKE :search" : "";
                $count_sql = "SELECT COUNT(*) FROM buku $where";
                $stmt = $pdo->prepare($count_sql);
                if ($search) {
                    $stmt->bindValue(':search', "%$search%");
                }
                $stmt->execute();
                $total = $stmt->fetchColumn();
                $pages = ceil($total / $limit);
                ?>

                <!-- Search & Tambah -->
                <div class="row mb-3">
                    <div class="col-md-8">
                        <form method="GET" class="d-flex">
                            <input type="search" name="search" class="form-control me-2" value="<?= htmlspecialchars($search) ?>" placeholder="Cari judul atau pengarang...">
                            <button class="btn btn-outline-primary"><i class="bi bi-search"></i></button>
                        </form>
                    </div>
                    <div class="col-md-4 text-end">
                        <a href="#tambahModal" class="btn btn-success" data-bs-toggle="modal">
                            <i class="bi bi-plus-circle"></i> Tambah Buku
                        </a>
                    </div>
                </div>

<!-- Tabel Buku -->
<div class="table-responsive">
    <table class="table table-hover">
        <thead class="table-dark">
            <tr>
                <th>No</th> <!-- Ubah dari ID ke No -->
                <th>Judul</th>
                <th>Pengarang</th>
                <th>Penerbit</th>
                <th>Tahun</th>
                <th>Stok</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            <?php
            $no = $offset + 1;  // Nomor urut mulai dari offset + 1
            $sql = "SELECT * FROM buku $where ORDER BY id ASC LIMIT :limit OFFSET :offset";
            $stmt = $pdo->prepare($sql);
            if ($search) {
                $stmt->bindValue(':search', "%$search%");
            }
            $stmt->bindValue(':limit', $limit, PDO::PARAM_INT);
            $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
            $stmt->execute();
            
            while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
                echo "<tr>
                        <td><strong>{$no}</strong></td>
                        <td>" . htmlspecialchars($row['judul']) . "</td>
                        <td>" . htmlspecialchars($row['pengarang']) . "</td>
                        <td>" . htmlspecialchars($row['penerbit']) . "</td>
                        <td>{$row['tahun_terbit']}</td>
                        <td><span class='badge bg-" . ($row['stok'] > 0 ? 'success' : 'danger') . "'>{$row['stok']}</span></td>
                        <td>
                            <button class='btn btn-sm btn-warning edit-btn' 
                                    data-id='{$row['id']}' 
                                    data-judul='" . htmlspecialchars($row['judul']) . "' 
                                    data-pengarang='" . htmlspecialchars($row['pengarang']) . "' 
                                    data-penerbit='" . htmlspecialchars($row['penerbit']) . "' 
                                    data-tahun='{$row['tahun_terbit']}' 
                                    data-stok='{$row['stok']}'>
                                <i class='bi bi-pencil'></i>
                            </button>
                            <a href='hapus.php?id={$row['id']}' class='btn btn-sm btn-danger' onclick='return confirm(\"Yakin hapus buku ini?\")'>
                                <i class='bi bi-trash'></i>
                            </a>
                        </td>
                    </tr>";
                $no++;  // Increment nomor urut
            }
            ?>
        </tbody>
    </table>
</div>


                <!-- Pagination -->
                <?php if ($pages > 1): ?>
                <nav>
                    <ul class="pagination justify-content-center">
                        <?php for ($i = 1; $i <= $pages; $i++): ?>
                            <li class="page-item <?= $i == $page ? 'active' : '' ?>">
                                <a class="page-link" href="?page=<?= $i ?><?= $search ? '&search=' . urlencode($search) : '' ?>"><?= $i ?></a>
                            </li>
                        <?php endfor; ?>
                    </ul>
                </nav>
                <?php endif; ?>
            </div>
        </div>
    </div>

    <!-- Modal Tambah -->
    <div class="modal fade" id="tambahModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-success text-white">
                    <h5 class="modal-title"><i class="bi bi-plus-circle"></i> Tambah Buku Baru</h5>
                    <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
                </div>
                <form method="POST" action="tambah.php">
                    <div class="modal-body">
                        <input type="hidden" name="action" value="tambah">
                        <div class="mb-3">
                            <label class="form-label">Judul Buku</label>
                            <input type="text" name="judul" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Pengarang</label>
                            <input type="text" name="pengarang" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Penerbit</label>
                            <input type="text" name="penerbit" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Tahun Terbit</label>
                            <input type="number" name="tahun_terbit" class="form-control" min="1900" max="2026" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Stok</label>
                            <input type="number" name="stok" class="form-control" min="0" value="0" required>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                        <button type="submit" class="btn btn-success">Simpan Buku</button>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal Edit -->
    <div class="modal fade" id="editModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header bg-warning text-dark">
                    <h5 class="modal-title"><i class="bi bi-pencil"></i> Edit Buku</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <form method="POST" action="edit.php">
                    <div class="modal-body">
                        <input type="hidden" name="action" value="edit">
                        <input type="hidden" name="id" id="edit_id">
                        <div class="mb-3">
                            <label class="form-label">Judul Buku</label>
                            <input type="text" name="judul" id="edit_judul" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Pengarang</label>
                            <input type="text" name="pengarang" id="edit_pengarang" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Penerbit</label>
                            <input type="text" name="penerbit" id="edit_penerbit" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Tahun Terbit</label>
                            <input type="number" name="tahun_terbit" id="edit_tahun" class="form-control" min="1900" max="2026" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Stok</label>
                            <input type="number" name="stok" id="edit_stok" class="form-control" min="0" required>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                        <button type="submit" class="btn btn-warning">Update Buku</button>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        // Edit modal functionality
        document.addEventListener('DOMContentLoaded', function() {
            const editButtons = document.querySelectorAll('.edit-btn');
            editButtons.forEach(btn => {
                btn.addEventListener('click', function() {
                    document.getElementById('edit_id').value = this.dataset.id;
                    document.getElementById('edit_judul').value = this.dataset.judul;
                    document.getElementById('edit_pengarang').value = this.dataset.pengarang;
                    document.getElementById('edit_penerbit').value = this.dataset.penerbit;
                    document.getElementById('edit_tahun').value = this.dataset.tahun;
                    document.getElementById('edit_stok').value = this.dataset.stok;
                    new bootstrap.Modal(document.getElementById('editModal')).show();
                });
            });
        });
    </script>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2026-01-09 124054" src="https://github.com/user-attachments/assets/b81ea878-2a1d-474e-b271-77f8d6af7192" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6fb4ffc8-6def-44c2-ae1f-6339bd139e9e" />

---

## 6. tambah.php - Proses Tambah Buku Baru

Fungsi:
- Terima POST dari modal tambah (index.php/dashboard.php)
- Insert buku via PDO prepared statement (secure SQL)
- Redirect ke index.php setelah sukses

Flow: Modal → POST tambah.php → DB → index.php
Tech: PDO positional params ? (anti SQL injection)
```
<?php
require 'config.php';
if($_POST['action'] == 'tambah') {
    $stmt = $pdo->prepare("INSERT INTO buku (judul, pengarang, penerbit, tahun_terbit, stok) VALUES (?, ?, ?, ?, ?)");
    $stmt->execute([$_POST['judul'], $_POST['pengarang'], $_POST['penerbit'], $_POST['tahun_terbit'], $_POST['stok']]);
}
header('Location: index.php');
?>
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3dfeeb4d-1567-4b2f-9f37-77e05995a45b" />

---

## 7. edit.php - Proses Update Data Buku

Fungsi:
- Terima POST dari modal edit (index.php)
- UPDATE buku via PDO prepared statement (secure)
- WHERE id=? → hanya 1 buku yang diupdate
- Redirect ke index.php setelah sukses

Flow: Edit modal → POST edit.php → DB → index.php
Tech: PDO positional params (anti SQL injection)

```
<?php
require 'config.php';
if($_POST['action'] == 'edit') {
    $stmt = $pdo->prepare("UPDATE buku SET judul=?, pengarang=?, penerbit=?, tahun_terbit=?, stok=? WHERE id=?");
    $stmt->execute([
        $_POST['judul'], $_POST['pengarang'], $_POST['penerbit'],
        $_POST['tahun_terbit'], $_POST['stok'], $_POST['id']
    ]);
}
header('Location: index.php');
?>
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d26b3b94-d9f8-43d4-b45d-8067d391ed6e" />

---

## 8. hapus.php - Proses Hapus Buku

Fungsi:
- Terima GET id dari tombol hapus index.php
- DELETE buku via PDO prepared statement (WHERE id=?)
- Redirect ke index.php setelah hapus

Flow: Tombol Hapus → GET hapus.php?id=5 → DB → index.php
Tech: PDO positional param (secure anti SQL injection)
```
<?php
require 'config.php';
$id = $_GET['id'] ?? 0;
if($id) {
    $stmt = $pdo->prepare("DELETE FROM buku WHERE id = ?");
    $stmt->execute([$id]);
}
header('Location: index.php');
?>
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fb63c993-797a-4580-a2c3-72fe660f7658" />
/>

---

## 9. laporan.php - Laporan Lengkap + Export Katalog Buku

Fitur:
- Statistik 4 cards: Total Buku/Tersedia/Habis/Total Stok
- Tabel semua buku (ORDER BY id DESC)
- Export CSV (export_csv.php) + Print PDF (@media print)
- Print CSS: hilangkan tombol, tabel compact
- Login required + navigasi ke dashboard/index

Tech: PHP PDO + Bootstrap 5 + CSS Print Media
Usage: /laporan → overview + download data lengkap!
```
<?php 
require 'config.php'; 
session_start();

// Cek login
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Laporan Buku - Katalog Buku</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">
    <style>
@media print {
    .no-print { display: none !important; }
    .table { font-size: 12px; }
    body { background: white !important; }
}
</style>

</head>
<body class="bg-light">
    <div class="container mt-4">
        <!-- Header -->
        <div class="row mb-4">
            <div class="col-12">
                <div class="card shadow">
                    <div class="card-header bg-info text-white">
                        <h2 class="mb-0"><i class="bi bi-file-earmark-text"></i> Laporan Buku</h2>
                    </div>
                    <div class="card-body text-center">
                        <a href="dashboard.php" class="btn btn-outline-primary mb-3">
                            <i class="bi bi-arrow-left"></i> Kembali ke Dashboard
                        </a>
                        <a href="index.php" class="btn btn-outline-secondary">
                            <i class="bi bi-book"></i> Katalog Buku
                        </a>
                    </div>
                </div>
            </div>
        </div>

        <!-- Statistik Utama -->
        <div class="row mb-4">
            <?php
            $stats = [
                $pdo->query("SELECT COUNT(*) FROM buku")->fetchColumn(),
                $pdo->query("SELECT COUNT(*) FROM buku WHERE stok > 0")->fetchColumn(),
                $pdo->query("SELECT COUNT(*) FROM buku WHERE stok = 0")->fetchColumn(),
                $pdo->query("SELECT SUM(stok) FROM buku")->fetchColumn() ?? 0
            ];
            ?>
            <div class="col-md-3">
                <div class="card shadow h-100">
                    <div class="card-body text-center">
                        <i class="bi bi-book-half display-4 text-primary"></i>
                        <h3><?= $stats[0] ?></h3>
                        <p class="text-muted">Total Buku</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="card shadow h-100">
                    <div class="card-body text-center">
                        <i class="bi bi-check-lg display-4 text-success"></i>
                        <h3><?= $stats[1] ?></h3>
                        <p class="text-muted">Tersedia</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="card shadow h-100">
                    <div class="card-body text-center">
                        <i class="bi bi-x-lg display-4 text-danger"></i>
                        <h3><?= $stats[2] ?></h3>
                        <p class="text-muted">Habis</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3">
                <div class="card shadow h-100">
                    <div class="card-body text-center">
                        <i class="bi bi-cart display-4 text-warning"></i>
                        <h3><?= $stats[3] ?></h3>
                        <p class="text-muted">Total Stok</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Tabel Buku Lengkap -->
        <div class="card shadow">
            <div class="card-header bg-secondary text-white">
                <h5>Daftar Semua Buku</h5>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-striped">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Judul</th>
                                <th>Pengarang</th>
                                <th>Penerbit</th>
                                <th>Tahun</th>
                                <th>Stok</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php
                            $stmt = $pdo->query("SELECT * FROM buku ORDER BY id DESC");
                            while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
                                $badge = $row['stok'] > 0 ? 'bg-success' : 'bg-danger';
                                echo "<tr>
                                        <td>{$row['id']}</td>
                                        <td><strong>" . htmlspecialchars($row['judul']) . "</strong></td>
                                        <td>" . htmlspecialchars($row['pengarang']) . "</td>
                                        <td>" . htmlspecialchars($row['penerbit']) . "</td>
                                        <td>{$row['tahun_terbit']}</td>
                                        <td><span class='badge $badge fs-6'>{$row['stok']}</span></td>
                                    </tr>";
                            }
                            ?>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

<!-- Export - REPLACE BAGIAN INI -->
<div class="card mt-4 shadow">
    <div class="card-header bg-success text-white">
        <h5><i class="bi bi-download"></i> Export Data Buku</h5>
    </div>
    <div class="card-body text-center">
        <!-- Export CSV -->
        <form method="POST" action="export_csv.php" class="d-inline">
            <button type="submit" name="export_csv" class="btn btn-success me-3 mb-2">
                <i class="bi bi-filetype-csv"></i> Download CSV
            </button>
        </form>
        
        <!-- Export PDF (via print styling) -->
        <button onclick="window.print()" class="btn btn-danger mb-2">
            <i class="bi bi-printer"></i> Print PDF
        </button>
        
        <p class="text-muted mt-3 small">
            CSV: Excel/Sheets | PDF: Print → Save as PDF
        </p>
        <a href="index.php" class="btn btn-outline-primary mt-2">Kembali ke Katalog</a>
    </div>
</div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
<img width="1920" height="1080" alt="Cuplikan layar 2026-01-09 130435" src="https://github.com/user-attachments/assets/523fc1e3-d108-4c22-bbc7-68cfc09d4449" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e150859e-0ef9-41e5-9a7f-8924b6ddf40c" />
cetak pdf
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d699236f-99b1-4d66-9b1a-1bb351f88948" />

---

## 10. export_csv.php - Export Data Buku ke CSV

Fungsi:
- Download katalog_buku_YYYY-MM-DD.csv (nama otomatis tanggal)
- PDO query semua buku → fputcsv streaming
- UTF-8 BOM (\xEF\xBB\xBF) → Excel buka normal (Indonesia OK)
- Header: ID,Judul,Pengarang,Penerbit,Tahun,Stok
- Login required + direct download (no tampilan HTML)

Flow: Tombol CSV → POST → stream CSV → Excel/Sheets
Tech: php://output + fputcsv + BOM UTF-8
```
<?php
require 'config.php';
session_start();
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}

if (isset($_POST['export_csv'])) {
    header('Content-Type: text/csv; charset=utf-8');
    header('Content-Disposition: attachment; filename=katalog_buku_' . date('Y-m-d') . '.csv');
    
    $output = fopen('php://output', 'w');
    fputs($output, "\xEF\xBB\xBF"); // BOM untuk UTF-8 Excel
    
    // Header CSV
    fputcsv($output, ['ID', 'Judul', 'Pengarang', 'Penerbit', 'Tahun', 'Stok']);
    
    // Data
    $stmt = $pdo->query("SELECT * FROM buku ORDER BY id");
    while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
        fputcsv($output, [
            $row['id'],
            $row['judul'],
            $row['pengarang'],
            $row['penerbit'],
            $row['tahun_terbit'],
            $row['stok']
        ]);
    }
    fclose($output);
    exit;
}
?>
```
<img width="509" height="161" alt="image" src="https://github.com/user-attachments/assets/455f5084-7d3f-45da-ac4e-698892cfc256" />

---

## 11. logout.php - Proses Logout Admin

Fungsi:
- session_destroy() → hapus semua session data
- Auto redirect ke login.php
- No HTML → pure backend script

Flow: Tombol Logout → logout.php → session clear → login.php
Tech: PHP native session_destroy()
```
<?php
session_start();
session_destroy();
header('Location: login.php');
exit;
?>
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bf0d323a-ba4c-4385-92b7-3c4f5c380a91" />

---

## 12. Cek Data di Database (phpMyAdmin)

### 12.1 Kolom tabel users
- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (`katalog_buku` ).  
- Membuka tabel `users` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `username`, `password` dan `nama` akan muncul.
<img width="591" height="179" alt="image" src="https://github.com/user-attachments/assets/06aca520-e27f-43a9-8a38-df7fa91ffc79" />

### 12.2 Kolom tabel buku
- Membuka `http://localhost/phpmyadmin`.  
- Memilih database yang digunakan (`katalog_buku` ).  
- Membuka tabel `buku` dan klik tab **Jelajahi**.  
- Setelah menyimpan data dari aplikasi, baris baru dengan kolom `id`, `judul`, `pengarang` `penerbit` `tahun terbit` `isbn` `stok` dan `created_at` akan muncul.
<img width="814" height="184" alt="image" src="https://github.com/user-attachments/assets/445d5474-1f0f-4fa0-b295-cd69d8f0256e" />

