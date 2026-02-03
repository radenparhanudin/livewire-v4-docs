Livewire memungkinkan Anda untuk menyertakan **styles** khusus **component** secara langsung di dalam **single-file** dan **multi-file components** Anda. **Styles** ini secara otomatis di-**scoped** ke **component** Anda, mencegahnya bocor ke bagian lain dari aplikasi Anda.

Pendekatan ini mencerminkan cara kerja tag `<script>` di **Livewire components**, memberi Anda cara yang kohesif untuk menjaga PHP, HTML, JavaScript, dan CSS milik **component** Anda tetap di satu tempat.

## Scoped styles

Secara **default**, **styles** yang didefinisikan di dalam **component** Anda di-**scoped** hanya untuk **component** tersebut. Ini berarti **CSS selectors** Anda hanya akan memengaruhi elemen-elemen di dalam **component** Anda, meskipun **selectors** yang sama ada di tempat lain di halaman tersebut.

### Single-file components

Tambahkan tag `<style>` pada tingkat akar (**root level**) dari **single-file component** Anda:

```blade
<?php

use Livewire\Component;

new class extends Component {
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
};
?>

<div>
    <h1 class="title">Count: {{ $count }}</h1>
    <button class="btn" wire:click="increment">+</button>
</div>

<style>
.title {
    color: blue;
    font-size: 2rem;
}

.btn {
    background: indigo;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 0.25rem;
}
</style>

```

**Styles** `.title` dan `.btn` hanya akan diterapkan pada elemen di dalam **component** ini, bukan pada elemen lain di halaman dengan **classes** yang sama.

### Multi-file components

Untuk **multi-file components**, buatlah file CSS dengan nama yang sama dengan **component** Anda:

```
resources/views/components/counter/
├── counter.php
├── counter.blade.php
└── counter.css          # Scoped styles

```

`counter.css`

```css
.title {
    color: blue;
    font-size: 2rem;
}

.btn {
    background: indigo;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 0.25rem;
}

```

## How scoping works

Livewire secara otomatis membungkus **styles** Anda dalam sebuah **selector** yang menargetkan **root element** dari **component** Anda. Di balik layar, CSS Anda ditransformasi menggunakan **CSS nesting**:

```css
/* Apa yang Anda tulis */
.btn { background: blue; }

/* Apa yang disajikan ke browser */
[wire\:name="counter"] {
    .btn { background: blue; }
}

```

Ini menggunakan atribut `wire:name` yang ditambahkan Livewire ke setiap **root element** milik **component**, memastikan **styles** hanya berlaku di dalam **component** tersebut.

### Targeting the component root

Anda dapat menggunakan **selector** `&` untuk menargetkan **root element** dari **component** itu sendiri:

```blade
<style>
& {
    border: 2px solid gray;
    padding: 1rem;
}

.title {
    margin-top: 0;
}
</style>

```

Ini akan menambahkan **border** dan **padding** pada elemen paling luar dari **component**.

## Global styles

Terkadang Anda membutuhkan **styles** yang berlaku secara **global** daripada di-**scoped** ke satu **component**. Tambahkan atribut `global` pada tag **style** Anda:

### Single-file components

```blade
<style global>
body {
    font-family: system-ui, sans-serif;
}

.prose {
    max-width: 65ch;
    line-height: 1.6;
}
</style>

```

### Multi-file components

Buat file dengan ekstensi `.global.css`:

```
resources/views/components/counter/
├── counter.php
├── counter.blade.php
├── counter.css           # Scoped styles
└── counter.global.css    # Global styles

```

## Combining scoped and global styles

Anda dapat menggunakan baik **scoped** maupun **global styles** di dalam **component** yang sama:

```blade
<?php

use Livewire\Component;

new class extends Component {
    // ...
};
?>

<div class="counter">
    <h1 class="title">My Counter</h1>
</div>

<style>
.title {
    color: blue;
}
</style>

<style global>
.counter-page-layout {
    display: grid;
    place-items: center;
}
</style>

```

## Style deduplication

Livewire secara otomatis melakukan **deduplicate** pada **styles** ketika beberapa **instances** dari **component** yang sama muncul di sebuah halaman. **Styles** tersebut hanya dimuat satu kali, tidak peduli berapa banyak **component instances** yang ada.

## When to use component styles

**Gunakan scoped styles saat:**

* **Styling** bersifat spesifik untuk satu **component**.
* Anda ingin menghindari tabrakan nama (**collisions**) **CSS class**.
* Anda sedang membangun **components** yang dapat digunakan kembali (**reusable**) dan mandiri (**self-contained**).

**Gunakan global styles saat:**

* Anda perlu memberikan **style** pada elemen di luar **component** Anda.
* Anda mendefinisikan **utility classes** yang digunakan di banyak **components**.
* Anda menimpa (**overriding**) **styles** dari **third-party library**.

**Gunakan @assets untuk external stylesheets:**

* Saat memuat CSS dari CDN.
* Saat menyertakan **styles** dari **third-party library**.

```blade
@assets
<link rel="stylesheet" href="https://cdn.example.com/library.css">
@endassets

```

## Browser support

**Scoped styles** menggunakan [CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting), yang didukung di semua browser modern (Chrome 120+, Firefox 117+, Safari 17.2+). Untuk dukungan browser yang lebih lama, pertimbangkan untuk menggunakan **CSS preprocessor** atau direktif `@assets` dengan **pre-compiled stylesheets**.

## See also

* **[JavaScript](https://www.google.com/search?q=/docs/4.x/javascript)** — Menggunakan **JavaScript** di dalam **components**
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Format dan organisasi **component**
* **[Alpine](https://www.google.com/search?q=/docs/4.x/alpine)** — Interaktivitas **client-side** dengan **Alpine.js**
