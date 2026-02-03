Livewire **components** dapat digunakan sebagai halaman utuh dengan menetapkannya langsung ke **routes**. Hal ini memungkinkan Anda membangun halaman aplikasi yang lengkap menggunakan Livewire **components**, dengan kemampuan tambahan seperti **custom layouts**, **page titles**, dan **route parameters**.

## Routing to components

Untuk melakukan **routing** ke sebuah **component**, gunakan **method** `Route::livewire()` di file `routes/web.php` Anda:

```php
Route::livewire('/posts/create', 'pages::post.create');

```

Saat Anda mengunjungi URL yang ditentukan, **component** tersebut akan di-**render** sebagai halaman lengkap menggunakan **layout** aplikasi Anda.

## Layouts

**Components** yang di-**render** melalui **routes** akan menggunakan **layout file** aplikasi Anda. Secara **default**, Livewire mencari **layout** bernama `layouts::app` yang berlokasi di `resources/views/layouts/app.blade.php`.

Anda dapat membuat file ini jika belum ada dengan menjalankan **command** berikut:

```shell
php artisan livewire:layout

```

**Command** ini akan men-**generate** file bernama `resources/views/layouts/app.blade.php`.

Pastikan Anda telah membuat file Blade di lokasi tersebut dan menyertakan **placeholder** `{{ $slot }}`:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        {{ $slot }}

        @livewireScripts
    </body>
</html>

```

Anda dapat menyesuaikan **default layout** dengan memperbarui opsi **configuration** `component_layout` di file `config/livewire.php` Anda:

```php
'component_layout' => 'layouts::dashboard',

```

### Component-specific layouts

Untuk menggunakan **layout** yang berbeda bagi **component** tertentu, Anda dapat menempatkan **attribute** `#[Layout]` di atas **component class** Anda:

```php
<?php

use Livewire\Attributes\Layout;
use Livewire\Component;

new #[Layout('layouts::dashboard')] class extends Component { 
	// ...
};

```

Alternatif lainnya, Anda dapat menggunakan **method** `->layout()` di dalam **method** `render()` pada **component** Anda:

```php
<?php

use Livewire\Component;

new class extends Component {
	// ...

    public function render()
    {
        return $this->view()
            ->layout('layouts::dashboard'); 
    }
};

```

## Setting the page title

Menetapkan **page titles** yang unik untuk setiap halaman di aplikasi Anda sangat membantu bagi pengguna maupun mesin pencari (SEO).

Untuk menetapkan **custom page title** pada sebuah **page component**, pertama-tama pastikan **layout file** Anda menyertakan judul yang dinamis:

```blade
<head>
    <title>{{ $title ?? config('app.name') }}</title>
</head>

```

Selanjutnya, di atas **Livewire component class** Anda, tambahkan **attribute** `#[Title]` dan masukkan judul halaman Anda:

```php
<?php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Create post')] class extends Component { 
	// ...
};

```

Ini akan mengatur **page title** untuk **component** tersebut. Dalam contoh ini, judul halaman akan menjadi "Create Post" saat **component** di-**render**.

Jika Anda perlu mengirimkan judul yang dinamis, seperti judul yang menggunakan **component property**, Anda dapat menggunakan **fluent method** `->title()` di dalam **method** `render()` pada **component**:

```php
public function render()
{
    return $this->view()
         ->title('Create Post'); 
}

```

## Setting additional layout file slots

Jika **layout file** Anda memiliki **named slots** selain `$slot`, Anda dapat menetapkan kontennya di dalam Blade **view** Anda dengan mendefinisikan `<x-slot>` di luar **root element**. Sebagai contoh, jika Anda ingin mengatur bahasa halaman (**page language**) untuk setiap **component** secara individu, Anda dapat menambahkan **slot** `$lang` yang dinamis ke dalam tag HTML pembuka di **layout file** Anda:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', $lang ?? app()->getLocale()) }}">
    <head>
        </head>
    <body>
        {{ $slot }}
        </body>
</html>

```

Kemudian, di dalam **component view** Anda, definisikan elemen `<x-slot>` di luar **root element**:

```blade
<x-slot:lang>fr</x-slot> <div>
    </div>

```

## Accessing route parameters

Saat bekerja dengan **page components**, Anda mungkin perlu mengakses **route parameters** di dalam Livewire **component** Anda.

Sebagai demonstrasi, pertama, definisikan sebuah **route** dengan parameter di file `routes/web.php` Anda:

```php
Route::livewire('/posts/{id}', 'pages::show-post');

```

Di sini, kita mendefinisikan **route** dengan parameter `id` yang merepresentasikan ID dari sebuah **post**.

Selanjutnya, perbarui Livewire **component** Anda untuk menerima **route parameter** di dalam **method** `mount()`:

```php
<?php

use App\Models\Post;
use Livewire\Component;

new class extends Component {
    public Post $post;

    public function mount($id) 
    {
        $this->post = Post::findOrFail($id);
    }
};

```

Dalam contoh ini, karena nama parameter `$id` sesuai dengan **route parameter** `{id}`, jika URL `/posts/1` dikunjungi, Livewire akan mengirimkan nilai "1" sebagai `$id`.

## Using route model binding

Laravel **route model binding** memungkinkan Anda secara otomatis me-**resolve** Eloquent **models** dari **route parameters**.

Setelah mendefinisikan **route** dengan parameter **model** di file `routes/web.php` Anda:

```php
Route::livewire('/posts/{post}', 'pages::show-post');

```

Anda sekarang dapat menerima **route model parameter** melalui **method** `mount()` pada **component** Anda:

```php
<?php

use App\Models\Post;
use Livewire\Component;

new class extends Component {
    public Post $post;

    public function mount(Post $post) 
    {
        $this->post = $post;
    }
};

```

Livewire tahu untuk menggunakan "**route model binding**" karena **type-hint** `Post` ditambahkan pada parameter `$post` di dalam `mount()`.

Seperti sebelumnya, Anda dapat mengurangi kode **boilerplate** dengan mengabaikan **method** `mount()`:

```php
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post; 
};

```

**Property** `$post` akan secara otomatis diisi dengan **model** yang di-**bound** melalui **route parameter** `{post}`.

---

## See also (Lihat juga)

* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Pelajari tentang cara membuat dan mengorganisir komponen.
* **[Navigate](https://www.google.com/search?q=/docs/4.x/navigate)** — Membangun navigasi seperti SPA antar halaman.
* **[Redirecting](https://www.google.com/search?q=/docs/4.x/redirecting)** — Mengalihkan pengguna setelah pengiriman formulir atau aksi.
* **[Layout Attribute](https://www.google.com/search?q=/docs/4.x/attribute-layout)** — Menentukan layout untuk komponen halaman penuh.
* **[Title Attribute](https://www.google.com/search?q=/docs/4.x/attribute-title)** — Mengatur judul halaman secara dinamis.
