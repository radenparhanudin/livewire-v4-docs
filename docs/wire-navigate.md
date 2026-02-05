Fitur `wire:navigate` milik Livewire membuat navigasi halaman jauh lebih cepat, memberikan pengalaman seperti SPA (*Single Page Application*) bagi pengguna Anda.

Halaman ini adalah referensi sederhana untuk direktif `wire:navigate`. Pastikan untuk membaca [halaman tentang fitur Navigate Livewire](https://www.google.com/search?q=/docs/4.x/navigate) untuk dokumentasi yang lebih lengkap.

Di bawah ini adalah contoh sederhana penambahan `wire:navigate` pada tautan di sebuah *nav bar*:

```blade
<nav>
    <a href="/" wire:navigate>Dashboard</a>
    <a href="/posts" wire:navigate>Posts</a>
    <a href="/users" wire:navigate>Users</a>
</nav>

```

Ketika salah satu dari tautan ini diklik, Livewire akan mencegat klik tersebut dan, alih-alih membiarkan browser melakukan kunjungan halaman penuh (*full page visit*), Livewire akan mengambil halaman tersebut di latar belakang dan menukarnya dengan halaman saat ini (menghasilkan navigasi halaman yang jauh lebih cepat dan mulus).

## Styling active links dengan data-current

Livewire secara otomatis menambahkan atribut `data-current` ke tautan `wire:navigate` mana pun yang cocok dengan URL halaman saat ini. Hal ini memungkinkan Anda untuk mengatur gaya tautan navigasi yang aktif menggunakan CSS atau Tailwind tanpa direktif tambahan apa pun:

```blade
<nav>
    <a href="/" wire:navigate class="data-current:font-bold">Dashboard</a>
    <a href="/posts" wire:navigate class="data-current:font-bold">Posts</a>
    <a href="/users" wire:navigate class="data-current:font-bold">Users</a>
</nav>

```

Atribut `data-current` ditambahkan dan dihapus secara otomatis saat pengguna menavigasi antar halaman. Baca selengkapnya tentang [menyoroti tautan aktif di dokumentasi Navigate](https://www.google.com/search?q=/docs/4.x/navigate%23using-the-data-current-attribute).

## Prefetching halaman saat hover

Dengan menambahkan **modifier** `.hover`, Livewire akan melakukan *pre-fetch* (mengambil data terlebih dahulu) sebuah halaman ketika pengguna mengarahkan kursor (*hover*) di atas tautan. Dengan cara ini, halaman sudah selesai diunduh dari server saat pengguna mengeklik tautan tersebut.

```blade
<a href="/" wire:navigate.hover>Dashboard</a>

```

---

## Pelajari lebih dalam

Untuk dokumentasi yang lebih lengkap mengenai fitur ini, kunjungi [halaman dokumentasi navigate Livewire](https://www.google.com/search?q=/docs/4.x/navigate).

---

## See also

* **[Navigate](https://www.google.com/search?q=/docs/4.x/navigate)** — Panduan lengkap navigasi SPA
* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Membuat *page components* yang mendukung rute (*routable*)
* **[@persist](https://www.google.com/search?q=/docs/4.x/directive-persist)** — Mempertahankan elemen selama navigasi berlangsung

---

## Referensi

```blade
wire:navigate

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.hover` | Melakukan *prefetch* halaman saat pengguna melakukan *hover* di atas tautan |
