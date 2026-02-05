Livewire menawarkan direktif `wire:init` untuk menjalankan **action** segera setelah **component** di-*render*. Hal ini sangat membantu dalam kasus di mana Anda tidak ingin menghambat pemuatan seluruh halaman, tetapi ingin memuat beberapa data segera setelah halaman dimuat.

```blade
<div wire:init="loadPosts">
    </div>

```

**Action** `loadPosts` akan dijalankan segera setelah **component** Livewire di-*render* pada halaman.

Namun, dalam banyak kasus, [fitur lazy loading Livewire](https://www.google.com/search?q=/docs/4.x/lazy) lebih disarankan daripada menggunakan `wire:init`.

---

## Referensi

```blade
wire:init="action"

```

Direktif ini tidak memiliki **modifiers**.
