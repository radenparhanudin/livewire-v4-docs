Direktif `@placeholder` menampilkan konten kustom sementara komponen atau **islands** yang bersifat *lazy* atau *deferred* sedang dimuat.

## Penggunaan dasar dengan lazy components

Untuk *single-file* dan *multi-file components*, gunakan `@placeholder` untuk menentukan apa yang ditampilkan selama proses pemuatan:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    public $amount;

    public function mount()
    {
        // Query database yang lambat...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }
};
?>

@placeholder
    <div>
        <svg class="animate-spin h-5 w-5">...</svg>
    </div>
@endplaceholder

<div>
    Revenue this month: {{ $amount }}
</div>

```

Ketika di-*render* dengan `<livewire:revenue lazy />`, **placeholder** akan muncul sampai komponen selesai dimuat.

> [!tip] Khusus komponen berbasis view
> Direktif `@placeholder` berfungsi untuk *single-file* dan *multi-file components*. Untuk komponen berbasis kelas (*class-based components*), gunakan metode `placeholder()` sebagai gantinya.

> [!warning] Kesesuaian tipe elemen root
> **Placeholder** dan komponen harus memiliki tipe elemen *root* yang sama. Jika **placeholder** Anda menggunakan `<div>`, maka komponen Anda juga harus menggunakan `<div>`.

---

## Penggunaan dengan islands

Gunakan `@placeholder` di dalam **lazy islands** untuk menyesuaikan status pemuatan (*loading states*):

```blade
@island(lazy: true)
    @placeholder
        <div class="animate-pulse">
            <div class="h-32 bg-gray-200 rounded"></div>
        </div>
    @endplaceholder

    <div>
        Revenue: {{ $this->revenue }}

        <button type="button" wire:click="$refresh">Refresh</button>
    </div>
@endisland

```

**Placeholder** muncul saat **island** sedang dimuat, lalu digantikan dengan konten yang sebenarnya.

---

## Skeleton placeholders

**Placeholders** sangat ideal untuk membuat *skeleton loaders* yang sesuai dengan tata letak konten Anda:

```blade
@placeholder
    <div class="space-y-4">
        <div class="h-4 bg-gray-200 rounded w-3/4"></div>
        <div class="h-4 bg-gray-200 rounded"></div>
        <div class="h-4 bg-gray-200 rounded w-5/6"></div>
    </div>
@endplaceholder

<div>
    <h2>{{ $post->title }}</h2>
    <p>{{ $post->content }}</p>
</div>

```

---

## Pelajari lebih lanjut

Untuk pemuatan komponen secara *lazy*, lihat [dokumentasi Lazy Loading](https://www.google.com/search?q=/docs/4.x/lazy).

Untuk status pemuatan pada **island**, lihat [dokumentasi Islands](https://www.google.com/search?q=/docs/4.x/islands).
