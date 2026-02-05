Atribut `#[Url]` menyimpan nilai sebuah properti di dalam *query string* URL, memungkinkan pengguna untuk membagikan dan menandai (*bookmark*) status tertentu dari sebuah halaman.

## Penggunaan dasar

Terapkan atribut `#[Url]` pada properti apa pun yang harus dipertahankan di dalam URL:

```php
<?php // resources/views/components/user/⚡index.blade.php

use Livewire\Attributes\Computed;
use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    #[Url] // [tl! highlight]
    public $search = '';

    #[Computed]
    public function users()
    {
        return User::search($this->search)->get();
    }
};
?>

<div>
    <input type="text" wire:model.live="search" placeholder="Search users...">

    <ul>
        @foreach ($this->users as $user)
            <li wire:key="{{ $user->id }}">{{ $user->name }}</li>
        @endforeach
    </ul>
</div>

```

Ketika seorang pengguna mengetik "bob" ke dalam bidang pencarian, URL akan diperbarui menjadi `https://example.com/users?search=bob`. Jika mereka membagikan URL ini atau me-*refresh* halaman, nilai pencarian tersebut akan tetap ada.

---

## Cara kerjanya

Atribut `#[Url]` melakukan dua hal utama:

1. **Menulis ke URL** - Ketika properti berubah, ia memperbarui *query string*.
2. **Membaca dari URL** - Saat halaman dimuat, ia menginisialisasi properti berdasarkan nilai dari *query string*.

Hal ini menciptakan status (*state*) yang dapat dibagikan dan ditandai untuk komponen Anda.

---

## URL vs Session

Baik `#[Url]` maupun `#[Session]` sama-masing mempertahankan nilai properti, namun dengan pertimbangan yang berbeda:

| Fitur | `#[Url]` | `#[Session]` |
| --- | --- | --- |
| Persisten setelah *refresh* | ✅ | ✅ |
| Persisten saat URL dibagikan | ✅ | ❌ |
| Menjaga URL tetap bersih | ❌ | ✅ |
| Terlihat oleh pengguna | ✅ | ❌ |
| Status dapat dibagikan | ✅ | ❌ |

Gunakan `#[Url]` ketika Anda ingin pengguna bisa membagikan atau menandai status saat ini. Gunakan `#[Session]` ketika status tersebut bersifat privat.

---

## Menggunakan alias

Persingkat atau samarkan nama properti di URL dengan parameter `as`:

```php
new class extends Component {
    #[Url(as: 'q')] // [tl! highlight]
    public $search = '';
};

```

URL akan menampilkan `?q=bob` alih-alih `?search=bob`.

---

## Mengecualikan nilai (Excluding values)

Secara default, Livewire hanya menambahkan parameter *query* ketika nilainya berbeda dari nilai awalnya. Gunakan `except` untuk menyesuaikan ini:

```php
new class extends Component {
    #[Url(except: '')] // [tl! highlight]
    public $search = '';

    public function mount()
    {
        $this->search = auth()->user()->username;
    }
};

```

Sekarang Livewire hanya akan mengecualikan `search` dari URL ketika nilainya berupa string kosong, bukan saat nilainya sama dengan *username* awal.

---

## Riwayat Browser (Browser history)

Secara default, Livewire menggunakan `history.replaceState()` untuk memodifikasi URL tanpa menambahkan entri riwayat browser. Untuk menambahkan entri riwayat (memungkinkan tombol *back* memulihkan nilai *query* sebelumnya), gunakan `history`:

```php
#[Url(history: true)] // [tl! highlight]
public $search = '';

```

Kini, mengeklik tombol *back* pada browser akan memulihkan nilai pencarian sebelumnya, alih-alih berpindah ke halaman sebelumnya.

---

## Kapan harus menggunakan

Gunakan `#[Url]` saat:

* Membangun antarmuka pencarian atau filter.
* Mengimplementasikan navigasi halaman (*pagination*).
* Membuat tampilan yang dapat dibagikan (posisi peta, filter yang dipilih, dll.).
* Memungkinkan pengguna untuk menandai status tertentu.
* Mendukung navigasi tombol *back/forward* browser melalui status-status aplikasi.

---

## Pertimbangan SEO

Parameter *query* diindeks oleh mesin pencari dan disertakan dalam analitik:

* **Bagus untuk SEO** - Setiap kombinasi *query* yang unik menciptakan URL unik yang dapat diindeks.
* **Pelacakan Analitik** - Melacak filter dan pencarian apa yang sering digunakan pengguna.
* **Dapat dibagikan di media sosial** - Parameter *query* akan tetap terjaga saat tautan dibagikan.

---

## Referensi

```php
#[Url(
    ?string $as = null,
    bool $history = false,
    bool $keep = false,
    mixed $except = null,
    mixed $nullable = null,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$as` | `?string` | `null` | Nama kustom untuk parameter di URL. |
| `$history` | `bool` | `false` | Memasukkan perubahan URL ke riwayat browser (mengaktifkan tombol *back*). |
| `$keep` | `bool` | `false` | Menjaga parameter *query* saat berpindah navigasi. |
| `$except` | `mixed` | `null` | Nilai yang dikecualikan agar tidak tampil di URL. |
| `$nullable` | `mixed` | `null` | Nilai yang digunakan saat parameter hilang dari URL. |
