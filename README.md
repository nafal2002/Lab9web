# Modul Login - Dokumentasi

Selamat datang di dokumentasi modul login dalam framework lanjutan! Modul ini bertujuan untuk mengimplementasikan fitur login yang aman dan fungsional dalam aplikasi web menggunakan framework yang telah dipelajari.

## Deskripsi

Modul login merupakan komponen penting dalam aplikasi web yang memungkinkan pengguna untuk mengautentikasi diri sebelum dapat mengakses fitur-fitur tertentu. Dalam modul ini, kita akan mempelajari implementasi login yang melibatkan aspek-aspek seperti:

- Pembuatan form login dengan input username dan password.
- Validasi data input pengguna.
- Pengamanan kata sandi dengan teknik hashing.
- Penyimpanan informasi pengguna di database.
- Mekanisme otentikasi dan otorisasi.
- Pengelolaan sesi pengguna.

# Jika ingin Menambahkan Running text  
``` <div id="runningtext">
		<marquee behavior="scroll" scrollamount="3" onmouseover="this.stop();" onmouseout="this.start();" direction="left">
			Selamat Datang di Website Saya
		</marquee>
		</div>
```

## Petunjuk Penggunaan

1. Pastikan Anda telah menginstal framework yang digunakan dan persyaratan lain yang diperlukan.
2. Clone repositori ini ke direktori lokal Anda.
3. Konfigurasi database yang akan digunakan untuk menyimpan informasi pengguna. (Contoh: MySQL, PostgreSQL, MongoDB)
4. Lakukan migrasi database sesuai dengan skema yang telah ditentukan.
5. Pastikan dependensi yang diperlukan telah diinstal.
6. Konfigurasikan pengaturan aplikasi yang relevan, seperti koneksi database dan pengaturan keamanan.
7. Jalankan aplikasi dan buka halaman login pada browser.
8. Masukkan kredensial pengguna yang telah terdaftar untuk melakukan login.
9. Jika kredensial valid, pengguna akan diarahkan ke halaman beranda atau fitur yang sesuai.
10. Jika kredensial tidak valid, pesan kesalahan akan ditampilkan dan pengguna diminta untuk mencoba lagi.

![Tabel User Login](img/table_user_login.png)

### Membuat Tabel User

```php
CREATE TABLE user (
id INT(11) auto_increment,
username VARCHAR(200) NOT NULL,
useremail VARCHAR(200),
userpassword VARCHAR(200),
PRIMARY KEY(id)
);
```

# Membuat Model User
<p>Selanjutnya adalah membuat Model untuk memproses data Login. Buat file baru pada direktori
app/Models dengan nama UserModel.php</p>

```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class UserModel extends Model
{
  protected $table = 'user';
  protected $primaryKey = 'id';
  protected $useAutoIncrement = true;
  protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
```

# Membuat Controller User
<p>Buat Controller baru dengan nama User.php pada direktori app/Controllers. Kemudian tambahkan
method index() untuk menampilkan daftar user, dan method login() untuk proses Login.</p>

```php
<?php

namespace App\Controllers;

use App\Models\UserModel;

class User extends BaseController
{
  public function index()
  {
    $title = 'Daftar User';
    $model = new UserModel();
    $users = $model->findAll();
    return view('user/index', compact('users', 'title'));
  }
  public function login()
  {
    helper(['form']);
    $email = $this->request->getPost('email');
    $password = $this->request->getPost('password');
    if (!$email) {
      return view('user/login');
    }
    $session = session();
    $model = new UserModel();
    $login = $model->where('useremail', $email)->first();
    if ($login) {
      $pass = $login['userpassword'];
      if (password_verify($password, $pass)) {
        $login_data = [
          'user_id' => $login['id'],
          'user_name' => $login['username'],
          'user_email' => $login['useremail'],
          'logged_in' => TRUE,
        ];
        $session->set($login_data);
        return redirect('admin/artikel');
      } else {
        $session->setFlashdata("flash_msg", "Password salah.");
        return redirect()->to('/user/login');
      }
    } else {
      $session->setFlashdata("flash_msg", "email tidak terdaftar.");
      return redirect()->to('/user/login');
    }
  }
}
```

# Membuat View Login
<p>Buat direktori baru dengan nama user pada direktori app/views, kemudian buat file baru dengan nama
login.php.</p>

```php
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Login</title>
  <link rel="stylesheet" href="<?= base_url('/style.css'); ?>">
</head>
<body>
  <div id="login-wrapper">
    <h1 class="h1-login">Sign In</h1>
    <?php if (session()->getFlashdata('flash_msg')) : ?>
      <div class="alert alert-danger"><?= session()->getFlashdata('flash_msg') ?></div>
    <?php endif; ?>
    <form action="" method="post">
      <div class="mb-3">
        <label for="InputForEmail" class="form-label">Email address</label>
        <input type="email" name="email" class="form-control" id="InputForEmail" value="<?= set_value('email') ?>">
      </div>
      <div class="mb-3">
        <label for="InputForPassword" class="form-label">Password</label>
        <input type="password" name="password" class="form-control" id="InputForPassword">
      </div>
      <button type="submit" class="btn btn-primary">Login</button>
    </form>
  </div>
</body>
</html>
```

# Membuat Database Seeder
- Database seeder digunakan untuk membuat data dummy. Untuk keperluan uji coba modul login, kita
perlu memasukkan data user dan password kedalam database. Untuk itu buat database seeder
untuk tabel user. Buka CLI, kemudian tulis perintah berikut:

```bash
php spark make:seeder UserSeeder
```

- Selanjutnya, buka file UserSeeder.php yang berada di lokasi direktori
/app/Database/Seeds/UserSeeder.php kemudian isi dengan kode berikut:

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
        $model->insert([
            'username' => 'admin',
            'useremail' => 'admin@email.com',
            'userpassword' => password_hash('admin123', PASSWORD_DEFAULT),
        ]);
    }
}
```

- Kemudian, buka kembali CLI dan ketik perintah berikut:

```bash
php spark db:seed UserSeeder
```

- Lalu, uji coba login nya dengan cara mengakses url http://localhost:8080/user/login seperti berikut:

![Login Form](img/login_form.png)

# Menambahkan Auth Filter
- Selanjutnya membuat filter untuk halaman Admin. Buat file baru dengan nama Auth.php pada
direktori app/Filters, Kemudian masukan kode berikut:

```php
<?php

namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
  public function before(RequestInterface $request, $arguments = null)
  {
    // jika user belum login
    if (!session()->get('logged_in')) {
      // maka redirct ke halaman login
      return redirect()->to('/user/login');
    }
  }
  public function after(RequestInterface $request, ResponseInterface $response, $arguments = null)
  {
  }
}
```

- Kemudian buka file app/Config/Filters.php tambahkan kode berikut:

```php
'auth' => App\Filters\Auth::class
```

![Config Filters](img/config_filters.png)

- Lalu buka file app/Config/Routes.php dan sesuaikan kodenya.

![Config Routes](img/config_routes.png)

- Dan uji coba akses menu admin, dengan cara buka url dengan alamat http://localhost:8080/admin/artikel ketika alamat tersebut diakses maka akan dimuculkan halaman login.

![Login Admin](img/login_form.png)

# Fungsi Logout
<p>Tambahkan method logout() pada Controller User seperti berikut:</p>

```php
  public function logout()
  {
    session()->destroy();
    return redirect()->to('user/login');
  }
```

## Kontribusi

Jika Anda ingin berkontribusi pada pengembangan modul login ini, Anda dapat mengikuti langkah-langkah berikut:

1. Fork repositori ini.
2. Buat branch baru untuk fitur atau perbaikan yang akan Anda lakukan.
3. Lakukan perubahan yang diperlukan.
4. Lakukan commit dan push ke branch Anda.
5. Ajukan pull request untuk menggabungkan perubahan Anda ke repositori utama.

## Lisensi

Tuliskan informasi lisensi yang relevan untuk modul login ini.

## Kontak

Jika Anda memiliki pertanyaan, saran, atau masalah terkait modul login ini, silakan hubungi kami melalui [email](nafalmumtaz93@gmail.com) atau buat [issue](https://github.com/nafal2002/) di repositori ini.

Terima kasih telah menggunakan modul login ini!
