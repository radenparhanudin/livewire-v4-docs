`wire:bind` adalah direktif yang secara dinamis menghubungkan (*bind*) atribut HTML ke properti **component** atau ekspresi tertentu. Berbeda dengan menggunakan sintaks atribut milik Blade, `wire:bind` memperbarui atribut secara reaktif di sisi klien tanpa memerlukan proses *re-render* penuh dari server.

Jika Anda sudah terbiasa dengan direktif `x-bind` milik Alpine, keduanya pada dasarnya adalah hal yang sama.

## Penggunaan dasar

```blade
<input wire:model="message" wire:bind:class="message.length > 240 && 'text-red-500'">

```

Saat pengguna mengetik, `wire:bind:class` akan bereaksi terhadap panjang pesan dan menerapkan class tersebut secara instan di sisi klien.

---

## Kasus penggunaan umum

### Binding styles

```blade
<div wire:bind:style="{ 'color': textColor, 'font-size': fontSize + 'px' }">
    Styled text
</div>

```

### Binding href

```blade
<a wire:bind:href="url">Dynamic link</a>

```

### Binding disabled state

```blade
<button wire:bind:disabled="isArchived">Delete</button>

```

### Binding data attributes

```blade
<div wire:bind:data-count="count">...</div>

```

---

## Referensi

```blade
wire:bind:{attribute}="expression"

```

Ganti `{attribute}` dengan nama atribut HTML apa pun yang valid (misalnya: `class`, `style`, `href`, `disabled`, `data-*`).

Direktif ini tidak memiliki **modifiers**.
