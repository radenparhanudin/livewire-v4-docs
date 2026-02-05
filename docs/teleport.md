Livewire memungkinkan Anda untuk melakukan *teleport* bagian dari **template** Anda ke bagian lain dari **DOM** di halaman tersebut sepenuhnya.

Hal ini sangat berguna untuk hal-hal seperti **nested dialogs**. Saat menumpuk satu **dialog** di dalam **dialog** lainnya, *z-index* dari **parent modal** akan diterapkan pada **nested modal**. Ini dapat menyebabkan masalah pada penataan gaya *backdrops* dan *overlays*. Untuk menghindari masalah ini, Anda dapat menggunakan direktif `@teleport` Livewire untuk merender setiap **nested modal** sebagai saudara kandung (*siblings*) dalam **DOM** yang dirender.

Fungsionalitas ini ditenagai oleh [direktif `x-teleport` milik Alpine](https://www.google.com/search?q=%5Bhttps://alpinejs.dev/directives/teleport%5D(https://alpinejs.dev/directives/teleport)).

## Penggunaan dasar

Untuk melakukan *teleport* pada sebagian **template** Anda ke bagian lain dari **DOM**, Anda dapat membungkusnya dalam direktif `@teleport` Livewire.

Di bawah ini adalah contoh penggunaan `@teleport` untuk merender konten **modal dialog** di akhir elemen `<body>` pada halaman:

```blade
<div>
    <div x-data="{ open: false }">
        <button @click="open = ! open">Toggle Modal</button>

        @teleport('body')
            <div x-show="open">
                Modal contents...
            </div>
        @endteleport
    </div>
</div>

```

> [!info]
> Selektor `@@teleport` dapat berupa string apa pun yang biasanya Anda masukkan ke dalam fungsi seperti `document.querySelector()`.
> Anda dapat mempelajari lebih lanjut tentang `document.querySelector()` dengan merujuk pada [dokumentasi MDN](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector).

Sekarang, ketika **template** Livewire di atas dirender di halaman, bagian konten dari **modal** tersebut akan dirender di akhir `<body>`:

```html
<body>
    <div x-show="open">
        Modal contents...
    </div>
</body>

```

> [!warning] Anda harus melakukan teleport ke luar component
> Livewire hanya mendukung proses *teleporting* HTML ke luar **component** Anda. Sebagai contoh, melakukan *teleport* **modal** ke tag `<body>` diperbolehkan, tetapi melakukan *teleport* ke elemen lain di dalam **component** Anda sendiri tidak akan berfungsi.

> [!warning] Teleporting hanya berfungsi dengan satu elemen akar (root)
> Pastikan Anda hanya menyertakan satu elemen akar (*root element*) di dalam pernyataan `@@teleport`.
