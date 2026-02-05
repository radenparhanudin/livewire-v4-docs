Atribut `#[Layout]` menentukan **layout** Blade mana yang harus digunakan oleh sebuah **full-page component**, memungkinkan Anda untuk menyesuaikan **layout** pada setiap komponen secara individu.

## Penggunaan dasar

Terapkan atribut `#[Layout]` pada sebuah **full-page component** untuk menggunakan **layout** tertentu:

```php
<?php // resources/views/pages/posts/⚡index.blade.php

use Livewire\Attributes\Layout;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new #[Layout('layouts::dashboard')] class extends Component { // [tl! highlight]
    #[Computed]
    public function posts()
    {
        return Post::all();
    }
};
?>

<div>
    <h1>Posts</h1>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">{{ $post->title }}</div>
    @endforeach
</div>

```

Komponen ini akan di-*render* menggunakan **layout** `resources/views/layouts/dashboard.blade.php` alih-alih menggunakan default.

---

## Layout default

Secara default, Livewire menggunakan **layout** yang ditentukan dalam file konfigurasi `config/livewire.php` Anda:

```php
'component_layout' => 'layouts::app',

```

Atribut `#[Layout]` menggantikan (*overrides*) default ini untuk komponen-komponen tertentu.

---

## Mengirim data ke layouts

Anda dapat mengirimkan data tambahan ke **layout** Anda menggunakan sintaks array:

```php
new #[Layout('layouts::dashboard', ['title' => 'Posts Dashboard'])] class extends Component { // [tl! highlight]
    // ...
};

```

Di dalam file **layout** Anda, variabel `$title` akan tersedia untuk digunakan:

```blade
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title ?? 'My App' }}</title>
</head>
<body>
    {{ $slot }}
</body>
</html>

```

---

## Alternatif: Menggunakan metode layout()

Alih-alih menggunakan atribut, Anda dapat menggunakan metode `layout()` di dalam metode `render()` Anda:

```php
<?php // resources/views/pages/posts/⚡index.blade.php

use Livewire\Component;

new class extends Component {
    public function render()
    {
        return view('livewire.posts.index')
            ->layout('layouts::dashboard', ['title' => 'Posts']); // [tl! highlight]
    }
};

```

Pendekatan menggunakan atribut jauh lebih bersih untuk *single-file components* yang tidak memerlukan metode `render()`.

---

## Menggunakan layout berbeda per halaman

Pola yang umum digunakan adalah menggunakan **layout** yang berbeda untuk bagian aplikasi yang berbeda pula:

```php
// Halaman Admin
new #[Layout('layouts::admin')] class extends Component { }

// Halaman Marketing
new #[Layout('layouts::marketing')] class extends Component { }

// Halaman Dashboard
new #[Layout('layouts::dashboard')] class extends Component { }

```

---

## Kapan harus menggunakan

Gunakan `#[Layout]` ketika:

* Anda memiliki beberapa **layout** dalam aplikasi Anda (admin, marketing, dashboard, dll.).
* Halaman tertentu memerlukan **layout** yang berbeda dari default.
* Anda sedang membangun sebuah **full-page component** (bukan komponen biasa).
* Anda ingin menjaga konfigurasi **layout** tetap dekat dengan definisi komponen.

> [!info] Hanya untuk full-page components
> Atribut `#[Layout]` hanya berlaku untuk **full-page components**. Komponen biasa yang di-*render* di dalam *view* lain tidak menggunakan **layouts**.

---

## Referensi

```php
#[Layout(
    string $name,
    array $params = [],
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$name` | `string` | *required* | Nama **layout** Blade yang akan digunakan. |
| `$params` | `array` | `[]` | Data tambahan untuk dikirimkan ke **layout**. |
