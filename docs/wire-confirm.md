Sebelum melakukan tindakan berbahaya di Livewire, Anda mungkin ingin memberikan semacam konfirmasi visual kepada pengguna Anda.

Livewire mempermudah hal ini dengan menambahkan `wire:confirm` sebagai tambahan pada **action** apa pun (`wire:click`, `wire:submit`, dll.).

Berikut adalah contoh penambahan dialog konfirmasi pada tombol "Delete post":

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm="Are you sure you want to delete this post?"
>
    Delete post </button>

```

Ketika pengguna mengeklik "Delete post", Livewire akan memicu dialog konfirmasi (menggunakan *confirmation alert* bawaan browser). Jika pengguna menekan tombol *escape* atau mengeklik *cancel*, **action** tidak akan dijalankan. Jika mereka menekan "OK", **action** akan diselesaikan.

---

## Prompting users for input

Untuk tindakan yang lebih berbahaya seperti menghapus akun pengguna sepenuhnya, Anda mungkin ingin menyajikan *confirmation prompt* di mana mereka harus mengetikkan rangkaian karakter tertentu untuk mengonfirmasi tindakan tersebut.

Livewire menyediakan **modifier** `.prompt` yang bermanfaat. Jika diterapkan pada `wire:confirm`, ia akan meminta input dari pengguna dan hanya mengonfirmasi **action** jika input tersebut cocok (*case-sensitive*) dengan string yang disediakan (ditentukan oleh karakter "|" (pipa) di akhir nilai `wire:confirm`):

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm.prompt="Are you sure?\n\nType DELETE to confirm|DELETE"
>
    Delete account </button>

```

Ketika pengguna menekan "Delete account", **action** hanya akan dijalankan jika "DELETE" dimasukkan ke dalam *prompt*, jika tidak, **action** akan dibatalkan.

---

## See also

* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Pelajari cara kerja metode komponen.
* **[wire:click](https://www.google.com/search?q=/docs/4.x/wire-click)** — Pemicu umum untuk konfirmasi.
* **[wire:submit](https://www.google.com/search?q=/docs/4.x/wire-submit)** — Konfirmasi sebelum pengiriman formulir.

---

## Referensi

```blade
wire:confirm="message"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.prompt` | Meminta input dari pengguna; format: "message\|expected-input" |
