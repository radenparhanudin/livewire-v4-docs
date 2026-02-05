Atribut `#[Session]` mempertahankan nilai sebuah properti di dalam **session** pengguna, menjaganya tetap ada meskipun halaman di-refresh atau pengguna berpindah navigasi.

## Penggunaan dasar

Terapkan atribut `#[Session]` pada properti apa pun yang ingin dipertahankan dalam **session**:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Livewire\Attributes\Session;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Session] // [tl! highlight]
    public $search = '';

    #[Computed]
    public function posts()
    {
        return $this->search === ''
            ? Post::all()
            : Post::where('title', 'like', "%{$this->search}%")->get();
    }
};
?>

<div>
    <input type="text" wire:model.live="search" placeholder="Search posts...">

    @foreach($this->posts as $post)
        <div wire:key="{{ $post->id }}">{{ $post->title }}</div>
    @endforeach
</div>

```

Setelah pengguna memasukkan nilai pencarian, mereka dapat me-refresh halaman atau pergi ke halaman lain dan kembali lagi—nilai pencarian tersebut akan tetap tersimpan.

---

## Cara kerjanya

Setiap kali properti berubah, Livewire menyimpan nilai barunya di dalam **session** pengguna. Saat **component** dimuat, Livewire mengambil nilai tersebut dari **session** dan menginisialisasi properti dengannya.

Ini menciptakan pengalaman pengguna yang persisten tanpa harus mengubah URL.

---

## Session vs URL

Baik `#[Session]` maupun `#[Url]` sama-masing mempertahankan nilai properti, namun dengan pertimbangan yang berbeda:

| Fitur | `#[Session]` | `#[Url]` |
| --- | --- | --- |
| Persisten setelah refresh | ✅ | ✅ |
| Persisten saat URL dibagikan | ❌ | ✅ |
| Menjaga URL tetap bersih | ✅ | ❌ |
| Terlihat oleh pengguna | ❌ | ✅ |
| State dapat dibagikan | ❌ | ✅ |

Gunakan `#[Session]` ketika Anda menginginkan persistensi tanpa mengotori URL atau ketika **state** tidak seharusnya bisa dibagikan kepada orang lain.

---

## Custom session keys

Secara default, Livewire membuat kunci **session** menggunakan nama komponen dan nama properti. Anda dapat menyesuaikannya:

```php
new class extends Component {
    #[Session(key: 'post_search')] // [tl! highlight]
    public $search = '';
};

```

Properti ini akan disimpan di dalam **session** menggunakan kunci `post_search`.

---

## Dynamic session keys

Anda dapat membuat kunci secara dinamis menggunakan properti lainnya:

```php
new class extends Component {
    public Author $author;

    #[Session(key: 'search-{author.id}')] // [tl! highlight]
    public $search = '';
};

```

Jika `$author->id` adalah `4`, kunci **session** menjadi `search-4`. Hal ini memungkinkan nilai **session** yang berbeda untuk setiap penulis.

---

## Kapan harus menggunakan

Gunakan `#[Session]` ketika:

* Mempertahankan preferensi pengguna (tema, bahasa, status *sidebar*).
* Menjaga status filter/pencarian saat navigasi antar halaman.
* Menyimpan data formulir untuk mencegah kehilangan data saat refresh.
* Menjaga **UI state** tetap privat bagi pengguna tersebut.
* Menghindari URL yang berantakan akibat parameter pencarian (*query parameters*).

---

## Pertimbangan Performa

> [!warning] Jangan menyimpan data dalam jumlah besar
> **Session** Laravel dimuat ke dalam memori pada setiap *request*. Menyimpan terlalu banyak data di **session** pengguna dapat memperlambat seluruh aplikasi untuk pengguna tersebut. Hindari menyimpan *collections* atau objek yang besar.

**Penggunaan yang Baik:**

* Nilai sederhana (string, angka, boolean).
* Array kecil (opsi filter, preferensi).
* ID Model (bukan seluruh objek model).

**Penggunaan yang Buruk:**

* *Collections* berukuran besar.
* Model Eloquent lengkap.
* Data biner atau konten file.

> [!tip] Alternatif: URL persistence
> Jika Anda ingin **state** dapat dibagikan melalui URL atau dapat di-*bookmark*, pertimbangkan untuk menggunakan atribut [`#[Url]`](https://www.google.com/search?q=/docs/4.x/url) sebagai gantinya. Parameter URL mempertahankan status di bilah alamat, sementara properti **session** menjaga URL tetap bersih.

---

## Referensi

```php
#[Session(
    ?string $key = null,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$key` | `?string` | `null` | Kunci session kustom (dibuat otomatis jika tidak diisi). |
