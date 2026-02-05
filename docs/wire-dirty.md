Pada halaman HTML tradisional yang berisi formulir, formulir tersebut hanya akan dikirimkan ketika pengguna menekan tombol "Submit".

Namun, Livewire mampu melakukan lebih dari sekadar pengiriman formulir tradisional. Anda dapat melakukan validasi input formulir secara *real-time* atau bahkan menyimpan formulir saat pengguna mengetik.

Dalam skenario pembaruan "*real-time*" ini, sangat berguna untuk memberi sinyal kepada pengguna ketika formulir atau sebagian dari formulir telah diubah, tetapi belum disimpan ke database.

Ketika sebuah formulir berisi input yang belum disimpan, formulir tersebut dianggap "**dirty**" (kotor). Formulir tersebut hanya akan menjadi "**clean**" (bersih) ketika sebuah *network request* telah dipicu untuk menyinkronkan *state* server dengan *state* di sisi klien.

## Penggunaan dasar

Livewire memungkinkan Anda untuk beralih elemen visual pada halaman dengan mudah menggunakan direktif `wire:dirty`.

Dengan menambahkan `wire:dirty` ke sebuah elemen, Anda menginstruksikan Livewire untuk hanya menampilkan elemen tersebut ketika *state* di sisi klien berbeda dengan *state* di sisi server.

Sebagai demonstrasi, berikut adalah contoh formulir `UpdatePost` yang berisi indikasi visual "Unsaved changes..." untuk memberi tahu pengguna bahwa formulir berisi input yang belum disimpan:

```blade
<form wire:submit="update">
    <input type="text" wire:model="title">

    <button type="submit">Update</button>

    <div wire:dirty>Unsaved changes...</div> </form>

```

Karena `wire:dirty` telah ditambahkan ke pesan "Unsaved changes...", pesan tersebut akan disembunyikan secara default. Livewire akan secara otomatis menampilkan pesan tersebut saat pengguna mulai mengubah input formulir.

Ketika pengguna mengirimkan formulir, pesan tersebut akan menghilang kembali karena data server / klien sudah sinkron kembali.

### Menghapus elemen

Dengan menambahkan **modifier** `.remove` ke `wire:dirty`, Anda justru dapat menampilkan elemen secara default dan hanya menyembunyikannya ketika **component** berada dalam *state* "**dirty**":

```blade
<div wire:dirty.remove>The data is in-sync...</div>

```

---

## Menargetkan pembaruan properti

Bayangkan Anda menggunakan `wire:model.live.blur` untuk memperbarui properti di server segera setelah pengguna meninggalkan kolom input. Dalam skenario ini, Anda dapat memberikan indikasi "**dirty**" hanya untuk properti tersebut dengan menambahkan `wire:target` ke elemen yang berisi direktif `wire:dirty`.

Berikut adalah contoh untuk hanya menampilkan indikasi *dirty* ketika properti `title` telah diubah:

```blade
<form wire:submit="update">
    <input wire:model.live.blur="title">

    <div wire:dirty wire:target="title">Unsaved title...</div> <button type="submit">Update</button>
</form>

```

---

## Beralih class (Toggling classes)

Seringkali, alih-alih beralih elemen secara keseluruhan, Anda mungkin ingin beralih class CSS individu pada sebuah input ketika *state*-nya "**dirty**".

Di bawah ini adalah contoh di mana pengguna mengetik ke dalam kolom input dan *border* menjadi kuning, menunjukkan *state* "belum disimpan". Kemudian, ketika pengguna berpindah dari kolom tersebut (*tab away*), *border* dihapus, menunjukkan bahwa *state* telah disimpan di server:

```blade
<input wire:model.live.blur="title" wire:dirty.class="border-yellow-500">

```

---

## Menggunakan ekspresi `$dirty`

Selain direktif `wire:dirty`, Anda dapat memeriksa *dirty state* secara terprogram menggunakan ekspresi `$dirty` di dalam direktif Livewire atau `$wire.$dirty()` di Alpine.

### Memeriksa apakah seluruh component kotor

Untuk memeriksa apakah ada properti pada **component** yang memiliki perubahan yang belum disimpan:

```blade
<div wire:show="$dirty">You have unsaved changes</div>

```

### Memeriksa apakah properti tertentu kotor

Untuk memeriksa apakah properti tertentu telah dimodifikasi:

```blade
<div wire:show="$dirty('title')">Title has been modified</div>

```

Anda juga dapat memeriksa properti yang bersarang (*nested properties*):

```blade
<div wire:show="$dirty('user.name')">Name has been modified</div>

```

### Logika kondisional berdasarkan dirty state

Anda dapat menggunakan `$wire.$dirty()` di Alpine untuk menjalankan logika secara kondisional:

```blade
<button x-on:click="$wire.$dirty('title') && $wire.save()">
    Save Title
</button>

```

Atau menerapkan class kondisional dengan Alpine:

```blade
<input
    wire:model="email"
    :class="$wire.$dirty('email') && 'border-yellow-500'"
>

```

---

## Referensi

```blade
wire:dirty
wire:target="property"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.remove` | Tampilkan elemen secara default, sembunyikan saat *dirty* |
| `.class="class-name"` | Tambahkan class CSS saat *dirty* |

### Ekspresi `$dirty`

| Ekspresi | Deskripsi |
| --- | --- |
| `$dirty` | Mengembalikan `true` jika ada properti yang memiliki perubahan belum disimpan |
| `$dirty('property')` | Mengembalikan `true` jika properti yang ditentukan memiliki perubahan belum disimpan |
| `$dirty(['title', 'description'])` | Mengembalikan `true` jika salah satu properti yang ditentukan memiliki perubahan belum disimpan |

Dapat digunakan dalam direktif Livewire seperti `wire:show="$dirty"` atau di Alpine sebagai `$wire.$dirty()`.
