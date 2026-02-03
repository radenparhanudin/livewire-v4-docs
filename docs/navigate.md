Banyak aplikasi web modern dibangun sebagai "single page applications" (SPAs). Dalam aplikasi ini, setiap halaman yang dirender oleh aplikasi tidak lagi memerlukan pemuatan ulang halaman browser secara penuh (*full page reload*), menghindari beban pengunduhan ulang aset JavaScript dan CSS pada setiap permintaan.

Alternatif dari *single page application* adalah *multi-page application*. Dalam aplikasi ini, setiap kali pengguna mengklik tautan, halaman HTML yang sepenuhnya baru diminta dan dirender di browser.

Meskipun kebanyakan aplikasi PHP secara tradisional adalah *multi-page applications*, Livewire menawarkan pengalaman *single page application* melalui atribut sederhana yang dapat Anda tambahkan ke tautan di aplikasi Anda: `wire:navigate`.

## Basic usage

Mari kita pelajari contoh penggunaan `wire:navigate`. Di bawah ini adalah file rute Laravel tipikal (`routes/web.php`) dengan tiga Livewire **components** yang didefinisikan sebagai rute:

```php
use App\Livewire\Dashboard;
use App\Livewire\ShowPosts;
use App\Livewire\ShowUsers;

Route::livewire('/', 'pages::dashboard');

Route::livewire('/posts', 'pages::show-posts');

Route::livewire('/users', 'pages::show-users');

```

Dengan menambahkan `wire:navigate` ke setiap tautan di menu navigasi pada setiap halaman, Livewire akan mencegah penanganan standar dari klik tautan dan menggantinya dengan versi miliknya sendiri yang lebih cepat:

```blade
<nav>
    <a href="/" wire:navigate>Dashboard</a>
    <a href="/posts" wire:navigate>Posts</a>
    <a href="/users" wire:navigate>Users</a>
</nav>

```

Berikut adalah rincian tentang apa yang terjadi ketika tautan `wire:navigate` diklik:

* Pengguna mengklik sebuah tautan
* Livewire mencegah browser mengunjungi halaman baru tersebut
* Sebaliknya, Livewire meminta halaman tersebut di latar belakang dan menampilkan **loading bar** di bagian atas halaman
* Ketika HTML untuk halaman baru telah diterima, Livewire mengganti URL halaman saat ini, tag `<title>` dan konten `<body>` dengan elemen-elemen dari halaman baru tersebut

Teknik ini menghasilkan waktu muat halaman yang jauh lebih cepat — seringkali dua kali lebih cepat — dan membuat aplikasi "terasa" seperti *single page application* yang ditenagai JavaScript.

## Redirects

Ketika salah satu Livewire **components** Anda melakukan **redirect** pengguna ke URL lain di dalam aplikasi Anda, Anda juga dapat menginstruksikan Livewire untuk menggunakan fungsionalitas `wire:navigate` untuk memuat halaman baru tersebut. Untuk mencapai ini, berikan argumen `Maps` ke **method** `redirect()`:

```php
return $this->redirect('/posts', navigate: true);

```

Sekarang, alih-alih permintaan halaman penuh yang digunakan untuk **redirect** pengguna ke URL baru, Livewire akan mengganti konten dan URL halaman saat ini dengan yang baru.

## Prefetching links

Secara **default**, Livewire menyertakan strategi halus untuk *prefetch* halaman sebelum pengguna mengklik tautan:

* Pengguna menekan tombol mouse mereka (**mouse down**)
* Livewire mulai meminta halaman tersebut
* Mereka mengangkat tombol mouse untuk menyelesaikan klik (**click**)
* Livewire menyelesaikan permintaan tersebut dan menavigasi ke halaman baru

Mengejutkan bahwa waktu antara pengguna menekan dan mengangkat tombol mouse seringkali cukup waktu untuk memuat setengah atau bahkan seluruh halaman dari server.

Jika Anda menginginkan pendekatan yang lebih agresif untuk *prefetching*, Anda dapat menggunakan **modifier** `.hover` pada tautan:

```blade
<a href="/posts" wire:navigate.hover>Posts</a>

```

**Modifier** `.hover` akan menginstruksikan Livewire untuk melakukan *prefetch* halaman setelah pengguna mengarahkan kursor (**hover**) ke atas tautan selama `60` milidetik.

> [!warning] Prefetching on hover meningkatkan penggunaan server
> Karena tidak semua pengguna akan mengklik tautan yang mereka soroti, menambahkan `.hover` akan meminta halaman yang mungkin tidak dibutuhkan, meskipun Livewire mencoba memitigasi sebagian beban ini dengan menunggu `60` milidetik sebelum melakukan *prefetch* halaman.

## Persisting elements across page visits

Terkadang, ada bagian dari antarmuka pengguna yang perlu Anda pertahankan di antara pemuatan halaman, seperti pemutar audio atau video. Misalnya, dalam aplikasi *podcast*, pengguna mungkin ingin terus mendengarkan sebuah episode saat mereka menelusuri halaman lain.

Anda dapat mencapai ini di Livewire dengan **directive** `@persist`.

Dengan membungkus elemen dengan `@persist` dan memberikannya sebuah nama, saat halaman baru diminta menggunakan `wire:navigate`, Livewire akan mencari elemen pada halaman baru yang memiliki `@persist` yang cocok. Alih-alih mengganti elemen tersebut seperti biasa, Livewire akan menggunakan elemen DOM yang sudah ada dari halaman sebelumnya ke halaman baru, menjaga **state** apa pun di dalam elemen tersebut.

Berikut adalah contoh elemen pemutar `<audio>` yang dipertahankan di seluruh halaman menggunakan `@persist`:

```blade
@persist('player')
    <audio src="{{ $episode->file }}" controls></audio>
@endpersist

```

Jika HTML di atas muncul di kedua halaman — halaman saat ini dan halaman berikutnya — elemen aslinya akan digunakan kembali pada halaman baru. Dalam kasus pemutar audio, pemutaran audio tidak akan terputus saat menavigasi dari satu halaman ke halaman lainnya.

Harap diperhatikan bahwa elemen yang dipertahankan harus diletakkan di luar Livewire **components** Anda. Praktik umum adalah menempatkan elemen yang dipertahankan di **layout** utama Anda, seperti `resources/views/layouts/app.blade.php`.

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        <main>
            {{ $slot }}
        </main>

        @persist('player') <audio src="{{ $episode->file }}" controls></audio>
        @endpersist

        @livewireScripts
    </body>
</html>

```

### Highlighting active links

Anda mungkin terbiasa menyoroti tautan halaman yang sedang aktif di **navbar** menggunakan Blade di sisi server seperti ini:

```blade
<nav>
    <a href="/" class="@if (request->is('/')) font-bold text-zinc-800 @endif">Dashboard</a>
    <a href="/posts" class="@if (request->is('/posts')) font-bold text-zinc-800 @endif">Posts</a>
    <a href="/users" class="@if (request->is('/users')) font-bold text-zinc-800 @endif">Users</a>
</nav>

```

Namun, ini tidak akan bekerja di dalam elemen yang dipertahankan karena mereka digunakan kembali di antara pemuatan halaman. Sebaliknya, Anda memiliki dua opsi untuk menyoroti tautan aktif selama navigasi:

#### Menggunakan atribut `data-current`

Livewire secara otomatis menambahkan atribut `data-current` ke tautan `wire:navigate` apa pun yang cocok dengan halaman saat ini. Ini memungkinkan Anda untuk memberikan gaya pada tautan aktif dengan CSS atau Tailwind tanpa **directives** tambahan apa pun:

```blade
<nav>
    <a href="/dashboard" wire:navigate class="data-current:font-bold data-current:text-zinc-800">Dashboard</a>
    <a href="/posts" wire:navigate class="data-current:font-bold data-current:text-zinc-800">Posts</a>
    <a href="/users" wire:navigate class="data-current:font-bold data-current:text-zinc-800">Users</a>
</nav>

```

Ketika halaman `/posts` dikunjungi, tautan "Posts" akan secara otomatis menerima atribut `data-current` dan diberikan gaya yang sesuai.

Anda juga dapat menggunakan CSS biasa untuk menata tautan aktif:

```css
[data-current] {
    font-weight: bold;
    color: #18181b;
}

```

Jika Anda ingin menonaktifkan perilaku ini sambil tetap menggunakan `wire:navigate`, Anda dapat menambahkan **directive** `wire:current.ignore`:

```blade
<a href="/posts" wire:navigate wire:current.ignore>Posts</a>

```

#### Menggunakan directive `wire:current`

Atau, Anda dapat menggunakan **directive** `wire:current` milik Livewire untuk menambahkan **CSS classes** ke tautan yang sedang aktif:

```blade
<nav>
    <a href="/dashboard" ... wire:current="font-bold text-zinc-800">Dashboard</a>
    <a href="/posts" ... wire:current="font-bold text-zinc-800">Posts</a>
    <a href="/users" ... wire:current="font-bold text-zinc-800">Users</a>
</nav>

```

Sekarang, ketika halaman `/posts` dikunjungi, tautan "Posts" akan memiliki perlakuan *font* yang lebih kuat daripada tautan lainnya.

> [!tip] Pilih data-current untuk kesederhanaan
> Meskipun kedua pendekatan bekerja dengan baik, menggunakan atribut `data-current` seringkali lebih sederhana dan lebih fleksibel karena tidak memerlukan **directive** tambahan dan bekerja secara mulus dengan varian atribut data milik Tailwind.

Baca lebih lanjut di [dokumentasi `wire:current](https://www.google.com/search?q=/docs/4.x/wire-current)`.

### Preserving scroll position

Secara **default**, Livewire akan mempertahankan posisi gulir (**scroll position**) sebuah halaman saat menavigasi bolak-balik antar halaman. Namun, terkadang Anda mungkin ingin mempertahankan posisi gulir dari elemen individual yang Anda pertahankan di antara pemuatan halaman.

Untuk melakukan ini, Anda harus menambahkan `wire:navigate:scroll` ke elemen yang berisi **scrollbar** seperti ini:

```html
@persist('sidebar')
<div class="overflow-y-scroll" wire:navigate:scroll> </div>
@endpersist

```

## JavaScript hooks

Setiap navigasi halaman memicu tiga **lifecycle hooks**:

* `livewire:navigate`
* `livewire:navigating`
* `livewire:navigated`

Penting untuk dicatat bahwa **events** ini di-**dispatch** pada semua tipe navigasi. Ini termasuk navigasi manual menggunakan `Livewire.navigate()`, **redirect** dengan navigasi diaktifkan, serta penekanan tombol kembali (*back*) dan maju (*forward*) di browser.

Berikut adalah contoh mendaftarkan pendengar (**listeners**) untuk setiap **events** ini:

```js
document.addEventListener('livewire:navigate', (event) => {
    // Dipicu ketika sebuah navigasi dimulai.

    // Dapat "dibatalkan" (mencegah navigasi benar-benar dilakukan):
    event.preventDefault()

    // Berisi konteks yang berguna tentang pemicu navigasi:
    let context = event.detail

    // Objek URL dari tujuan navigasi yang dimaksud...
    context.url

    // Boolean [true/false] yang menunjukkan apakah navigasi ini
    // dipicu oleh navigasi kembali/maju (history state)...
    context.history

    // Boolean [true/false] yang menunjukkan apakah ada
    // versi cache dari halaman ini untuk digunakan alih-alih
    // mengambil yang baru via network round-trip...
    context.cached
})

document.addEventListener('livewire:navigating', (e) => {
    // Dipicu ketika HTML baru akan ditukar ke halaman...

    // Ini adalah tempat yang baik untuk mengubah HTML apa pun sebelum halaman
    // dinavigasi pergi...

    // Anda dapat mendaftarkan callback onSwap untuk menjalankan kode setelah
    // HTML baru ditukar ke halaman tetapi sebelum script dimuat.
    // Ini adalah tempat yang baik untuk menerapkan gaya kritis seperti dark mode
    // untuk mencegah flickering...
    e.detail.onSwap(() => {
        // ...
    })
})

document.addEventListener('livewire:navigated', () => {
    // Dipicu sebagai langkah terakhir dari navigasi halaman apa pun...

    // Juga dipicu pada pemuatan halaman awal alih-alih "DOMContentLoaded"...
})

```

> [!warning] Event listeners akan bertahan di seluruh halaman
> Ketika Anda memasang **event listener** ke dokumen, ia tidak akan dihapus saat Anda menavigasi ke halaman yang berbeda. Ini dapat menyebabkan perilaku yang tidak terduga jika Anda memerlukan kode untuk berjalan hanya setelah menavigasi ke halaman tertentu, atau jika Anda menambahkan **event listener** yang sama di setiap halaman. Jika Anda tidak menghapus **event listener** Anda, hal itu dapat menyebabkan pengecualian pada halaman lain saat ia mencari elemen yang tidak ada, atau Anda mungkin berakhir dengan **event listener** yang dieksekusi berkali-kali per navigasi.
> Cara mudah untuk menghapus **event listener** setelah berjalan adalah dengan meneruskan opsi `{once: true}` sebagai parameter ketiga ke fungsi `addEventListener`.
> ```js
> document.addEventListener('livewire:navigated', () => {
>     // ...
> }, { once: true })
> 
> ```
> 
> 

## Manually visiting a new page

Selain `wire:navigate`, Anda dapat secara manual memanggil **method** `Livewire.navigate()` untuk memicu kunjungan ke halaman baru menggunakan JavaScript:

```html
<script>
    // ...

    Livewire.navigate('/new/url')
</script>

```

## Using with analytics software

Saat menavigasi halaman menggunakan `wire:navigate` di aplikasi Anda, tag `<script>` apa pun di dalam `<head>` hanya dievaluasi saat halaman awal dimuat.

Ini menciptakan masalah bagi perangkat lunak analitik seperti [Fathom Analytics](https://usefathom.com/). Alat-alat ini mengandalkan cuplikan (**snippet**) `<script>` yang dievaluasi pada setiap perubahan halaman, bukan hanya yang pertama.

Alat seperti [Google Analytics](https://marketingplatform.google.com/about/analytics/) sudah cukup pintar untuk menangani ini secara otomatis, namun, saat menggunakan Fathom Analytics, Anda harus menambahkan `data-spa="auto"` ke tag script Anda untuk memastikan setiap kunjungan halaman dilacak dengan benar:

```blade
<head>
    @if (! config('app.debug'))
        <script src="https://cdn.usefathom.com/script.js" data-site="ABCDEFG" data-spa="auto" defer></script> @endif
</head>

```

## Script evaluation

Saat menavigasi ke halaman baru menggunakan `wire:navigate`, tampilannya *terasa* seolah browser telah berpindah halaman; namun, dari sudut pandang browser, secara teknis Anda masih berada di halaman asli.

Karena hal ini, **styles** dan **scripts** dijalankan secara normal pada halaman pertama, tetapi pada halaman-halaman berikutnya, Anda mungkin harus menyesuaikan cara Anda menulis JavaScript yang biasanya.

Berikut adalah beberapa peringatan dan skenario yang harus Anda ketahui saat menggunakan `wire:navigate`.

### Jangan mengandalkan `DOMContentLoaded`

Merupakan praktik umum untuk menempatkan JavaScript di dalam **event listener** `DOMContentLoaded` agar kode yang ingin Anda jalankan hanya dieksekusi setelah halaman dimuat sepenuhnya.

Saat menggunakan `wire:navigate`, `DOMContentLoaded` hanya dipicu pada kunjungan halaman pertama, bukan pada kunjungan berikutnya.

Untuk menjalankan kode pada setiap kunjungan halaman, ganti setiap instansi `DOMContentLoaded` dengan `livewire:navigated`:

```js
document.addEventListener('DOMContentLoaded', () => { // [tl! remove]
document.addEventListener('livewire:navigated', () => { // [tl! add]
    // ...
})

```

Sekarang, kode apa pun yang ditempatkan di dalam **listener** ini akan dijalankan pada kunjungan halaman awal, dan juga setelah Livewire selesai menavigasi ke halaman-halaman berikutnya.

Mendengarkan **event** ini sangat berguna untuk hal-hal seperti inisialisasi **third-party libraries**.

### Scripts di dalam `<head>` dimuat satu kali

Jika dua halaman menyertakan tag `<script>` yang sama di dalam `<head>`, **script** tersebut hanya akan dijalankan pada kunjungan halaman awal dan tidak pada kunjungan halaman berikutnya.

```blade
<head>
    <script src="/app.js"></script>
</head>

<head>
    <script src="/app.js"></script>
</head>

```

### Scripts `<head>` baru dievaluasi

Jika halaman berikutnya menyertakan tag `<script>` baru di dalam `<head>` yang tidak ada di dalam `<head>` pada kunjungan halaman awal, Livewire akan menjalankan tag `<script>` baru tersebut.

Dalam contoh di bawah ini, *halaman dua* menyertakan **JavaScript library** baru untuk alat pihak ketiga. Saat pengguna menavigasi ke *halaman dua*, **library** tersebut akan dievaluasi.

```blade
<head>
    <script src="/app.js"></script>
</head>

<head>
    <script src="/app.js"></script>
    <script src="/third-party.js"></script>
</head>

```

> [!info] Head assets bersifat blocking
> Jika Anda menavigasi ke halaman baru yang berisi **asset** seperti `<script src="...">` di dalam tag **head**, **asset** tersebut akan diambil dan diproses sebelum navigasi selesai dan halaman baru ditukar (**swapped in**). Ini mungkin perilaku yang mengejutkan, tetapi ini memastikan bahwa setiap **scripts** yang bergantung pada **assets** tersebut akan memiliki akses langsung ke mereka.

### Reloading saat assets berubah

Merupakan praktik umum untuk menyertakan **version hash** dalam nama file JavaScript utama aplikasi. Ini memastikan bahwa setelah melakukan **deploy** versi baru aplikasi Anda, pengguna akan menerima **JavaScript asset** yang segar, dan bukan versi lama yang disajikan dari **browser cache**.

Namun, karena sekarang Anda menggunakan `wire:navigate` dan setiap kunjungan halaman bukan lagi pemuatan halaman browser yang baru, pengguna Anda mungkin masih menerima JavaScript yang basi setelah **deployments**.

Untuk mencegah hal ini, Anda dapat menambahkan `data-navigate-track` ke tag `<script>` di dalam `<head>`:

```blade
<head>
    <script src="/app.js?id=123" data-navigate-track></script>
</head>

<head>
    <script src="/app.js?id=456" data-navigate-track></script>
</head>

```

Saat pengguna mengunjungi *halaman dua*, Livewire akan mendeteksi **JavaScript asset** yang baru dan memicu **full browser page reload**.

Jika Anda menggunakan [Vite plug-in milik Laravel](https://laravel.com/docs/vite#loading-your-scripts-and-styles) untuk membungkus (**bundle**) dan menyajikan **assets** Anda, Livewire menambahkan `data-navigate-track` ke tag **HTML asset** yang dirender secara otomatis. Anda dapat terus mereferensikan **assets** dan **scripts** Anda seperti biasa:

```blade
<head>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>

```

Livewire akan secara otomatis menyuntikkan `data-navigate-track` ke tag HTML yang dirender.

> [!warning] Hanya perubahan query string yang dilacak
> Livewire hanya akan memuat ulang halaman jika **query string** elemen `[data-navigate-track]` (`?id="456"`) berubah, bukan URI itu sendiri (`/app.js`).

### Scripts di dalam `<body>` dievaluasi ulang

Karena Livewire mengganti seluruh konten `<body>` pada setiap halaman baru, semua tag `<script>` di halaman baru akan dijalankan:

```blade
<body>
    <script>
        console.log('Berjalan di halaman satu')
    </script>
</body>

<body>
    <script>
        console.log('Berjalan di halaman dua')
    </script>
</body>

```

Jika Anda memiliki tag `<script>` di dalam **body** yang hanya ingin dijalankan sekali, Anda dapat menambahkan atribut `data-navigate-once` ke tag `<script>` tersebut dan Livewire hanya akan menjalankannya pada kunjungan halaman awal:

```blade
<script data-navigate-once>
    console.log('Hanya berjalan di halaman satu')
</script>

```

## Customizing the progress bar

Ketika halaman membutuhkan waktu lebih dari 150ms untuk dimuat, Livewire akan menampilkan **progress bar** di bagian atas halaman.

Anda dapat menyesuaikan warna bilah ini atau menonaktifkannya sama sekali di dalam file **config** Livewire (`config/livewire.php`):

```php
'navigate' => [
    'show_progress_bar' => false,
    'progress_bar_color' => '#2299dd',
],

```

## See also

* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Buat **routable page components**
* **[Redirecting](https://www.google.com/search?q=/docs/4.x/redirecting)** — Navigasi secara programatik dari **actions**
* **[@persist](https://www.google.com/search?q=/docs/4.x/directive-persist)** — Pertahankan elemen di seluruh navigasi halaman
* **[wire:navigate](https://www.google.com/search?q=/docs/4.x/wire-navigate)** — Tambahkan navigasi SPA ke tautan
