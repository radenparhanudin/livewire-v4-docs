Atribut `#[Validate]` menghubungkan aturan validasi dengan properti komponen, memungkinkan validasi otomatis secara real-time dan deklarasi aturan yang lebih bersih.

## Penggunaan dasar

Terapkan atribut `#[Validate]` pada properti yang memerlukan validasi:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Validate('required|min:3')] // [tl! highlight]
    public $title = '';

    #[Validate('required|min:3')] // [tl! highlight]
    public $content = '';

    public function save()
    {
        $this->validate();

        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect('/posts');
    }
};
?>

<div>
    <input type="text" wire:model="title">
    @error('title') <span class="error">{{ $message }}</span> @enderror

    <textarea wire:model="content"></textarea>
    @error('content') <span class="error">{{ $message }}</span> @enderror

    <button wire:click="save">Save Post</button>
</div>

```

Dengan `#[Validate]`, Livewire secara otomatis memvalidasi properti pada setiap pembaruan, memberikan umpan balik instan kepada pengguna.

---

## Cara kerjanya

Saat Anda menambahkan `#[Validate]` ke sebuah properti:

1. **Validasi Otomatis** - Properti divalidasi setiap kali nilainya diperbarui.
2. **Umpan Balik Real-time** - Pengguna melihat error validasi seketika.
3. **Validasi Manual** - Anda tetap memanggil `$this->validate()` sebelum menyimpan untuk memastikan semua properti telah tervalidasi.

---

## Validasi Real-time

Secara default, `#[Validate]` memvalidasi properti saat diperbarui:

```php
#[Validate('required|email|unique:users,email')]
public $email = '';

// Di template menggunakan wire:model.live.blur
<input type="email" wire:model.live.blur="email">

```

Saat pengguna mengisi formulir, mereka menerima umpan balik validasi secara langsung.

---

## Menonaktifkan validasi otomatis

Untuk memvalidasi hanya saat memanggil `$this->validate()` secara eksplisit, gunakan `onUpdate: false`:

```php
#[Validate('required|min:3', onUpdate: false)] // [tl! highlight]
public $title = '';

```

Kini validasi hanya berjalan saat metode simpan dipanggil, bukan pada setiap pembaruan properti.

---

## Nama atribut kustom

Sesuaikan nama kolom dalam pesan validasi:

```php
#[Validate('required', as: 'tanggal lahir')] // [tl! highlight]
public $dob;

```

Pesan error akan menjadi "The tanggal lahir field is required" alih-alih "The dob field is required".

---

## Pesan validasi kustom

Ganti pesan validasi default:

```php
#[Validate('required', message: 'Mohon isi judul postingan')] // [tl! highlight]
public $title;

```

Untuk banyak aturan, gunakan beberapa atribut:

```php
#[Validate('required', message: 'Judul wajib diisi')]
#[Validate('min:3', message: 'Judul ini terlalu pendek')]
public $title;

```

---

## Validasi Array

Validasi properti array beserta isinya:

```php
#[Validate([
    'tasks' => 'required|array|min:1',
    'tasks.*' => 'required|string|min:3',
])]
public $tasks = [];

```

---

## Batasan

> [!warning] Objek Rule tidak didukung
> Atribut PHP tidak dapat menggunakan objek `Rule` Laravel secara langsung. Untuk aturan kompleks seperti `Rule::exists()`, gunakan metode `rules()` sebagai gantinya:
> ```php
> protected function rules() {
>     return ['email' => ['required', Rule::unique('users')->ignore($this->userId)]];
> }
> 
> ```
> 
> 

---

## Kapan harus menggunakan

Gunakan `#[Validate]` ketika:

* Membangun formulir dengan umpan balik validasi real-time.
* Menempatkan aturan validasi bersamaan dengan definisi properti.
* Membuat logika validasi yang sederhana dan mudah dibaca.

Gunakan metode `rules()` ketika:

* Anda memerlukan objek `Rule` Laravel.
* Aturan bergantung pada nilai dinamis.
* Anda menangani validasi kondisional yang kompleks.

---

## Referensi

```php
#[Validate(
    mixed $rule = null,
    ?string $attribute = null,
    ?string $as = null,
    mixed $message = null,
    bool $onUpdate = true,
    bool $translate = true,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$rule` | `mixed` | `null` | Aturan validasi yang diterapkan. |
| `$as` | `?string` | `null` | Nama "ramah" yang ditampilkan di pesan error. |
| `$message` | `mixed` | `null` | Pesan error kustom. |
| `$onUpdate` | `bool` | `true` | Apakah menjalankan validasi saat properti diperbarui. |
| `$translate` | `bool` | `true` | Apakah akan menerjemahkan pesan validasi menggunakan sistem lokalisasi Laravel. |
