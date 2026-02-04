Saat pengguna berinteraksi dengan **components** Livewire Anda, memberikan umpan balik visual selama **network requests** sangat penting untuk pengalaman pengguna yang baik. Livewire secara otomatis menambahkan atribut `data-loading` ke elemen apa pun yang memicu **network request**, sehingga memudahkan Anda untuk mengatur gaya **loading states**.

> [!tip] Lebih baik menggunakan data-loading daripada wire:loading
> Livewire juga menyediakan **directive** [`wire:loading`](https://www.google.com/search?q=/docs/4.x/wire-loading) untuk beralih elemen selama permintaan. Meskipun `wire:loading` lebih sederhana untuk skenario muncul/sembunyi dasar, ia memiliki lebih banyak batasan (memerlukan `wire:target` untuk spesifisitas, tidak bekerja dengan baik dengan **events** lintas **components**, dll.). Untuk sebagian besar kasus penggunaan, Anda sebaiknya lebih memilih menggunakan **selectors** `data-loading` seperti yang ditunjukkan dalam panduan ini.

## Basic usage

Livewire secara otomatis menambahkan atribut `data-loading` ke elemen apa pun yang memicu **network request**. Ini memungkinkan Anda untuk mengatur gaya **loading states** secara langsung dengan CSS atau Tailwind tanpa menggunakan **directives** `wire:loading`.

Berikut adalah contoh sederhana menggunakan tombol dengan `wire:click`:

```blade
<button wire:click="save" class="data-loading:opacity-50">
    Save Changes
</button>

```

Saat tombol diklik dan permintaan sedang berjalan, tombol tersebut akan otomatis menjadi semi-transparan berkat kehadiran atribut `data-loading` pada elemen tersebut.

## How it works

Atribut `data-loading` secara otomatis ditambahkan ke elemen yang memicu **network requests**, termasuk:

* **Actions**: `wire:click="save"`
* **Form submissions**: `wire:submit="create"`
* **Property updates**: `wire:model.live="search"`
* **Events**: `wire:click="$dispatch('refresh')"`

Yang penting, atribut ini ditambahkan bahkan saat mengirimkan **events** yang ditangani oleh **components** lain:

```blade
<button wire:click="$dispatch('refresh-stats')">
    Refresh
</button>

```

Meskipun **event** tersebut diambil oleh **component** yang berbeda, tombol yang mengirimkan **event** tersebut akan tetap menerima atribut `data-loading` selama **network request** berlangsung.

## Styling with Tailwind

Tailwind v4 ke atas menyediakan **selectors** yang kuat untuk bekerja dengan atribut `data-loading`.

### Basic styling

Gunakan varian `data-loading:` dari Tailwind untuk menerapkan gaya saat elemen sedang memuat:

```blade
<button wire:click="save" class="data-loading:opacity-50">
    Save
</button>

```

### Menampilkan elemen selama loading

Untuk menampilkan elemen hanya saat pemuatan aktif, gunakan varian `not-data-loading:hidden`:

```blade
<button wire:click="save">
    Save
</button>

<span class="not-data-loading:hidden">
    Saving...
</span>

```

Pendekatan ini lebih disukai daripada `hidden data-loading:block` karena ia bekerja tanpa mempedulikan tipe tampilan elemen tersebut (**flex, inline, grid**, dll.).

### Styling children

Anda dapat mengatur gaya elemen anak saat induknya memiliki atribut `data-loading` menggunakan varian `in-data-loading:`:

```blade
<button wire:click="save">
    <span class="in-data-loading:hidden">Save</span>
    <span class="not-in-data-loading:hidden">Saving...</span>
</button>

```

> [!warning] Varian in-data-loading berlaku untuk semua leluhur
> Varian `in-data-loading:` akan terpicu jika **leluhur** elemen mana pun (tidak peduli seberapa jauh di atas pohon DOM) memiliki atribut `data-loading`. Ini dapat menyebabkan perilaku yang tidak terduga jika Anda memiliki **loading states** yang bersarang.

### Styling parents

Atur gaya elemen induk saat mereka berisi anak dengan `data-loading` menggunakan varian `has-data-loading:`:

```blade
<div class="has-data-loading:opacity-50">
    <button wire:click="save">Save</button>
</div>

```

Saat tombol diklik, seluruh `div` induk akan menjadi semi-transparan.

### Styling siblings

Anda dapat mengatur gaya elemen saudara (**siblings**) menggunakan utilitas `peer` Tailwind dengan varian `peer-data-loading:`:

```blade
<div>
    <button wire:click="save" class="peer">
        Save
    </button>

    <span class="peer-data-loading:opacity-50">
        Saving...
    </span>
</div>

```

### Complex selectors

Untuk kebutuhan gaya yang lebih tingkat lanjut, Anda dapat menggunakan varian arbitrer untuk menargetkan elemen tertentu:

```blade
<div class="[&[data-loading]>*]:opacity-50" wire:click="save">
    <span>Child 1</span>
    <span>Child 2</span>
</div>

<button class="[&[data-loading]_.icon]:animate-spin" wire:click="save">
    <svg class="icon"></svg>
    Save
</button>

```

Pelajari lebih lanjut tentang varian **state** Tailwind dan **arbitrary selectors** di [dokumentasi Tailwind CSS](https://tailwindcss.com/docs/hover-focus-and-other-states).

## Advantages over wire:loading

Pendekatan atribut `data-loading` menawarkan beberapa keunggulan dibandingkan **directive** `wire:loading` tradisional:

1. **Tidak perlu targeting**: Tidak seperti `wire:loading` yang sering kali memerlukan `wire:target` untuk menentukan **action** mana yang harus direspon, atribut `data-loading` secara otomatis dibatasi lingkupnya pada elemen yang memicu permintaan.
2. **Gaya yang lebih elegan**: Sistem varian Tailwind menyediakan cara yang lebih bersih dan deklaratif untuk mengatur gaya **loading states** secara langsung di dalam **markup** Anda.
3. **Bekerja dengan events**: Atribut ini ditambahkan bahkan saat mengirimkan **events** yang ditangani oleh **components** lain, sesuatu yang sebelumnya sulit dicapai dengan `wire:loading`.
4. **Komposisi yang lebih baik**: Pengaturan gaya dengan varian Tailwind berpadu lebih baik dengan **utility classes** dan **states** lainnya.

## Tailwind 4 requirement

> [!info] Tailwind v4+ diperlukan untuk varian tingkat lanjut
> Varian `in-data-loading:`, `has-data-loading:`, `peer-data-loading:`, dan `not-data-loading:` memerlukan Tailwind CSS v4 atau di atasnya. Jika Anda menggunakan versi Tailwind yang lebih lama, Anda tetap dapat menargetkan atribut `data-loading` menggunakan sintaks `data-loading:` atau CSS standar.

## Using with plain CSS

Jika Anda tidak menggunakan Tailwind, Anda dapat menargetkan atribut `data-loading` dengan CSS standar:

```css
[data-loading] {
    opacity: 0.5;
}

button[data-loading] {
    background-color: #ccc;
}

```

Anda juga dapat menggunakan CSS untuk mengatur gaya elemen anak:

```css
[data-loading] .loading-text {
    display: inline;
}

[data-loading] .default-text {
    display: none;
}

```

---

## See also

* **[wire:loading](https://www.google.com/search?q=/docs/4.x/wire-loading)** — Menampilkan dan menyembunyikan elemen selama permintaan
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Menampilkan umpan balik selama pemrosesan **action**
* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Menunjukkan progres pengiriman formulir
* **[Lazy Loading](https://www.google.com/search?q=/docs/4.x/lazy)** — Menampilkan **loading states** untuk **lazy components**
