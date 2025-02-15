# Custom Relationship in Laravel

## Case Overview
Relasi ini digunakan ketika field untuk hubungan antar tabel tidak menggunakan `id` (primary key), tetapi field custom seperti `kode_user`. Namun, field `id` tetap dipertahankan untuk kebutuhan auto increment di masa depan.

Misalnya, tabel `users` memiliki field `kode_user` yang unik, dan tabel `posts` menggunakan `kode_user` sebagai foreign key.

---

## Migration Setup

### Tabel `users`

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id(); 
            $table->string('kode_user')->unique(); // Custom unique field
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
}
```

### Tabel `posts`

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePostsTable extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id(); 
            $table->string('kode_user'); // Custom foreign key
            $table->foreign('kode_user')
                  ->references('kode_user')
                  ->on('users')
                  ->onDelete('cascade');
            $table->string('title');
            $table->text('content');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

---

## Model Setup

### User Model (`User.php`)

```php
public function posts()
{
    // Parameter kedua: foreign key di tabel posts, parameter ketiga: local key di tabel users
    return $this->hasMany(Post::class, 'kode_user', 'kode_user');
}
```

### Post Model (`Post.php`)

```php
public function user()
{
    // Parameter kedua: foreign key di tabel posts, parameter ketiga: owner key di tabel users
    return $this->belongsTo(User::class, 'kode_user', 'kode_user');
}
```

---

## Notes

- **Field `id` tetap digunakan** untuk kebutuhan auto increment dan standarisasi, meskipun relasi tidak mengacu padanya.
- **`kode_user`** dipastikan unik agar relasi tetap valid.
- Jika ingin menggunakan `foreignId()` seperti default Laravel, field yang dirujuk harus primary key.
