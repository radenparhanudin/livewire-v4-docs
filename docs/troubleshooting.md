Di Livewire HQ, kami berusaha menyingkirkan masalah dari jalur Anda sebelum Anda menemukannya. Namun, terkadang ada beberapa masalah yang tidak dapat kami selesaikan tanpa menimbulkan masalah baru, dan di lain waktu, ada masalah yang tidak dapat kami antisipasi.

Berikut adalah beberapa error dan skenario umum yang mungkin Anda temui di aplikasi Livewire Anda.

## Ketidakcocokan Komponen (Component mismatches)

Saat berinteraksi dengan komponen Livewire di halaman Anda, Anda mungkin menemui perilaku aneh atau pesan error seperti berikut:

```
Error: Component already initialized

```

```
Error: Snapshot missing on Livewire component with id: ...

```

Ada banyak alasan mengapa Anda menemui pesan ini, tetapi yang paling umum adalah lupa menambahkan `wire:key` pada elemen dan komponen di dalam loop `@foreach`.

### Menambahkan `wire:key`

Setiap kali Anda memiliki loop di dalam template Blade menggunakan sesuatu seperti `@foreach`, Anda perlu menambahkan `wire:key` pada tag pembuka elemen pertama di dalam loop tersebut:

```blade
@foreach($posts as $post)
    <div wire:key="{{ $post->id }}"> ...
    </div>
@endforeach

```

Ini memastikan bahwa Livewire dapat melacak elemen yang berbeda dalam loop ketika isi loop tersebut berubah.

Hal yang sama berlaku untuk komponen Livewire di dalam loop:

```blade
@foreach($posts as $post)
    <livewire:show-post :$post :wire:key="$post->id" /> @endforeach

```

Namun, ada skenario jebakan yang mungkin tidak Anda duga:

Ketika Anda memiliki komponen Livewire yang tertanam jauh di dalam loop `@foreach`, Anda **TETAP** perlu menambahkan key padanya. Contohnya:

```blade
@foreach($posts as $post)
    <div wire:key="{{ $post->id }}">
        ...
        <livewire:show-post :$post :wire:key="$post->id" /> ...
    </div>
@endforeach

```

Tanpa key pada komponen Livewire yang bersarang tersebut, Livewire tidak akan bisa mencocokkan komponen di dalam loop tersebut di antara permintaan jaringan (*network requests*).

#### Memberikan awalan pada key (Prefixing keys)

Skenario rumit lainnya yang mungkin Anda hadapi adalah memiliki key yang duplikat di dalam komponen yang sama. Hal ini sering terjadi jika menggunakan ID model sebagai key, yang terkadang bisa bertabrakan.

Berikut adalah contoh di mana kita perlu menambahkan awalan `post-` dan `author-` untuk menandai setiap set key sebagai unik. Jika tidak, jika Anda memiliki model `$post` dan `$author` dengan ID yang sama, Anda akan mengalami tabrakan ID:

```blade
<div>
    @foreach($posts as $post)
        <div wire:key="post-{{ $post->id }}">...</div> @endforeach

    @foreach($authors as $author)
        <div wire:key="author-{{ $author->id }}">...</div> @endforeach
</div>

```

---

## Beberapa Instansi Alpine (Multiple instances of Alpine)

Saat menginstal Livewire, Anda mungkin menemui pesan error seperti berikut:

```
Error: Detected multiple instances of Alpine running

```

```
Alpine Expression Error: $wire is not defined

```

Jika ini terjadi, kemungkinan besar Anda menjalankan dua versi Alpine pada halaman yang sama. Livewire sudah menyertakan bundel Alpine-nya sendiri di balik layar, jadi Anda harus menghapus versi Alpine lainnya pada halaman Livewire di aplikasi Anda.

Satu skenario umum di mana hal ini terjadi adalah menambahkan Livewire ke aplikasi yang sudah menyertakan Alpine. Misalnya, jika Anda menginstal *starter kit* Laravel Breeze dan kemudian menambahkan Livewire setelahnya, Anda akan menemui masalah ini.

Solusinya sederhana: hapus instansi Alpine tambahan tersebut.

### Menghapus Alpine bawaan Laravel Breeze

Jika Anda menginstal Livewire di dalam Laravel Breeze yang sudah ada (versi Blade + Alpine), Anda perlu menghapus baris berikut dari `resources/js/app.js`:

```js
import './bootstrap';

import Alpine from 'alpinejs'; // [tl! remove:4]

window.Alpine = Alpine;

Alpine.start();

```

### Menghapus Alpine versi CDN

Karena Livewire versi 2 ke bawah tidak menyertakan Alpine secara default, Anda mungkin menyertakan CDN Alpine sebagai tag script di bagian *head* layout Anda. Di Livewire v3, Anda dapat menghapus CDN ini sepenuhnya, dan Livewire akan menyediakan Alpine secara otomatis untuk Anda:

```html
    ...
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script> </head>

```

Catatan: Anda juga dapat menghapus plugin Alpine tambahan apa pun, karena Livewire menyertakan semua plugin Alpine kecuali `@alpinejs/ui`.

---

## `@alpinejs/ui` Tidak Ditemukan

Versi Alpine yang dibundel dengan Livewire mencakup semua plugin Alpine KECUALI `@alpinejs/ui`. Jika Anda menggunakan komponen *headless* dari [Alpine Components](https://alpinejs.dev/components), yang bergantung pada plugin ini, Anda mungkin menemui error seperti berikut:

```
Uncaught Alpine: no element provided to x-anchor

```

Untuk memperbaikinya, Anda cukup menyertakan plugin `@alpinejs/ui` sebagai CDN di file layout Anda seperti ini:

```html
    ...
    <script defer src="https://unpkg.com/@alpinejs/ui@3.13.7-beta.0/dist/cdn.min.js"></script> </head>

```

Catatan: pastikan untuk menyertakan versi terbaru dari plugin ini, yang dapat Anda temukan di [halaman dokumentasi komponen apa pun](https://alpinejs.dev/component/headless-dialog/docs).
