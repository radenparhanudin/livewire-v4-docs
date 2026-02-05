Dalam aplikasi *real-time*, memberikan indikasi visual bahwa perangkat pengguna tidak lagi terhubung ke internet sangatlah membantu.

Sebagai contoh, jika Anda membangun platform blogging dengan Livewire, Anda mungkin ingin memberi tahu pengguna jika mereka sedang *offline* agar mereka tidak menulis draf seluruh postingan blog tanpa kemampuan Livewire untuk menyimpannya ke database.

Livewire menyediakan direktif `wire:offline` untuk kasus-kasus seperti ini. Dengan menambahkan `wire:offline` ke sebuah elemen di dalam **Livewire component**, elemen tersebut akan disembunyikan secara default dan menjadi terlihat saat pengguna kehilangan koneksi:

```blade
<div wire:offline>
    This device is currently offline.
</div>

```

Elemen tersebut akan menghilang kembali ketika koneksi jaringan telah dipulihkan.

---

## Beralih class (Toggling classes)

Menambahkan **modifier** `class` memungkinkan Anda untuk menambahkan sebuah class ke elemen saat pengguna kehilangan koneksi mereka. Class tersebut akan dihapus kembali setelah pengguna kembali *online*:

```blade
<div wire:offline.class="bg-red-300">

```

Atau, menggunakan **modifier** `.remove`, Anda dapat menghapus sebuah class saat pengguna kehilangan koneksi. Dalam contoh ini, class `bg-green-300` akan dihapus dari `<div>` selama pengguna kehilangan koneksi:

```blade
<div class="bg-green-300" wire:offline.class.remove="bg-green-300">

```

---

## Beralih atribut (Toggling attributes)

**Modifier** `.attr` memungkinkan Anda menambahkan atribut ke sebuah elemen saat pengguna kehilangan koneksi. Dalam contoh ini, tombol "Save" akan menjadi nonaktif (*disabled*) selama pengguna kehilangan koneksi:

```blade
<button wire:offline.attr="disabled">Save</button>

```

---

## Referensi

```blade
wire:offline

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.class="class-name"` | Menambahkan class CSS saat *offline* |
| `.class.remove="class-name"` | Menghapus class CSS saat *offline* |
| `.attr="attribute"` | Menambahkan atribut HTML saat *offline* |
