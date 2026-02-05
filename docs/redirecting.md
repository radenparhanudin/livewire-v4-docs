Setelah seorang pengguna melakukan suatu tindakan — seperti mengirimkan formulir — Anda mungkin ingin mengarahkan (*redirect*) mereka ke halaman lain di aplikasi Anda.

Karena *request* Livewire bukanlah *request* browser halaman penuh yang standar, *HTTP redirect* standar tidak akan berfungsi. Sebaliknya, Anda perlu memicu pengalihan melalui JavaScript. Untungnya, Livewire menyediakan metode *helper* `$this->redirect()` yang sederhana untuk digunakan di dalam *component* Anda. Secara internal, Livewire akan menangani proses pengalihan tersebut di sisi *frontend*.

Jika Anda lebih suka, Anda juga dapat menggunakan [utilitas redirect bawaan Laravel](https://laravel.com/docs/responses#redirects) di dalam *component* Anda juga.

## Basic usage

Di bawah ini adalah contoh *component* Livewire `post.create` yang mengarahkan pengguna ke halaman lain setelah mereka mengirimkan formulir untuk membuat postingan:

```php
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        $this->redirect('/posts'); // [tl! highlight]
    }
};
?>

<form wire:submit="save">
    </form>

```

Seperti yang Anda lihat, ketika *action* `save` dipicu, sebuah pengalihan juga akan dipicu ke `/posts`. Saat Livewire menerima respons ini, ia akan mengarahkan pengguna ke URL baru di *frontend*.

---

## Redirect to Route

Jika Anda ingin mengarahkan ke sebuah halaman menggunakan nama rutenya, Anda dapat menggunakan `redirectRoute`.

Sebagai contoh, jika Anda memiliki halaman dengan rute bernama `'profile'` seperti ini:

```php
    Route::get('/user/profile', function () {
        // ...
    })->name('profile');

```

Anda dapat menggunakan `redirectRoute` untuk mengarahkan ke halaman tersebut menggunakan nama rute seperti berikut:

```php
    $this->redirectRoute('profile');

```

Jika Anda perlu meneruskan parameter ke rute tersebut, Anda dapat menggunakan argumen kedua dari metode `redirectRoute` seperti ini:

```php
    $this->redirectRoute('profile', ['id' => 1]);

```

---

## Redirect to intended

Jika Anda ingin mengarahkan pengguna kembali ke halaman sebelumnya (halaman asal), Anda dapat menggunakan `redirectIntended`. Metode ini menerima URL default opsional sebagai argumen pertamanya yang digunakan sebagai *fallback* jika halaman sebelumnya tidak dapat ditentukan:

```php
    $this->redirectIntended('/default/url');

```

---

## Redirecting to full-page components

Karena Livewire menggunakan fitur pengalihan bawaan Laravel, Anda dapat menggunakan semua metode pengalihan yang tersedia dalam aplikasi Laravel standar.

Sebagai contoh, jika Anda menggunakan *component* Livewire sebagai *full-page component* untuk sebuah rute seperti ini:

```php
Route::livewire('/posts', 'pages::show-posts');

```

Anda dapat mengarahkannya cukup dengan menggunakan jalur rute:

```php
public function save()
{
    // ...

    $this->redirect('/posts');
}

```

---

## Redirect to controller actions

Jika Anda ingin mengarahkan ke rute yang ditangani oleh *controller action*, Anda dapat menggunakan `redirectAction()`:

```php
$this->redirectAction([UserController::class, 'index']);

```

Anda dapat meneruskan parameter ke *controller action* sebagai argumen kedua:

```php
$this->redirectAction([UserController::class, 'show'], ['id' => 1]);

```

---

## Flash messages

Selain memungkinkan Anda menggunakan metode pengalihan bawaan Laravel, Livewire juga mendukung [utilitas session flash data Laravel](https://laravel.com/docs/session#flash-data).

Untuk meneruskan *flash data* bersama dengan pengalihan, Anda dapat menggunakan metode `session()->flash()` Laravel seperti berikut:

```php
<?php

use Livewire\Component;

new class extends Component {
    // ...

    public function update()
    {
        // ...

        session()->flash('status', 'Post successfully updated.');

        $this->redirect('/posts');
    }
};
?>

```

Dengan asumsi halaman tujuan pengalihan berisi potongan kode Blade berikut, pengguna akan melihat pesan "Post successfully updated." setelah memperbarui postingan:

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif

```

---

## See also

* **[Navigate](https://www.google.com/search?q=/docs/4.x/navigate)** — Gunakan navigasi SPA untuk pengalihan
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Mengalihkan setelah penyelesaian tindakan
* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Mengalihkan setelah pengiriman formulir berhasil
* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Navigasi antar *page components*
