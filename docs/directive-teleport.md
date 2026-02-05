Direktif `@teleport` me-render sebagian dari template Anda di lokasi yang berbeda dalam DOM, di luar penempatan normal komponen tersebut.

## Penggunaan dasar

Bungkus konten dengan `@teleport` dan tentukan di mana konten tersebut akan di-render menggunakan CSS selector:

```blade
<div>
    <div x-data="{ open: false }">
        <button @click="open = ! open">Toggle Modal</button>

        @teleport('body')
            <div x-show="open">
                Konten modal...
            </div>
        @endteleport
    </div>
</div>

```

Konten modal akan di-render di bagian akhir elemen `<body>`:

```html
<body>
    <div x-show="open">
        Konten modal...
    </div>
</body>

```

> [!info] CSS Selector apa pun yang valid
> Selector pada `@@teleport` bisa berupa string apa pun yang biasa Anda masukkan ke `document.querySelector()`, seperti `'body'`, `'#modal-root'`, atau `'.modal-container'`.

---

## Mengapa menggunakan teleport?

Teleporting sangat berguna untuk **modals**, **dropdowns**, dan **popovers** yang bersarang (*nested*), di mana gaya (*styles*) induk atau nilai `z-index` dapat mengganggu perenderan yang semestinya.

**Tanpa teleporting:**

```blade
<div style="z-index: 10;">
    <div style="z-index: 20;">
        </div>
</div>

```

**Dengan teleporting:**

```blade
<div style="z-index: 10;">
    @teleport('body')
        <div style="z-index: 20;">
            </div>
    @endteleport
</div>

```

---

## Kasus penggunaan umum

**Modal dialogs:**

```blade
@teleport('body')
    <div class="fixed inset-0 bg-black/50" x-show="showModal">
        <div class="modal">
            </div>
    </div>
@endteleport

```

**Dropdown menus:**

```blade
@teleport('body')
    <div class="absolute" x-show="open" style="top: {{ $top }}px; left: {{ $left }}px;">
        </div>
@endteleport

```

**Toast notifications:**

```blade
@teleport('#notifications-container')
    <div class="toast">
        {{ $message }}
    </div>
@endteleport

```

---

## Batasan Penting

> [!warning] Harus diteleportasi ke luar komponen
> Livewire hanya mendukung teleportasi HTML ke luar komponen Anda. Melakukan teleportasi ke elemen lain di dalam komponen yang sama tidak akan berhasil.

> [!warning] Memerlukan elemen root tunggal
> Hanya sertakan satu elemen *root* tunggal di dalam pernyataan `@@teleport` Anda. Penggunaan beberapa elemen *root* tidak didukung.

**Valid:**

```blade
@teleport('body')
    <div>
        <h2>Judul</h2>
        <p>Konten</p>
    </div>
@endteleport

```

**Tidak Valid:**

```blade
@teleport('body')
    <h2>Judul</h2>
    <p>Konten</p>
@endteleport

```

---

## Didukung oleh Alpine

Fungsionalitas ini menggunakan [direktif `x-teleport` milik Alpine](https://www.google.com/search?q=%5Bhttps://alpinejs.dev/directives/teleport%5D(https://alpinejs.dev/directives/teleport)) di belakang layar.

---

## Referensi

```blade
@teleport(string $selector)
    @endteleport

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$selector` | `string` | *required* | CSS selector yang menentukan tempat perenderan konten (misal: `'body'`, `'#modal-root'`, `'.container'`) |
