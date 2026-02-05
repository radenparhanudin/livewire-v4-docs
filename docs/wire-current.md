Direktif `wire:current` memungkinkan Anda mendeteksi dan mengatur gaya tautan yang sedang aktif di sebuah halaman dengan mudah.

> [!tip] Pertimbangkan untuk menggunakan data-current sebagai gantinya
> Livewire secara otomatis menambahkan atribut `data-current` ke semua tautan `wire:navigate` yang cocok dengan halaman saat ini. Anda dapat mengatur gaya tautan ini secara langsung dengan varian `data-current:` milik Tailwind atau CSS, tanpa memerlukan direktif `wire:current`. [Pelajari lebih lanjut tentang data-current otomatis →](https://www.google.com/search?q=/docs/4.x/navigate%23using-the-data-current-attribute)

Berikut adalah contoh sederhana penambahan `wire:current` pada tautan di sebuah *navbar* sehingga tautan yang sedang aktif memiliki bobot font yang lebih tebal:

```blade
<nav>
    <a href="/dashboard" wire:current="font-bold text-zinc-800">Dashboard</a>
    <a href="/posts" wire:current="font-bold text-zinc-800">Posts</a>
    <a href="/users" wire:current="font-bold text-zinc-800">Users</a>
</nav>

```

Sekarang, ketika pengguna mengunjungi `/posts`, tautan "Posts" akan memiliki perlakuan font yang lebih kuat dibandingkan tautan lainnya.

Perlu dicatat bahwa `wire:current` berfungsi secara langsung dengan tautan `wire:navigate` dan perubahan halaman, serta secara otomatis menambahkan atribut `data-current` pada tautan yang cocok selain menambahkan class yang ditentukan.

## Exact matching

Secara default, `wire:current` menggunakan strategi pencocokan parsial (*partial matching*), yang berarti direktif ini akan diterapkan jika tautan dan halaman saat ini berbagi bagian awal dari jalur (*path*) URL.

Sebagai contoh, jika tautan adalah `/posts`, dan halaman saat ini adalah `/posts/1`, direktif `wire:current` akan tetap diterapkan.

Jika Anda ingin menggunakan pencocokan persis (*exact matching*), Anda dapat menambahkan **modifier** `.exact` pada direktif tersebut.

Berikut adalah contoh di mana Anda mungkin ingin menggunakan pencocokan persis untuk mencegah tautan "Dashboard" disorot saat pengguna mengunjungi `/posts`:

```blade
<nav>
    <a href="/" wire:current.exact="font-bold">Dashboard</a>
</nav>

```

---

## Strict matching

Secara default, `wire:current` akan menghapus garis miring di akhir (*trailing slashes*) (`/`) dari perbandingannya.

Jika Anda ingin menonaktifkan perilaku ini dan memaksa perbandingan string jalur yang ketat, Anda dapat menambahkan **modifier** `.strict`:

```blade
<nav>
    <a href="/posts/" wire:current.strict="font-bold">Dashboard</a>
</nav>

```

---

## Troubleshooting

Jika `wire:current` tidak mendeteksi tautan saat ini dengan benar, pastikan hal-hal berikut:

* Anda memiliki setidaknya satu **Livewire component** di halaman tersebut, atau telah menuliskan `@livewireScripts` secara manual di dalam *layout* Anda.
* Anda memiliki atribut `href` pada tautan tersebut.

---

## Referensi

```blade
wire:current="classes"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.exact` | Menggunakan pencocokan jalur persis alih-alih pencocokan parsial |
| `.strict` | Memaksa perbandingan jalur yang ketat termasuk garis miring di akhir (*trailing slashes*) |
