Indikator pemuatan (**loading indicators**) merupakan bagian penting dalam merancang antarmuka pengguna yang baik. Mereka memberikan umpan balik visual kepada pengguna saat sebuah *request* sedang dikirim ke server, sehingga mereka tahu bahwa mereka sedang menunggu sebuah proses selesai.

> [!tip] Pertimbangkan untuk menggunakan selektor data-loading sebagai gantinya
> Meskipun `wire:loading` sangat bagus untuk skenario muncul/sembunyi yang sederhana, Livewire v4 memperkenalkan atribut `data-loading` otomatis pada elemen yang memicu *network requests*. Pendekatan ini seringkali lebih sederhana dan fleksibel—Anda dapat mengatur gaya *loading states* secara langsung dengan Tailwind tanpa memerlukan direktif `wire:target`, dan ini bekerja dengan mulus bahkan saat mengirimkan *events* ke *components* lain. [Pelajari lebih lanjut tentang data-loading →](https://www.google.com/search?q=/docs/4.x/loading-states)

## Penggunaan dasar

Livewire menyediakan sintaks yang sederhana namun sangat kuat untuk mengontrol indikator pemuatan: `wire:loading`. Menambahkan `wire:loading` ke elemen apa pun akan menyembunyikannya secara default (menggunakan `display: none` di CSS) dan menampilkannya ketika sebuah *request* dikirim ke server.

Berikut adalah contoh dasar formulir pada *component* `CreatePost` dengan `wire:loading` yang digunakan untuk memunculkan pesan pemuatan:

```blade
<form wire:submit="save">
    <button type="submit">Save</button>

    <div wire:loading> Saving post...
    </div>
</form>

```

Ketika pengguna menekan tombol "Save", pesan "Saving post..." akan muncul di bawah tombol selama *action* "save" sedang dijalankan. Pesan tersebut akan hilang ketika respons diterima dari server dan diproses oleh Livewire.

### Menghapus elemen

Sebagai alternatif, Anda dapat menambahkan `.remove` untuk efek sebaliknya, yaitu menampilkan elemen secara default dan menyembunyikannya selama *requests* ke server:

```blade
<div wire:loading.remove>...</div>

```

---

## Beralih class (Toggling classes)

Selain mengatur visibilitas elemen secara keseluruhan, sering kali berguna untuk mengubah gaya elemen yang sudah ada dengan mengaktifkan atau menonaktifkan class CSS selama *requests* ke server. Teknik ini dapat digunakan untuk hal-hal seperti mengubah warna latar belakang, menurunkan opasitas, memicu animasi putar (*spinning animations*), dan banyak lagi.

Berikut adalah contoh sederhana penggunaan class [Tailwind](https://tailwindcss.com/) `opacity-50` untuk membuat tombol "Save" menjadi lebih pudar saat formulir sedang dikirim:

```blade
<button wire:loading.class="opacity-50">Save</button>

```

Seperti halnya memunculkan elemen, Anda dapat melakukan operasi class sebaliknya dengan menambahkan `.remove` pada direktif `wire:loading`. Pada contoh di bawah, class `bg-blue-500` pada tombol akan dihapus saat tombol "Save" ditekan:

```blade
<button class="bg-blue-500" wire:loading.class.remove="bg-blue-500">
    Save
</button>

```

---

## Beralih atribut (Toggling attributes)

Secara default, ketika sebuah formulir dikirim, Livewire akan secara otomatis menonaktifkan tombol *submit* dan menambahkan atribut `readonly` pada setiap elemen input selama formulir sedang diproses.

Namun, selain perilaku default ini, Livewire menawarkan **modifier** `.attr` untuk memungkinkan Anda mengubah atribut lain pada suatu elemen atau mengubah atribut pada elemen yang berada di luar formulir:

```blade
<button
    type="button"
    wire:click="remove"
    wire:loading.attr="disabled"
>
    Remove
</button>

```

Karena tombol di atas bukan tombol *submit*, tombol tersebut tidak akan dinonaktifkan oleh perilaku penanganan formulir default Livewire saat ditekan. Sebagai gantinya, kita menambahkan `wire:loading.attr="disabled"` secara manual untuk mencapai perilaku tersebut.

---

## Menargetkan action tertentu

Secara default, `wire:loading` akan dipicu setiap kali sebuah *component* melakukan *request* ke server.

Namun, pada *components* dengan banyak elemen yang dapat memicu *server requests*, Anda sebaiknya membatasi indikator pemuatan Anda ke *actions* individu.

Sebagai contoh, pertimbangkan formulir "Save post" berikut. Selain tombol "Save" yang mengirimkan formulir, mungkin juga ada tombol "Remove" yang menjalankan *action* "remove" pada *component*.

Dengan menambahkan `wire:target` pada elemen `wire:loading` berikut, Anda dapat menginstruksikan Livewire untuk hanya menampilkan pesan pemuatan saat tombol "Remove" diklik:

```blade
<form wire:submit="save">
    <button type="submit">Save</button>

    <button type="button" wire:click="remove">Remove</button>

    <div wire:loading wire:target="remove">  Removing post...
    </div>
</form>

```

### Menargetkan beberapa action

Anda mungkin berada dalam situasi di mana Anda ingin `wire:loading` bereaksi terhadap beberapa, tetapi tidak semua, *actions* di sebuah halaman. Dalam kasus ini, Anda dapat memasukkan beberapa *actions* ke dalam `wire:target` yang dipisahkan dengan koma. Contohnya:

```blade
<div wire:loading wire:target="save, remove"> 
    Updating post...
</div>

```

Indikator pemuatan ("Updating post...") sekarang hanya akan ditampilkan saat tombol "Remove" atau "Save" ditekan, dan tidak akan muncul saat kolom `$title` sedang dikirim ke server.

### Menargetkan action parameters

Dalam situasi di mana *action* yang sama dipicu dengan parameter yang berbeda dari beberapa tempat di halaman, Anda dapat membatasi `wire:target` lebih jauh ke *action* tertentu dengan memasukkan parameter tambahan. Sebagai contoh, pertimbangkan skenario di mana tombol "Remove" ada untuk setiap postingan di halaman:

```blade
@foreach ($posts as $post)
    <div wire:key="{{ $post->id }}">
        <button wire:click="remove({{ $post->id }})">Remove</button>

        <div wire:loading wire:target="remove({{ $post->id }})">
            Removing post...
        </div>
    </div>
@endforeach

```

Tanpa memasukkan `{{ $post->id }}` ke `wire:target="remove"`, pesan "Removing post..." akan muncul saat tombol apa pun di halaman tersebut diklik. Namun, karena kita memasukkan parameter unik pada setiap instansi `wire:target`, Livewire hanya akan menampilkan pesan pemuatan ketika parameter yang cocok dikirimkan ke *action* "remove".

### Menargetkan pembaruan properti

Livewire juga memungkinkan Anda untuk menargetkan pembaruan properti *component* tertentu dengan memasukkan nama properti ke direktif `wire:target`.

```blade
<input type="text" wire:model.live="username">

<div wire:loading wire:target="username">
    Checking availability...
</div>

```

Pesan "Checking availability..." akan muncul saat server diperbarui dengan nama pengguna baru saat pengguna mengetik di kolom input.

### Mengecualikan target pemuatan tertentu

Terkadang Anda mungkin ingin menampilkan indikator pemuatan untuk setiap *request* Livewire *kecuali* properti atau *action* tertentu. Dalam kasus ini, Anda dapat menggunakan **modifier** `wire:target.except` seperti berikut:

```blade
<div wire:loading wire:target.except="download">...</div>

```

---

## Menyesuaikan properti CSS display

Ketika `wire:loading` ditambahkan ke sebuah elemen, Livewire memperbarui properti CSS `display` pada elemen tersebut untuk menampilkan dan menyembunyikannya. Secara default, Livewire menggunakan `none` untuk menyembunyikan dan `inline-block` untuk menampilkan.

Jika Anda mengubah elemen yang menggunakan nilai *display* selain `inline-block`, seperti `flex`, Anda dapat menambahkan `.flex` pada `wire:loading`:

```blade
<div class="flex" wire:loading.flex>...</div>

```

Berikut adalah daftar lengkap nilai *display* yang tersedia:

* `wire:loading.inline-flex`
* `wire:loading.inline`
* `wire:loading.block`
* `wire:loading.table`
* `wire:loading.flex`
* `wire:loading.grid`

---

## Menunda indikator pemuatan (Delaying)

Pada koneksi yang cepat, pembaruan sering kali terjadi begitu cepat sehingga indikator pemuatan hanya berkedip sebentar di layar sebelum dihapus. Dalam kasus ini, indikator tersebut justru lebih mengganggu daripada membantu.

Oleh karena itu, Livewire menyediakan **modifier** `.delay` untuk menunda tampilan indikator. Jika Anda menambahkan `wire:loading.delay` pada sebuah elemen:

```blade
<div wire:loading.delay>...</div>

```

Elemen tersebut hanya akan muncul jika *request* memakan waktu lebih dari 200 milidetik. Pengguna tidak akan pernah melihat indikator tersebut jika *request* selesai sebelum waktu itu.

Anda dapat menyesuaikan durasi penundaan menggunakan salah satu alias interval berikut:

| Modifier | Durasi |
| --- | --- |
| `.shortest` | 50ms |
| `.shorter` | 100ms |
| `.short` | 150ms |
| `.delay` (default) | 200ms |
| `.long` | 300ms |
| `.longer` | 500ms |
| `.longest` | 1000ms |

## Styling dengan data-loading

Livewire secara otomatis menambahkan atribut `data-loading` ke elemen apa pun yang memicu *network request*. Hal ini memungkinkan Anda untuk mengatur gaya *loading states* secara langsung dengan CSS atau Tailwind tanpa menggunakan direktif `wire:loading`.

### Menggunakan varian atribut data Tailwind

Anda dapat menggunakan varian `data-loading` milik Tailwind untuk menerapkan gaya saat sebuah elemen sedang dalam proses pemuatan (*loading*):

```blade
<button
    wire:click="save"
    class="data-loading:opacity-50 data-loading:pointer-events-none"
>
    Save Changes
</button>

```

Ketika tombol diklik dan *request* sedang berjalan, tombol tersebut akan secara otomatis menjadi setengah transparan dan tidak dapat diklik.

### Menggunakan CSS

Jika Anda tidak menggunakan Tailwind, Anda dapat menargetkan atribut `data-loading` dengan CSS standar:

```css
[data-loading] {
    opacity: 0.5;
    pointer-events: none;
}

button[data-loading] {
    background-color: #ccc;
    cursor: wait;
}

```

### Styling elemen parent dan child

Anda dapat mengatur gaya elemen *parent* saat elemen *child* memiliki `data-loading` menggunakan varian `has-data-loading:`:

```blade
<div class="has-data-loading:opacity-50">
    <button wire:click="save">Save</button>
</div>

```

Atau mengatur gaya elemen *child* dari *parent* yang memiliki `data-loading` menggunakan varian `in-data-loading:`:

```blade
<button wire:click="save">
    <span class="in-data-loading:hidden">Save</span>
    <span class="hidden in-data-loading:block">Saving...</span>
</button>

```

---

## See also

* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Pendekatan modern dengan atribut *data-loading*
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Tampilkan umpan balik selama pemrosesan *action*
* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Tampilkan progres pengiriman formulir

---

## Referensi

```blade
wire:loading
wire:target="action"
wire:target="property"
wire:target.except="action"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.remove` | Tampilkan elemen secara default, sembunyikan saat *loading* |
| `.class="class-name"` | Tambahkan class CSS saat *loading* |
| `.class.remove="class-name"` | Hapus class CSS saat *loading* |
| `.attr="attribute"` | Tambahkan atribut HTML saat *loading* |
| `.delay` | Tunda tampilan indikator selama 200ms |
| `.delay.shortest` | Tunda selama 50ms |
| `.delay.shorter` | Tunda selama 100ms |
| `.delay.short` | Tunda selama 150ms |
| `.delay.long` | Tunda selama 300ms |
| `.delay.longer` | Tunda selama 500ms |
| `.delay.longest` | Tunda selama 1000ms |
| `.inline-flex` | Gunakan nilai display `inline-flex` |
| `.inline` | Gunakan nilai display `inline` |
| `.block` | Gunakan nilai display `block` |
| `.table` | Gunakan nilai display `table` |
| `.flex` | Gunakan nilai display `flex` |
| `.grid` | Gunakan nilai display `grid` |
