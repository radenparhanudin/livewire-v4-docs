Livewire menyediakan direktif `wire:click` yang sederhana untuk memanggil metode **component** (disebut juga sebagai **actions**) saat pengguna mengeklik elemen tertentu pada halaman.

Sebagai contoh, perhatikan **component** `ShowInvoice` di bawah ini:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Invoice;

class ShowInvoice extends Component
{
    public Invoice $invoice;

    public function download()
    {
        return response()->download(
            $this->invoice->file_path, 'invoice.pdf'
        );
    }
}

```

Anda dapat memicu metode `download()` dari class di atas saat pengguna mengeklik tombol "Download Invoice" dengan menambahkan `wire:click="download"`:

```html
<button type="button" wire:click="download"> Download Invoice
</button>

```

---

## Passing parameters

Anda dapat meneruskan parameter ke **actions** secara langsung di dalam direktif `wire:click`:

```blade
<button wire:click="delete({{ $post->id }})">Delete</button>

```

Saat tombol diklik, metode `delete()` akan dipanggil dengan ID dari postingan tersebut.

> [!warning] Jangan mempercayai action parameters
> Parameter **action** harus diperlakukan sama seperti input **HTTP request** dan tidak boleh dipercayai begitu saja. Selalu lakukan otorisasi kepemilikan sebelum memperbarui data.

---

## Menggunakan pada link

Saat menggunakan `wire:click` pada tag `<a>`, Anda harus menambahkan `.prevent` untuk mencegah perilaku tautan default. Jika tidak, browser akan melakukan navigasi ke `href` yang disediakan.

```blade
<a href="#" wire:click.prevent="show">View Details</a>

```

---

## Mencegah re-renders

Gunakan `.renderless` untuk melewatkan proses **re-render** **component** setelah **action** selesai. Ini berguna untuk **actions** yang hanya melakukan efek samping (*side effects*) seperti pencatatan log atau analitik:

```blade
<button wire:click.renderless="trackClick">Track Event</button>

```

---

## Mempertahankan posisi scroll

Secara default, memperbarui konten dapat mengubah posisi **scroll**. Gunakan `.preserve-scroll` untuk mempertahankan posisi **scroll** saat ini:

```blade
<button wire:click.preserve-scroll="loadMore">Load More</button>

```

---

## Eksekusi paralel

Secara default, Livewire mengantrekan (*queue*) **actions** di dalam **component** yang sama. Gunakan `.async` untuk mengizinkan **actions** berjalan secara paralel:

```blade
<button wire:click.async="process">Process</button>

```

---

## Pelajari lebih dalam

Direktif `wire:click` hanyalah salah satu dari banyak **event listeners** yang tersedia di Livewire. Untuk dokumentasi lengkap mengenai kemampuannya (dan **event listeners** lainnya), kunjungi [halaman dokumentasi Livewire actions](https://www.google.com/search?q=/docs/4.x/actions).

---

## See also

* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Panduan lengkap tentang **component actions**
* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — Mengirimkan **events** dari *click handlers*
* **[wire:confirm](https://www.google.com/search?q=/docs/4.x/wire-confirm)** — Menambahkan dialog konfirmasi pada **actions**

---

## Referensi

```blade
wire:click="methodName"
wire:click="methodName(param1, param2)"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.prevent` | Mencegah perilaku default browser |
| `.stop` | Menghentikan propagasi event (**event propagation**) |
| `.self` | Hanya terpicu jika event berasal dari elemen ini sendiri |
| `.once` | Memastikan **listener** hanya dipanggil satu kali |
| `.debounce` | Melakukan **debounce** pada handler selama 250ms (gunakan `.debounce.500ms` untuk durasi kustom) |
| `.throttle` | Membatasi (**throttle**) handler minimal setiap 250ms (gunakan `.throttle.500ms` untuk kustom) |
| `.window` | Mendengarkan event pada objek `window` |
| `.document` | Mendengarkan event pada objek `document` |
| `.outside` | Hanya mendengarkan klik di luar elemen tersebut |
| `.passive` | Tidak akan memblokir performa **scroll** |
| `.capture` | Mendengarkan selama fase **capturing** |
| `.camel` | Mengubah nama event menjadi *camel case* |
| `.dot` | Mengubah nama event menjadi notasi titik (*dot notation*) |
| `.renderless` | Melewatkan **re-rendering** setelah **action** selesai |
| `.preserve-scroll` | Mempertahankan posisi **scroll** selama pembaruan |
| `.async` | Mengeksekusi **action** secara paralel alih-alih mengantre |
