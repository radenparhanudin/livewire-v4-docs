Atribut `#[Modelable]` menetapkan sebuah properti di dalam **child component** agar dapat dihubungkan (*bound*) dari **parent component** menggunakan `wire:model`.

## Penggunaan dasar

Terapkan atribut `#[Modelable]` pada properti di **child component** agar properti tersebut dapat dihubungkan:

```php
<?php // resources/views/components/⚡todo-input.blade.php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable] // [tl! highlight]
    public $value = '';
};
?>

<div>
    <input type="text" wire:model="value">
</div>

```

Sekarang **parent component** dapat terhubung ke **child component** ini sama seperti elemen input lainnya:

```php
<?php // resources/views/components/⚡todos.blade.php

use Livewire\Component;

new class extends Component {
    public $todo = '';

    public function addTodo()
    {
        // Gunakan $this->todo di sini...
    }
};
?>

<div>
    <livewire:todo-input wire:model="todo" /> <button wire:click="addTodo">Add Todo</button>
</div>

```

Ketika pengguna mengetik di komponen `todo-input`, properti `$todo` milik **parent** akan diperbarui secara otomatis.

---

## Cara kerjanya

Tanpa `#[Modelable]`, Anda perlu menangani komunikasi dua arah antara **parent** dan **child** secara manual:

```php
// Tanpa #[Modelable] - pendekatan manual
<livewire:todo-input
    :value="$todo"
    @input="todo = $event.value"
/>

```

Atribut `#[Modelable]` menyederhanakan hal ini dengan mengizinkan `wire:model` bekerja langsung pada komponen tersebut.

---

## Membangun komponen input yang dapat digunakan kembali

`#[Modelable]` sangat cocok untuk membuat komponen input kustom yang terasa seperti input HTML bawaan:

```php
<?php // resources/views/components/⚡date-picker.blade.php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable]
    public $date = '';
};
?>

<div>
    <input
        type="date"
        wire:model="date"
        class="border rounded px-3 py-2"
    >
</div>

```

```blade
{{-- Penggunaan di parent --}}
<livewire:date-picker wire:model="startDate" />
<livewire:date-picker wire:model="endDate" />

```

---

## Modifiers

**Parent** dapat menggunakan **modifiers** `wire:model`, dan mereka akan bekerja sebagaimana mestinya:

```blade
{{-- Pembaruan langsung pada setiap ketukan tombol --}}
<livewire:todo-input wire:model.live="todo" />

{{-- Diperbarui saat kehilangan fokus (blur) --}}
<livewire:todo-input wire:model.live.blur="todo" />

{{-- Memberikan jeda (debounce) pada pembaruan --}}
<livewire:todo-input wire:model.live.debounce.500ms="todo" />

```

---

## Contoh: Custom rich text editor

Berikut adalah contoh yang lebih kompleks dari komponen editor teks kaya (*rich-text editor*):

```php
<?php // resources/views/components/⚡rich-editor.blade.php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable]
    public $content = '';
};
?>

<div>
    <div
        x-init="
            // Inisialisasi library rich text editor Anda di sini
            editor.on('change', () => {
                $wire.content = editor.getContent()
            })
        "
    >
        </div>
</div>

```

---

## Batasan

> [!warning] Hanya satu properti modelable per komponen
> Saat ini Livewire hanya mendukung satu atribut `#[Modelable]` per komponen, sehingga hanya atribut pertama yang akan dihubungkan.

---

## Kapan harus menggunakan

Gunakan `#[Modelable]` ketika:

* Membuat komponen input yang dapat digunakan kembali (*date pickers*, *color pickers*, *rich text editors*).
* Membangun komponen formulir yang perlu bekerja dengan `wire:model`.
* Membungkus pustaka JavaScript pihak ketiga sebagai komponen Livewire.
* Membuat input kustom dengan validasi atau pemformatan khusus.

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut tentang komunikasi antara *parent-child* dan pengikatan data, lihat [dokumentasi Nesting Components](https://www.google.com/search?q=/docs/4.x/nesting%23binding-to-child-data-using-wiremodel).
