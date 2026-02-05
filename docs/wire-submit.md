Livewire mempermudah penanganan pengiriman formulir (*form submissions*) melalui direktif `wire:submit`. Dengan menambahkan `wire:submit` ke elemen `<form>`, Livewire akan mencegat pengiriman formulir, mencegah perilaku default browser, dan memanggil metode apa pun pada **component** Livewire.

Berikut adalah contoh dasar penggunaan `wire:submit` untuk menangani pengiriman formulir "Create Post":

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}

```

```blade
<form wire:submit="save"> <input type="text" wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>
</form>

```

Pada contoh di atas, saat pengguna mengirimkan formulir dengan mengeklik "Save", `wire:submit` mencegat **event** `submit` dan memanggil **action** `save()` di server.

> [!info] Livewire secara otomatis memanggil `preventDefault()`
> `wire:submit` berbeda dari **event handlers** Livewire lainnya karena secara internal ia memanggil `event.preventDefault()` tanpa memerlukan **modifier** `.prevent`. Hal ini dikarenakan sangat jarang ada kasus di mana Anda mendengarkan **event** `submit` namun TIDAK ingin mencegah perilaku default browser (melakukan pengiriman formulir penuh ke sebuah *endpoint*).

> [!info] Livewire secara otomatis menonaktifkan formulir saat proses pengiriman
> Secara default, ketika Livewire sedang mengirimkan data formulir ke server, ia akan menonaktifkan tombol kirim (*submit buttons*) dan menandai semua input formulir sebagai `readonly`. Dengan cara ini, pengguna tidak dapat mengirimkan formulir yang sama lagi sampai pengiriman awal selesai.

---

## Pelajari lebih dalam

`wire:submit` hanyalah salah satu dari banyak **event listeners** yang disediakan Livewire. Dua halaman berikut menyediakan dokumentasi yang jauh lebih lengkap mengenai penggunaan `wire:submit` dalam aplikasi Anda:

* [Merespons event browser dengan Livewire](https://www.google.com/search?q=/docs/4.x/actions)
* [Membuat formulir di Livewire](https://www.google.com/search?q=/docs/4.x/forms)

---

## See also

* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Tangani pengiriman formulir dengan Livewire
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Proses data formulir di dalam **actions**
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Validasi formulir sebelum pengiriman

---

## Referensi

```blade
wire:submit="methodName"
wire:submit="methodName(param1, param2)"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.prevent` | Mencegah perilaku default browser (otomatis untuk `wire:submit`) |
| `.stop` | Menghentikan propagasi **event** |
| `.self` | Hanya terpicu jika **event** berasal dari elemen ini sendiri |
| `.once` | Memastikan **listener** hanya dipanggil satu kali |
| `.debounce` | Melakukan **debounce** pada handler selama 250ms (gunakan `.debounce.500ms` untuk durasi kustom) |
| `.throttle` | Membatasi (**throttle**) handler minimal setiap 250ms (gunakan `.throttle.500ms` untuk kustom) |
| `.window` | Mendengarkan **event** pada objek `window` |
| `.document` | Mendengarkan **event** pada objek `document` |
| `.passive` | Tidak akan memblokir performa **scroll** |
| `.capture` | Mendengarkan selama fase **capturing** |
| `.renderless` | Melewatkan **re-rendering** setelah **action** selesai |
| `.preserve-scroll` | Mempertahankan posisi **scroll** selama pembaruan |
| `.async` | Mengeksekusi **action** secara paralel alih-alih mengantre |
