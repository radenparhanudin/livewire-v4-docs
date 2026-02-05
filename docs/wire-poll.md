Polling adalah teknik yang digunakan dalam aplikasi web untuk melakukan "jajak pendapat" ke server (mengirim *request* dalam interval reguler) guna mendapatkan pembaruan. Ini adalah cara sederhana untuk menjaga halaman tetap mutakhir tanpa memerlukan teknologi yang lebih canggih seperti [WebSockets](https://www.google.com/search?q=/docs/4.x/events%23real-time-events-using-laravel-echo).

## Penggunaan dasar

Menggunakan polling di dalam Livewire semudah menambahkan `wire:poll` ke sebuah elemen.

Di bawah ini adalah contoh komponen `SubscriberCount` yang menampilkan jumlah pelanggan (*subscriber*) pengguna:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class SubscriberCount extends Component
{
    public function render()
    {
        return view('livewire.subscriber-count', [
            'count' => Auth::user()->subscribers->count(),
        ]);
    }
}

```

```blade
<div wire:poll> Subscribers: {{ $count }}
</div>

```

Biasanya, komponen ini akan menampilkan jumlah pelanggan dan tidak akan pernah diperbarui sampai halaman dimuat ulang. Namun, karena adanya `wire:poll` pada *template* komponen, komponen ini sekarang akan menyegarkan dirinya sendiri setiap `2.5` detik, menjaga jumlah pelanggan tetap mutakhir.

Anda juga dapat menentukan sebuah **action** untuk dijalankan pada interval polling dengan memberikan nilai ke `wire:poll`:

```blade
<div wire:poll="refreshSubscribers">
    Subscribers: {{ $count }}
</div>

```

Sekarang, metode `refreshSubscribers()` pada komponen akan dipanggil setiap `2.5` detik.

---

## Kontrol waktu (Timing control)

Kelemahan utama dari polling adalah penggunaan sumber daya yang intensif. Jika Anda memiliki seribu pengunjung pada halaman yang menggunakan polling, seribu *network requests* akan dipicu setiap `2.5` detik.

Cara terbaik untuk mengurangi *requests* dalam skenario ini adalah dengan memperpanjang interval polling.

Anda dapat mengontrol secara manual seberapa sering komponen akan melakukan polling dengan menambahkan durasi yang diinginkan ke `wire:poll` seperti berikut:

```blade
<div wire:poll.15s> <div wire:poll.15000ms> ```

---

## Background throttling

Untuk lebih menekan jumlah *request* ke server, Livewire secara otomatis melakukan **throttling** (pembatasan) polling saat halaman berada di latar belakang (*background*). Sebagai contoh, jika pengguna membiarkan halaman terbuka di tab browser yang berbeda, Livewire akan mengurangi jumlah polling *requests* sebesar 95% sampai pengguna kembali mengunjungi tab tersebut.

Jika Anda ingin menonaktifkan perilaku ini dan terus melakukan polling secara terus-menerus, bahkan saat tab berada di latar belakang, Anda dapat menambahkan **modifier** `.keep-alive` ke `wire:poll`:

```blade
<div wire:poll.keep-alive>

```

---

## Viewport throttling

Langkah lain yang dapat Anda ambil agar polling hanya dilakukan saat diperlukan adalah dengan menambahkan **modifier** `.visible` ke `wire:poll`. Modifier `.visible` menginstruksikan Livewire untuk hanya melakukan polling pada komponen saat komponen tersebut terlihat di halaman:

```blade
<div wire:poll.visible>

```

Jika komponen yang menggunakan `wire:visible` berada di bagian bawah halaman yang panjang, ia tidak akan mulai melakukan polling sampai pengguna menggulirnya ke dalam **viewport**. Ketika pengguna menggulir menjauh, polling akan berhenti kembali.

---

## Referensi

```blade
wire:poll
wire:poll="action"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.[number]s` | Interval polling dalam detik (misal: `.15s`) |
| `.[number]ms` | Interval polling dalam milidetik (misal: `.15000ms`) |
| `.keep-alive` | Terus melakukan polling meskipun tab berada di latar belakang |
| `.visible` | Hanya melakukan polling saat elemen terlihat di dalam **viewport** |
