[AlpineJS](https://alpinejs.dev/) adalah *library* JavaScript ringan yang mempermudah penambahan interaktivitas *client-side* ke halaman web Anda. Ia awalnya dibuat untuk melengkapi alat seperti Livewire di mana utilitas yang berfokus pada JavaScript sangat membantu untuk memberikan sentuhan interaktivitas di sekitar aplikasi Anda.

Livewire menyertakan Alpine secara otomatis (*out of the box*) sehingga tidak perlu menginstalnya ke dalam proyek Anda secara terpisah.

Tempat terbaik untuk mempelajari penggunaan AlpineJS adalah [dokumentasi Alpine](https://alpinejs.dev).

## A Basic Alpine component

Untuk meletakkan dasar bagi sisa dokumentasi ini, berikut adalah salah satu contoh *Alpine component* yang paling sederhana dan informatif. Sebuah "counter" kecil yang menampilkan angka di halaman dan memungkinkan pengguna untuk menambah angka tersebut dengan mengklik sebuah tombol:

```html
<div x-data="{ count: 0 }">
    <h2 x-text="count"></h2>

    <button x-on:click="count++">+</button>
</div>

```

*Alpine component* di atas dapat digunakan di dalam *Livewire component* apa pun di aplikasi Anda tanpa kendala. Livewire bertugas menjaga *state* Alpine tetap konsisten saat terjadi pembaruan *Livewire component*. Intinya, Anda bebas menggunakan *Alpine components* di dalam Livewire seolah-olah Anda menggunakan Alpine dalam konteks non-Livewire lainnya.

## Using Alpine inside Livewire

Mari kita eksplorasi contoh yang lebih nyata dalam menggunakan *Alpine component* di dalam sebuah *Livewire component*.

Di bawah ini adalah *Livewire component* sederhana yang menampilkan detail dari sebuah *post model* dari *database*. Secara *default*, hanya judul postingan yang ditampilkan:

```html
<div>
    <h1>{{ $post->title }}</h1>

    <div x-data="{ expanded: false }">
        <button type="button" x-on:click="expanded = ! expanded">
            <span x-show="! expanded">Show post content...</span>
            <span x-show="expanded">Hide post content...</span>
        </button>

        <div x-show="expanded">
            {{ $post->content }}
        </div>
    </div>
</div>

```

Dengan menggunakan Alpine, kita dapat menyembunyikan konten postingan sampai pengguna menekan tombol "Show post content...". Pada titik tersebut, *property* `expanded` milik Alpine akan diatur menjadi `true` dan konten akan ditampilkan di halaman karena `x-show="expanded"` digunakan untuk memberi Alpine kontrol atas visibilitas konten postingan tersebut.

Ini adalah contoh di mana Alpine sangat unggul: menambahkan interaktivitas ke dalam aplikasi Anda tanpa memicu *Livewire server-roundtrips*.

## Controlling Livewire from Alpine using `$wire`

Salah satu fitur paling kuat yang tersedia bagi Anda sebagai pengembang Livewire adalah `$wire`. *Object* `$wire` adalah *magic object* yang tersedia untuk semua *Alpine components* Anda yang digunakan di dalam Livewire.

Anda dapat menganggap `$wire` sebagai gerbang dari JavaScript ke PHP. Ia memungkinkan Anda untuk mengakses dan memodifikasi *Livewire component properties*, memanggil *Livewire component methods*, dan melakukan lebih banyak lagi; semuanya dari dalam AlpineJS.

### Accessing Livewire properties

Berikut adalah contoh utilitas "character count" sederhana dalam sebuah *form* untuk membuat postingan. Ini akan langsung menunjukkan kepada pengguna berapa banyak karakter yang terkandung di dalam konten postingan mereka saat mereka mengetik:

```html
<form wire:submit="save">
    <input wire:model="content" type="text">

    <small>
        Character count: <span x-text="$wire.content.length"></span> </small>

    <button type="submit">Save</button>
</form>

```

Seperti yang Anda lihat, `x-text` pada contoh di atas digunakan agar Alpine dapat mengontrol konten teks dari elemen `<span>`. `x-text` menerima ekspresi JavaScript apa pun di dalamnya dan secara otomatis bereaksi saat ada ketergantungan yang diperbarui. Karena kita menggunakan `$wire.content` untuk mengakses nilai `$content`, Alpine akan secara otomatis memperbarui konten teks setiap kali `$wire.content` diperbarui dari Livewire; dalam hal ini melalui `wire:model="content"`.

### Mutating Livewire properties

Berikut adalah contoh penggunaan `$wire` di dalam Alpine untuk menghapus bidang "title" pada sebuah *form* pembuatan postingan.

```html
<form wire:submit="save">
    <input wire:model="title" type="text">

    <button type="button" x-on:click="$wire.title = ''">Clear</button> <button type="submit">Save</button>
</form>

```

Saat pengguna mengisi formulir Livewire di atas, mereka dapat menekan "Clear" dan bidang judul akan dikosongkan tanpa mengirimkan *network request* dari Livewire. Interaksinya akan terasa "instan".

Berikut penjelasan singkat tentang apa yang terjadi:

* `x-on:click` memberitahu Alpine untuk mendengarkan klik pada elemen tombol.
* Saat diklik, Alpine menjalankan ekspresi JS yang diberikan: `$wire.title = ''`.
* Karena `$wire` adalah *magic object* yang mewakili *Livewire component*, semua *properties* dari *component* Anda dapat diakses atau diubah langsung dari JavaScript.
* `$wire.title = ''` menetapkan nilai `$title` di *Livewire component* Anda menjadi string kosong.
* Utilitas Livewire seperti `wire:model` akan langsung bereaksi terhadap perubahan ini, semuanya tanpa mengirimkan *server-roundtrip*.
* Pada *Livewire network request* berikutnya, *property* `$title` akan diperbarui menjadi string kosong di *backend*.

### Calling Livewire methods

Alpine juga dapat dengan mudah memanggil *Livewire methods*/*actions* apa pun hanya dengan memanggilnya langsung pada `$wire`.

Berikut adalah contoh penggunaan Alpine untuk mendengarkan *event* "blur" pada sebuah *input* dan memicu penyimpanan formulir. *Event* "blur" di-dispatch oleh browser saat pengguna menekan "tab" untuk memindahkan fokus dari elemen saat ini ke elemen berikutnya di halaman:

```html
<form wire:submit="save">
    <input wire:model="title" type="text" x-on:blur="$wire.save()">  <button type="submit">Save</button>
</form>

```

Biasanya, Anda cukup menggunakan `wire:model.live.blur="title"` dalam situasi ini, namun sebagai tujuan demonstrasi, ini menunjukkan bagaimana Anda dapat mencapainya menggunakan Alpine.

#### Passing parameters

Anda juga dapat meneruskan *parameters* ke *Livewire methods* dengan mengirimkannya langsung ke pemanggilan *method* `$wire`.

Pertimbangkan sebuah *component* dengan *method* `deletePost()` seperti ini:

```php
public function deletePost($postId)
{
    $post = Post::find($postId);

    // Otorisasi apakah user bisa menghapus...
    auth()->user()->can('update', $post);

    $post->delete();
}

```

Sekarang, Anda dapat meneruskan *parameter* `$postId` ke *method* `deletePost()` dari Alpine seperti ini:

```html
<button type="button" x-on:click="$wire.deletePost(1)">

```

Secara umum, sesuatu seperti `$postId` akan dihasilkan di Blade. Berikut contoh penggunaan Blade untuk menentukan `$postId` mana yang diteruskan Alpine ke dalam `deletePost()`:

```html
@foreach ($posts as $post)
    <button type="button" wire:key="{{ $post->id }}" x-on:click="$wire.deletePost({{ $post->id }})">
        Delete "{{ $post->title }}"
    </button>
@endforeach

```

Jika ada tiga postingan di halaman, *template* Blade di atas akan dirender menjadi sesuatu seperti berikut di browser:

```html
<button type="button" x-on:click="$wire.deletePost(1)">
    Delete "The power of walking"
</button>

<button type="button" x-on:click="$wire.deletePost(2)">
    Delete "How to record a song"
</button>

<button type="button" x-on:click="$wire.deletePost(3)">
    Delete "Teach what you learn"
</button>

```

Seperti yang Anda lihat, kita telah menggunakan Blade untuk merender ID postingan yang berbeda ke dalam ekspresi `x-on:click` Alpine.

#### Blade parameter "gotchas"

Ini adalah teknik yang sangat kuat, tetapi bisa membingungkan saat membaca *template* Blade Anda. Mungkin sulit untuk mengetahui bagian mana yang merupakan Blade dan bagian mana yang merupakan Alpine pada pandangan pertama. Oleh karena itu, sangat membantu untuk memeriksa HTML yang dirender di halaman untuk memastikan apa yang Anda harapkan dirender adalah akurat.

Berikut contoh yang sering membingungkan orang:

Katakanlah, alih-alih ID, *Post model* Anda menggunakan UUID untuk indeks (ID adalah integer, dan UUID adalah string karakter yang panjang).

Jika kita merender berikut ini sama seperti yang kita lakukan dengan ID, akan ada masalah:

```html
<button
    type="button"
    x-on:click="$wire.deletePost({{ $post->uuid }})"
>

```

*Template* Blade di atas akan merender berikut ini di HTML Anda:

```html
<button
    type="button"
    x-on:click="$wire.deletePost(93c7b04c-c9a4-4524-aa7d-39196011b81a)"
>

```

Perhatikan kurangnya tanda kutip di sekitar string UUID? Saat Alpine mengevaluasi ekspresi ini, JavaScript akan melemparkan kesalahan: "Uncaught SyntaxError: Invalid or unexpected token".

Untuk memperbaikinya, kita perlu menambahkan tanda kutip di sekitar ekspresi Blade seperti ini:

```html
<button
    type="button"
    x-on:click="$wire.deletePost('{{ $post->uuid }}')"
>

```

Sekarang *template* di atas akan dirender dengan benar dan semuanya akan berjalan sesuai harapan:

```html
<button
    type="button"
    x-on:click="$wire.deletePost('93c7b04c-c9a4-4524-aa7d-39196011b81a')"
>

```

### Refreshing a component

Anda dapat dengan mudah menyegarkan (*refresh*) sebuah *Livewire component* (memicu *network roundtrip* untuk merender ulang *Blade view* milik *component*) menggunakan `$wire.$refresh()`:

```html
<button type="button" x-on:click="$wire.$refresh()">

```

## Sharing state using `$wire.entangle`

> [!warning] Anda mungkin tidak membutuhkan ini
> Dalam hampir semua kasus, Anda harus menggunakan `$wire` untuk secara langsung mengakses *Livewire properties* dari Alpine alih-alih menggunakan `$wire.entangle()`. *Entangling* membuat *state* ganda yang dapat menyebabkan masalah prediktabilitas dan performa. API ini dipertahankan untuk kompatibilitas ke belakang tetapi tidak disarankan untuk kode baru.
> **Jangan gunakan direktif Blade `@@entangle**` - ini telah didepresiasi dan menyebabkan masalah saat menghapus elemen DOM.

Untuk kasus langka di mana Anda memerlukan sinkronisasi *state* dua arah (*bidirectional*) antara Alpine dan Livewire, Anda dapat menggunakan `$wire.entangle()`:

```blade
<div x-data="{ open: $wire.entangle('showDropdown') }">
    <button x-on:click="open = true">Show More...</button>

    <ul x-show="open">
        <li><button wire:click="archive">Archive</button></li>
    </ul>
</div>

```

Secara *default*, perubahan ditunda (*deferred*) sampai *Livewire request* berikutnya. Gunakan `.live` untuk sinkronisasi segera:

```blade
<div x-data="{ open: $wire.entangle('showDropdown').live }">

```

## Using the `@js` directive

Jika Anda perlu mengeluarkan data PHP untuk digunakan di Alpine secara langsung, Anda dapat menggunakan direktif `@js`.

```blade
<div x-data="{ posts: @js($posts) }">
    ...
</div>

```

## Manually bundling Alpine in your JavaScript build

Secara *default*, JavaScript Livewire dan Alpine disuntikkan ke setiap halaman Livewire secara otomatis.

Ini ideal untuk pengaturan yang lebih sederhana, namun Anda mungkin ingin menyertakan *Alpine components*, *stores*, dan *plugins* Anda sendiri ke dalam proyek Anda.

Untuk menyertakan Livewire dan Alpine melalui *JavaScript bundle* Anda sendiri di sebuah halaman sangatlah mudah.

Pertama, Anda harus menyertakan direktif `@livewireScriptConfig` di file *layout* Anda seperti ini:

```blade
<html>
<head>
    @livewireStyles
    @vite(['resources/js/app.js'])
</head>
<body>
    {{ $slot }}

    @livewireScriptConfig </body>
</html>

```

Ini memungkinkan Livewire untuk menyediakan *bundle* Anda dengan konfigurasi tertentu yang dibutuhkannya agar aplikasi Anda berjalan dengan benar.

Sekarang Anda dapat mengimpor Livewire dan Alpine di file `resources/js/app.js` Anda seperti ini:

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

// Daftarkan Alpine directives, components, atau plugins apa pun di sini...

Livewire.start()

```

Berikut adalah contoh pendaftaran *Alpine directive* kustom yang disebut "x-clipboard" di aplikasi Anda:

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

Alpine.directive('clipboard', (el) => {
    let text = el.textContent

    el.addEventListener('click', () => {
        navigator.clipboard.writeText(text)
    })
})

Livewire.start()

```

Sekarang direktif `x-clipboard` akan tersedia untuk semua *Alpine components* di aplikasi Livewire Anda.

## See also

* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Akses *Livewire properties* dari Alpine menggunakan `$wire`
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Panggil *Livewire actions* dari Alpine
* **[JavaScript](https://www.google.com/search?q=/docs/4.x/javascript)** — Jalankan *custom JavaScript* di dalam *components*
* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — *Dispatch* dan dengarkan *events* dengan Alpine
