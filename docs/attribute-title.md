Atribut `#[Title]` mengatur judul halaman (*page title*) untuk **full-page Livewire components**.

## Penggunaan dasar

Terapkan atribut `#[Title]` pada **full-page component** untuk mengatur judulnya:

```php
<?php // resources/views/pages/posts/⚡create.blade.php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Create Post')] class extends Component { // [tl! highlight]
    public $title = '';
    public $content = '';

    public function save()
    {
        // Simpan postingan...
    }
};
?>

<div>
    <h1>Create a New Post</h1>

    <input type="text" wire:model="title" placeholder="Post title">
    <textarea wire:model="content" placeholder="Post content"></textarea>

    <button wire:click="save">Save Post</button>
</div>

```

Tab browser akan menampilkan "Create Post" sebagai judul halaman.

---

## Konfigurasi layout

Agar atribut `#[Title]` dapat berfungsi, file **layout** Anda harus menyertakan variabel `$title`:

```blade
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title ?? 'My App' }}</title> </head>
<body>
    {{ $slot }}
</body>
</html>

```

Penggunaan `?? 'My App'` menyediakan judul cadangan (*fallback*) jika tidak ada judul yang ditentukan.

---

## Judul dinamis (Dynamic titles)

Untuk judul dinamis yang menggunakan properti **component**, gunakan metode `title()` di dalam metode `render()`:

```php
<?php // resources/views/pages/posts/⚡edit.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function mount($id)
    {
        $this->post = Post::findOrFail($id);
    }

    public function render()
    {
        return $this->view()
            ->title("Edit: {$this->post->title}"); // [tl! highlight]
    }
};
?>

<div>
    <h1>Edit Post</h1>
    </div>

```

Judul tersebut akan menyertakan judul postingan secara dinamis.

---

## Menggabungkan dengan layouts

Anda dapat menggunakan atribut `#[Title]` dan `#[Layout]` secara bersamaan:

```php
<?php // resources/views/pages/posts/⚡create.blade.php

use Livewire\Attributes\Layout;
use Livewire\Attributes\Title;
use Livewire\Component;

new
#[Layout('layouts.admin')]
#[Title('Create Post')]
class extends Component {
    // ...
};

```

Komponen ini akan menggunakan **layout** admin dengan "Create Post" sebagai judulnya.

---

## Kapan harus menggunakan

Gunakan **atribut `#[Title]**` ketika:

* Membangun **full-page components**.
* Anda menginginkan definisi judul yang bersih dan deklaratif.
* Judul bersifat statis atau jarang berubah.
* Anda mengikuti praktik terbaik SEO.

Gunakan **metode `title()**` ketika:

* Judul bergantung pada properti **component**.
* Anda perlu menghitung judul secara dinamis.
* Judul berubah berdasarkan *state* **component**.

---

## Pertimbangan SEO

Judul halaman yang baik sangat krusial untuk SEO:

* **Deskriptif** – "Edit Post: Dasar-dasar Laravel" lebih baik daripada sekadar "Edit".
* **Singkat dan Jelas** – Targetkan 50-60 karakter untuk menghindari pemotongan di hasil pencarian.
* **Sertakan Kata Kunci** – Membantu mesin pencari memahami konten halaman Anda.
* **Unik** – Setiap halaman harus memiliki judul yang berbeda.

---

## Hanya untuk full-page components

> [!info] Khusus Full-page Components
> Atribut `#[Title]` hanya berfungsi untuk **full-page components** yang diakses melalui *routing*. Komponen biasa yang di-*render* di dalam *view* lain tidak menggunakan judul ini—mereka mewarisi judul dari halaman induknya (*parent page*).

---

## Referensi

```php
#[Title(
    string $content,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$content` | `string` | *required* | Teks yang akan ditampilkan di bilah judul browser. |
