Penting untuk memastikan aplikasi Livewire Anda aman dan tidak mengekspos kerentanan aplikasi apa pun. Livewire memiliki fitur keamanan internal untuk menangani banyak kasus, namun, ada kalanya kode aplikasi Anda sendiri yang bertanggung jawab untuk menjaga keamanan komponen Anda.

## Otorisasi action parameters

Livewire **actions** sangatlah kuat, namun, setiap parameter yang diteruskan ke **actions** dapat diubah di sisi klien dan harus diperlakukan sebagai input pengguna yang tidak terpercaya (*un-trusted user input*).

Salah satu jebakan keamanan yang paling umum di Livewire adalah kegagalan dalam memvalidasi dan mengotorisasi pemanggilan **action** sebelum menyimpan perubahan ke database.

Berikut adalah contoh ketidakamanan yang diakibatkan oleh kurangnya otorisasi:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    // ...

    public function delete($id)
    {
        // TIDAK AMAN!

        $post = Post::find($id);

        $post->delete();
    }
}

```

```html
<button wire:click="delete({{ $post->id }})">Delete Post</button>

```

Alasan mengapa contoh di atas tidak aman adalah karena `wire:click="delete(...)"` dapat dimodifikasi di browser untuk meneruskan ID postingan APA PUN yang diinginkan oleh pengguna jahat.

**Action parameters** (seperti `$id` dalam kasus ini) harus diperlakukan sama seperti input apa pun yang tidak tepercaya dari browser.

Oleh karena itu, untuk menjaga keamanan aplikasi ini dan mencegah pengguna menghapus postingan milik pengguna lain, kita harus menambahkan otorisasi pada **action** `delete()`.

Pertama, mari buat [Laravel Policy](https://laravel.com/docs/authorization#creating-policies) untuk model Post dengan menjalankan perintah berikut:

```bash
php artisan make:policy PostPolicy --model=Post

```

Setelah menjalankan perintah di atas, sebuah Policy baru akan dibuat di dalam `app/Policies/PostPolicy.php`. Kita kemudian dapat memperbarui isinya dengan metode `delete` seperti ini:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Tentukan apakah postingan yang diberikan dapat dihapus oleh pengguna.
     */
    public function delete(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}

```

Sekarang, kita dapat menggunakan metode `$this->authorize()` dari komponen Livewire untuk memastikan pengguna adalah pemilik postingan sebelum menghapusnya:

```php
public function delete($id)
{
    $post = Post::find($id);

    // Jika pengguna bukan pemilik postingan,
    // AuthorizationException akan dilemparkan...
    $this->authorize('delete', $post); // [tl! highlight]

    $post->delete();
}

```

Bacaan lebih lanjut:

* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

---

## Otorisasi public properties

Serupa dengan **action parameters**, **public properties** di Livewire harus diperlakukan sebagai input pengguna yang tidak terpercaya.

Berikut adalah contoh yang sama seperti di atas tentang menghapus postingan, namun ditulis secara tidak aman dengan cara yang berbeda:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        // TIDAK AMAN!

        $post = Post::find($this->postId);

        $post->delete();
    }
}

```

```html
<button wire:click="delete">Delete Post</button>

```

Masalah dengan pendekatan ini adalah pengguna jahat dapat menyuntikkan elemen kustom ke halaman seperti:

```html
<input type="text" wire:model="postId">

```

Hal ini memungkinkan mereka untuk bebas memodifikasi `$postId` sebelum menekan "Delete Post". Karena **action** `delete` tidak mengotorisasi nilai `$postId`, pengguna sekarang dapat menghapus postingan mana pun di database.

Untuk melindungi dari risiko ini, ada dua solusi yang memungkinkan:

### Menggunakan model properties

Saat mengatur **public properties**, Livewire memperlakukan model secara berbeda dari nilai biasa seperti string dan integer. Karena itu, jika kita menyimpan seluruh model post sebagai properti pada komponen, Livewire akan memastikan ID-nya tidak pernah dirusak.

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public Post $post; // [tl! highlight]

    public function mount($postId)
    {
        $this->post = Post::find($postId);
    }

    public function delete()
    {
        $this->post->delete();
    }
}

```

Komponen ini sekarang aman karena tidak ada cara bagi pengguna jahat untuk mengubah properti `$post` ke model Eloquent yang berbeda.

### Mengunci properti (Locking)

Cara lain untuk mencegah properti diatur ke nilai yang tidak diinginkan adalah dengan menggunakan [atribut `#[Locked]](https://www.google.com/search?q=/docs/4.x/attribute-locked)`. Mengunci properti dilakukan dengan menerapkan atribut `#[Locked]`. Sekarang, jika pengguna mencoba merusak nilai ini, sebuah error akan dilemparkan.

Perlu dicatat bahwa properti dengan atribut **Locked** masih dapat diubah di sisi back-end, jadi tetap perlu berhati-hati agar input pengguna yang tidak terpercaya tidak diteruskan ke properti tersebut di dalam fungsi Livewire Anda sendiri.

```php
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Locked;

class ShowPost extends Component
{
    #[Locked] // [tl! highlight]
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $post->delete();
    }
}

```
### Mengotorisasi properti

Jika penggunaan **model property** tidak diinginkan dalam skenario Anda, Anda tentu saja dapat menggunakan cara manual dengan mengotorisasi penghapusan postingan di dalam **action** `delete`:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $this->authorize('delete', $post); // [tl! highlight]

        $post->delete();
    }
}

```

```html
<button wire:click="delete">Delete Post</button>

```

Sekarang, meskipun pengguna jahat masih dapat dengan bebas memodifikasi nilai `$postId`, saat **action** `delete` dipanggil, `$this->authorize()` akan melemparkan `AuthorizationException` jika pengguna tersebut bukan pemilik postingan tersebut.

Bacaan lebih lanjut:

* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

---

## Middleware

Ketika sebuah komponen Livewire dimuat pada halaman yang berisi [Authorization Middleware](https://laravel.com/docs/authorization#via-middleware) di tingkat rute, seperti ini:

```php
Route::livewire('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post'); // [tl! highlight]

```

Livewire akan memastikan bahwa **middleware** tersebut diterapkan kembali pada permintaan jaringan (*network requests*) Livewire berikutnya. Hal ini disebut sebagai "**Persistent Middleware**" di dalam inti Livewire.

**Persistent middleware** melindungi Anda dari skenario di mana aturan otorisasi atau izin pengguna telah berubah setelah pemuatan halaman awal.

Berikut adalah contoh lebih mendalam dari skenario tersebut:

```php
Route::livewire('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post'); // [tl! highlight]

```

```php
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Validate;

class UpdatePost extends Component
{
    public Post $post;

    #[Validate('required|min:5')]
    public $title = '';

    public $content = '';

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->post->update([
            'title' => $this->title,
            'content' => $this->content,
        ]);
    }
}

```

Seperti yang Anda lihat, middleware `can:update,post` diterapkan di tingkat rute. Ini berarti pengguna yang tidak memiliki izin untuk memperbarui postingan tidak dapat melihat halaman tersebut.

Namun, pertimbangkan skenario di mana seorang pengguna:

* Memuat halaman.
* Kehilangan izin untuk memperbarui setelah halaman dimuat.
* Mencoba memperbarui postingan setelah kehilangan izin tersebut.

Karena Livewire memiliki mekanisme internal untuk menerapkan kembali **middleware** dari titik akhir (*endpoint*) asli, Anda terlindungi dalam skenario ini.

### Konfigurasi persistent middleware

Secara default, Livewire mempertahankan **middleware** berikut di seluruh permintaan jaringan:

```php
\Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
\Laravel\Jetstream\Http\Middleware\AuthenticateSession::class,
\Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
\Illuminate\Routing\Middleware\SubstituteBindings::class,
\App\Http\Middleware\RedirectIfAuthenticated::class,
\Illuminate\Auth\Middleware\Authenticate::class,
\Illuminate\Auth\Middleware\Authorize::class,

```

Jika salah satu dari **middleware** di atas diterapkan pada pemuatan halaman awal, mereka akan dipertahankan (diterapkan kembali) ke permintaan jaringan apa pun di masa mendatang.

Namun, jika Anda menerapkan **middleware** kustom dari aplikasi Anda pada pemuatan halaman awal dan ingin ia tetap ada di antara permintaan Livewire, Anda perlu menambahkannya ke daftar ini dari sebuah [Service Provider](https://laravel.com/docs/providers#main-content) di aplikasi Anda:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Livewire;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::addPersistentMiddleware([ // [tl! highlight:2]
            App\Http\Middleware\EnsureUserHasRole::class,
        ]);
    }
}

```

Jika komponen Livewire dimuat pada halaman yang menggunakan **middleware** `EnsureUserHasRole`, ia sekarang akan dipertahankan dan diterapkan kembali ke setiap permintaan jaringan di masa mendatang ke komponen Livewire tersebut.

> [!warning] Argumen middleware tidak didukung
> Livewire saat ini tidak mendukung argumen **middleware** untuk definisi **persistent middleware**.
> ```php
> // Salah...
> Livewire::addPersistentMiddleware(AuthorizeResource::class.':admin');
> 
> ```
> 
> 

> // Benar...
> Livewire::addPersistentMiddleware(AuthorizeResource::class);
> ```
> 
> ```
> 
> 

### Menerapkan global Livewire middleware

Alternatifnya, jika Anda ingin menerapkan **middleware** tertentu ke setiap permintaan jaringan pembaruan Livewire, Anda dapat melakukannya dengan mendaftarkan rute pembaruan Livewire Anda sendiri dengan **middleware** apa pun yang Anda inginkan:

```php
Livewire::setUpdateRoute(function ($handle) {
    return Route::post('/livewire/update', $handle)
        ->middleware(App\Http\Middleware\LocalizeViewPaths::class);
});

```

Setiap permintaan AJAX/fetch Livewire yang dibuat ke server akan menggunakan titik akhir di atas dan menerapkan **middleware** `LocalizeViewPaths` sebelum menangani pembaruan komponen.

Pelajari lebih lanjut tentang [kustomisasi rute pembaruan di halaman Instalasi](https://livewire.laravel.com/docs/installation#configuring-livewires-update-endpoint).

---

## Snapshot checksums

Di antara setiap permintaan Livewire, sebuah **snapshot** diambil dari komponen Livewire dan dikirim ke browser. **Snapshot** ini digunakan untuk membangun kembali komponen selama siklus server berikutnya.

[Pelajari lebih lanjut tentang Livewire snapshots di dokumentasi Hydration.](https://livewire.laravel.com/docs/hydration#the-snapshot)

Karena permintaan fetch dapat dicegat dan dirusak di browser, Livewire menghasilkan "**checksum**" dari setiap **snapshot** yang menyertainya.

"**Checksum**" ini kemudian digunakan pada permintaan jaringan berikutnya untuk memverifikasi bahwa **snapshot** tersebut tidak berubah dengan cara apa pun.

Jika Livewire menemukan ketidakcocokan **checksum**, ia akan melemparkan `CorruptComponentPayloadException` dan permintaan tersebut akan gagal.

Ini melindungi aplikasi dari segala bentuk manipulasi jahat yang bertujuan memberikan kemampuan kepada pengguna untuk mengeksekusi atau memodifikasi kode yang tidak terkait.
