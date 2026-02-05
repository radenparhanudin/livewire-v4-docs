Atribut `#[Async]` memungkinkan **actions** berjalan secara paralel tanpa harus mengantre, membuatnya dieksekusi segera bahkan jika ada *requests* lain yang sedang berjalan (*in-flight*).

## Penggunaan dasar

Terapkan atribut `#[Async]` pada metode **action** apa pun yang harus dijalankan secara paralel:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Async;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[Async] // [tl! highlight]
    public function logActivity()
    {
        Activity::log('post-viewed', $this->post);
    }
};

```

```blade
<div wire:intersect="logActivity">
    </div>

```

Ketika `logActivity()` dipanggil, ia akan langsung dieksekusi tanpa memblokir *requests* lain atau terblokir oleh mereka.

---

## Kapan harus menggunakan

Gunakan `#[Async]` untuk operasi *fire-and-forget* (jalankan lalu lupakan) di mana hasilnya tidak memengaruhi apa yang ditampilkan di halaman:

* **Analitik dan Logging** – Melacak perilaku pengguna, tampilan halaman, atau interaksi.
* **Operasi Latar Belakang** – Memicu *jobs*, mengirim notifikasi, atau memperbarui layanan eksternal.
* **Hasil khusus JavaScript** – Mengambil data via `await $wire.getData()` yang akan dikonsumsi murni oleh JavaScript.

Berikut adalah contoh pelacakan klik tautan eksternal:

```php
<?php // resources/views/components/⚡external-link.blade.php

use Livewire\Attributes\Async;
use Livewire\Component;

new class extends Component {
    public $url;

    #[Async] // [tl! highlight]
    public function trackClick()
    {
        Analytics::track('external-link-clicked', [
            'url' => $this->url,
            'user_id' => auth()->id(),
        ]);
    }
};

```

```blade
<a href="{{ $url }}" target="_blank" wire:click="trackClick">
    Visit External Site
</a>

```

Karena pelacakan terjadi secara asinkron, klik pengguna tidak akan terhambat oleh *network request* tersebut.

---

## Kapan TIDAK boleh menggunakan

> [!warning] Async actions dan mutasi state tidak bisa digabung
> **Jangan pernah menggunakan async actions jika mereka memodifikasi state component yang tercermin di UI Anda.** Karena async actions berjalan secara paralel, Anda bisa berakhir dengan kondisi *race condition* yang tidak terduga di mana *state component* Anda berbeda di antara beberapa *requests* yang berjalan bersamaan.

Perhatikan contoh berbahaya ini:

```php
// Peringatan: Cuplikan ini menunjukkan hal yang TIDAK boleh dilakukan...

<?php // resources/views/components/⚡counter.blade.php

use Livewire\Attributes\Async;
use Livewire\Component;

new class extends Component {
    public $count = 0;

    #[Async] // Jangan lakukan ini! [tl! highlight]
    public function increment()
    {
        $this->count++; // Mutasi state di dalam async action [tl! highlight]
    }
};

```

Jika pengguna mengeklik tombol *increment* dengan cepat, beberapa *async requests* akan terpicu secara bersamaan. Setiap *request* dimulai dengan nilai `$count` awal yang sama, menyebabkan pembaruan data hilang (*lost updates*). Anda mungkin mengeklik 5 kali tetapi hanya melihat penghitung bertambah 1.

**Aturan praktisnya:** Hanya gunakan *async* untuk **actions** yang murni melakukan efek samping (*side effects*)—operasi yang tidak mengubah properti apa pun yang memengaruhi tampilan *view component* Anda.

---

## Pendekatan alternatif

### Menggunakan modifier .async

Alih-alih menggunakan atribut, Anda dapat membuat panggilan **action** tertentu menjadi asinkron dengan **modifier** `.async`:

```blade
<button wire:click.async="logActivity">Track Event</button>

```

Pendekatan ini berguna ketika Anda ingin sebuah **action** berjalan asinkron di beberapa tempat tetapi sinkron di tempat lainnya.

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut tentang *async actions*, *race conditions*, dan kasus penggunaan tingkat lanjut, lihat [dokumentasi Actions](https://www.google.com/search?q=/docs/4.x/actions%23parallel-execution-with-async).
