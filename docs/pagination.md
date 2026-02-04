Fitur pagination Laravel memungkinkan Anda untuk mengambil sebagian data dari database dan memberikan kemampuan bagi pengguna untuk menavigasi di antara *pages* (halaman) dari hasil tersebut.

Karena paginator Laravel awalnya dirancang untuk aplikasi statis, pada aplikasi non-Livewire, setiap navigasi halaman memicu kunjungan browser penuh ke URL baru yang berisi halaman yang diinginkan (`?page=2`).

Namun, ketika Anda menggunakan pagination di dalam komponen Livewire, pengguna dapat berpindah antar halaman sambil tetap berada di halaman yang sama. Livewire akan menangani semuanya di balik layar, termasuk memperbarui URL query string dengan halaman saat ini.

## Penggunaan dasar

Berikut adalah contoh paling dasar penggunaan pagination di dalam komponen `show-posts` untuk hanya menampilkan sepuluh postingan sekaligus:

> [!warning] Anda harus menggunakan trait `WithPagination`
> Untuk memanfaatkan fitur pagination Livewire, setiap komponen yang berisi pagination harus menggunakan trait `Livewire\WithPagination`.

```php
<?php // resources/views/components/⚡show-posts.blade.php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function posts()
    {
        return Post::paginate(10);
    }
};

```

```blade
<div>
    <div>
        @foreach ($this->posts as $post)
            @endforeach
    </div>

    {{ $this->posts->links() }}
</div>

```

Seperti yang Anda lihat, selain membatasi jumlah postingan yang ditampilkan melalui metode `Post::paginate()`, kita juga menggunakan `$this->posts->links()` untuk merender tautan navigasi halaman.

Untuk informasi lebih lanjut tentang pagination menggunakan Laravel, silakan lihat [dokumentasi pagination Laravel yang lengkap](https://laravel.com/docs/pagination).

## Menonaktifkan pelacakan URL query string

Secara default, paginator Livewire melacak halaman saat ini di dalam URL query string browser seperti berikut: `?page=2`.

Jika Anda ingin tetap menggunakan utilitas pagination Livewire tetapi menonaktifkan pelacakan query string, Anda dapat melakukannya menggunakan trait `WithoutUrlPagination`:

```php
use Livewire\WithoutUrlPagination;
use Livewire\WithPagination;
use Livewire\Component;

class ShowPosts extends Component
{
    use WithPagination, WithoutUrlPagination; // [tl! highlight]

    // ...
}

```

Sekarang, pagination akan berfungsi seperti yang diharapkan, tetapi halaman saat ini tidak akan muncul di query string. Ini juga berarti halaman saat ini tidak akan tersimpan jika terjadi perpindahan halaman browser secara penuh.

## Menyesuaikan perilaku gulir (scroll behavior)

Secara default, paginator Livewire akan menggulir layar ke atas halaman setelah setiap perubahan halaman.

Anda dapat menonaktifkan perilaku ini dengan meneruskan `false` ke parameter `scrollTo` pada metode `links()` seperti berikut:

```blade
{{ $posts->links(data: ['scrollTo' => false]) }}

```

Alternatifnya, Anda dapat memberikan pemilih CSS (CSS selector) apa pun ke parameter `scrollTo`, dan Livewire akan mencari elemen terdekat yang cocok dengan pemilih tersebut dan menggulir layar ke arahnya setelah setiap navigasi:

```blade
{{ $posts->links(data: ['scrollTo' => '#paginated-posts']) }}

```

## Mereset halaman

Saat melakukan pengurutan (*sorting*) atau penyaringan (*filtering*) hasil, sangat umum bagi kita untuk ingin mereset nomor halaman kembali ke `1`.

Untuk alasan ini, Livewire menyediakan metode `$this->resetPage()`, yang memungkinkan Anda mereset nomor halaman dari mana saja di dalam komponen Anda.

Komponen berikut menunjukkan penggunaan metode ini untuk mereset halaman setelah formulir pencarian dikirimkan:

```php
<?php // resources/views/components/⚡search-posts.blade.php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    public $query = '';

    public function search()
    {
        $this->resetPage();
    }

    #[Computed]
    public function posts()
    {
        return Post::where('title', 'like', '%'.$this->query.'%')->paginate(10);
    }
};

```

```blade
<div>
    <form wire:submit="search">
        <input type="text" wire:model="query">

        <button type="submit">Search posts</button>
    </form>

    <div>
        @foreach ($this->posts as $post)
            @endforeach
    </div>

    {{ $this->posts->links() }}
</div>

```

Sekarang, jika pengguna sedang berada di halaman `5` dan kemudian menyaring hasil lebih lanjut dengan menekan tombol "Search posts", halaman akan direset kembali ke `1`.

### Metode navigasi halaman yang tersedia

Selain `$this->resetPage()`, Livewire menyediakan metode berguna lainnya untuk menavigasi antar halaman secara terprogram dari komponen Anda:

| Metode | Deskripsi |
| --- | --- |
| `$this->setPage($page)` | Mengatur paginator ke nomor halaman tertentu |
| `$this->resetPage()` | Mereset halaman kembali ke 1 |
| `$this->nextPage()` | Pergi ke halaman berikutnya |
| `$this->previousPage()` | Pergi ke halaman sebelumnya |

## Paginator ganda (multiple paginators)

Karena Laravel dan Livewire menggunakan parameter URL query string untuk menyimpan dan melacak nomor halaman saat ini, jika satu halaman berisi beberapa paginator, penting untuk memberikan nama yang berbeda kepada mereka.

Untuk mendemonstrasikan masalah ini dengan lebih jelas, pertimbangkan komponen `show-clients` berikut:

```php
<?php // resources/views/components/⚡show-clients.blade.php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Client;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function clients()
    {
        return Client::paginate(10);
    }
};

```

Seperti yang Anda lihat, komponen di atas berisi kumpulan data *clients* yang dipaginasi. Jika pengguna menavigasi ke halaman `2` dari kumpulan hasil ini, URL-nya mungkin akan terlihat seperti berikut:

`http://application.test/?page=2`

Misalkan halaman tersebut juga berisi komponen `show-invoices` yang juga menggunakan pagination. Untuk melacak halaman saat ini dari masing-masing paginator secara independen, Anda perlu menentukan nama untuk paginator kedua seperti berikut:

```php
<?php // resources/views/components/⚡show-invoices.blade.php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Invoice;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function invoices()
    {
        return Invoice::paginate(10, pageName: 'invoices-page');
    }
};

```

Sekarang, karena parameter `pageName` telah ditambahkan ke metode `paginate`, ketika pengguna mengunjungi halaman `2` dari *invoices*, URL akan berisi informasi berikut:

`https://application.test/customers?page=2&invoices-page=2`

Saat menggunakan metode navigasi halaman Livewire pada paginator yang bernama, Anda harus memberikan nama halaman tersebut sebagai parameter tambahan:

```php
$this->setPage(2, pageName: 'invoices-page');

$this->resetPage(pageName: 'invoices-page');

$this->nextPage(pageName: 'invoices-page');

$this->previousPage(pageName: 'invoices-page');

```

## Hooking into page updates

Livewire memungkinkan Anda mengeksekusi kode sebelum dan sesudah halaman diperbarui dengan mendefinisikan salah satu metode berikut di dalam **component** Anda:

```php
<?php // resources/views/components/⚡show-posts.blade.php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    public function updatingPage($page)
    {
        // Berjalan sebelum halaman diperbarui untuk component ini...
    }

    public function updatedPage($page)
    {
        // Berjalan setelah halaman diperbarui untuk component ini...
    }

    #[Computed]
    public function posts()
    {
        return Post::paginate(10);
    }
};

```

### Named paginator hooks

**Hooks** di atas hanya berlaku untuk **paginator** default. Jika Anda menggunakan **named paginator**, Anda harus mendefinisikan metode menggunakan nama **paginator** tersebut.

Sebagai contoh, di bawah ini adalah tampilan **hook** untuk **paginator** bernama `invoices-page`:

```php
public function updatingInvoicesPage($page)
{
    //
}

```

### General paginator hooks

Jika Anda lebih suka tidak mereferensikan nama **paginator** dalam nama metode **hook**, Anda dapat menggunakan alternatif yang lebih umum dan menerima `$pageName` sebagai argumen kedua:

```php
public function updatingPaginators($page, $pageName)
{
    // Berjalan sebelum halaman diperbarui untuk component ini...
}

public function updatedPaginators($page, $pageName)
{
    // Berjalan setelah halaman diperbarui untuk component ini...
}

```

---

## Menggunakan simple theme

Anda dapat menggunakan metode `simplePaginate()` milik Laravel alih-alih `paginate()` untuk kecepatan dan kesederhanaan tambahan.

Saat mempaginasi hasil menggunakan metode ini, hanya tautan navigasi *next* dan *previous* yang akan ditampilkan kepada pengguna, bukan tautan individu untuk setiap nomor halaman:

```php
public function render()
{
    return view('show-posts', [
        'posts' => Post::simplePaginate(10),
    ]);
}

```

Untuk informasi lebih lanjut tentang **simple pagination**, silakan lihat [dokumentasi "simplePaginator" Laravel](https://laravel.com/docs/pagination#simple-pagination).

---

## Menggunakan cursor pagination

Livewire juga mendukung penggunaan **cursor pagination** Laravel — metode pagination yang lebih cepat dan berguna untuk kumpulan data yang besar:

```php
public function render()
{
    return view('show-posts', [
        'posts' => Post::cursorPaginate(10),
    ]);
}

```

Dengan menggunakan `cursorPaginate()` alih-alih `paginate()` atau `simplePaginate()`, **query string** di URL aplikasi Anda akan menyimpan *cursor* yang terenkripsi alih-alih nomor halaman standar. Contohnya:

`https://example.com/posts?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0`

Untuk informasi lebih lanjut tentang **cursor pagination**, silakan lihat [dokumentasi cursor pagination Laravel](https://laravel.com/docs/pagination#cursor-pagination).

---

## Menggunakan Bootstrap alih-alih Tailwind

Jika Anda menggunakan [Bootstrap](https://getbootstrap.com/) alih-alih [Tailwind](https://tailwindcss.com/) sebagai framework CSS aplikasi Anda, Anda dapat mengonfigurasi Livewire untuk menggunakan **pagination views** bergaya Bootstrap alih-alih tampilan Tailwind bawaan.

Untuk mencapainya, atur nilai konfigurasi `pagination_theme` di file `config/livewire.php` aplikasi Anda:

```php
'pagination_theme' => 'bootstrap',

```

> [!info] Mempublikasikan file konfigurasi Livewire
> Sebelum menyesuaikan tema pagination, Anda harus mempublikasikan file konfigurasi Livewire ke direktori `/config` aplikasi Anda dengan menjalankan perintah berikut:
> ```shell
> php artisan livewire:config
> 
> ```
> 
> 

---

## Memodifikasi default pagination views

Jika Anda ingin memodifikasi **pagination views** Livewire agar sesuai dengan gaya aplikasi Anda, Anda dapat melakukannya dengan mempublikasikannya menggunakan perintah berikut:

```shell
php artisan livewire:publish --pagination

```

Setelah menjalankan perintah ini, empat file berikut akan dimasukkan ke dalam direktori `resources/views/vendor/livewire`:

| Nama File View | Deskripsi |
| --- | --- |
| `tailwind.blade.php` | Tema pagination Tailwind standar |
| `tailwind-simple.blade.php` | Tema pagination Tailwind *simple* |
| `bootstrap.blade.php` | Tema pagination Bootstrap standar |
| `bootstrap-simple.blade.php` | Tema pagination Bootstrap *simple* |

Setelah file dipublikasikan, Anda memiliki kontrol penuh atas file-file tersebut. Saat merender tautan pagination menggunakan metode `->links()` di dalam **template** Anda, Livewire akan menggunakan file-file ini alih-alih miliknya sendiri.

---

## Menggunakan custom pagination views

Jika Anda ingin melewati **pagination views** Livewire sepenuhnya, Anda dapat merender tampilan Anda sendiri dengan salah satu dari dua cara berikut:

1. Metode `->links()` di **Blade view** Anda
2. Metode `paginationView()` atau `paginationSimpleView()` di **component** Anda

### Melalui `->links()`

Pendekatan pertama adalah dengan meneruskan nama **Blade view** pagination kustom Anda langsung ke metode `->links()`:

```blade
{{ $posts->links('custom-pagination-links') }}

```

Saat merender tautan pagination, Livewire sekarang akan mencari **view** di `resources/views/custom-pagination-links.blade.php`.

### Melalui `paginationView()` atau `paginationSimpleView()`

Pendekatan kedua adalah dengan mendeklarasikan metode `paginationView` atau `paginationSimpleView` di dalam **component** Anda yang mengembalikan nama **view** yang ingin Anda gunakan:

```php
public function paginationView()
{
    return 'custom-pagination-links-view';
}

public function paginationSimpleView()
{
    return 'custom-simple-pagination-links-view';
}

```

### Contoh pagination view

Di bawah ini adalah contoh sederhana **pagination view** Livewire tanpa gaya (*unstyled*) untuk referensi Anda.

Seperti yang Anda lihat, Anda dapat menggunakan **page navigation helpers** Livewire seperti `$this->nextPage()` langsung di dalam **template** Anda dengan menambahkan `wire:click="nextPage"` pada tombol:

```blade
<div>
    @if ($paginator->hasPages())
        <nav role="navigation" aria-label="Pagination Navigation">
            <span>
                @if ($paginator->onFirstPage())
                    <span>Previous</span>
                @else
                    <button wire:click="previousPage" wire:loading.attr="disabled" rel="prev">Previous</button>
                @endif
            </span>

            <span>
                @if ($paginator->onLastPage())
                    <span>Next</span>
                @else
                    <button wire:click="nextPage" wire:loading.attr="disabled" rel="next">Next</button>
                @endif
            </span>
        </nav>
    @endif
</div>

```

Untuk **loading states** yang bersifat visual saja (seperti perubahan opasitas), Anda dapat menggunakan atribut otomatis `data-loading` milik Livewire dengan class Tailwind sebagai gantinya:

```blade
<button wire:click="nextPage" class="data-loading:opacity-50" rel="next">
    Next
</button>

```

[Pelajari lebih lanjut tentang loading states →](https://www.google.com/search?q=/docs/4.x/loading-states)

---

## See also

* **[Query Parameters](https://www.google.com/search?q=/docs/4.x/url)** — Sinkronisasi status pagination dengan URL
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Tampilkan umpan balik selama perubahan halaman
* **[Computed Properties](https://www.google.com/search?q=/docs/4.x/computed-properties)** — Menanyakan data yang dipaginasi secara efisien
