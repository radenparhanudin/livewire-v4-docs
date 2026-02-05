Direktif `@island` membuat wilayah terisolasi di dalam sebuah **component** yang diperbarui secara independen, tanpa me-*render* ulang seluruh **component**.

## Penggunaan dasar

Bungkus bagian mana pun dari template Anda dengan `@island` untuk membuat wilayah terisolasi:

```blade
<?php // resources/views/components/⚡dashboard.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Revenue;

new class extends Component {
    #[Computed]
    public function revenue()
    {
        // Kalkulasi yang berat...
        return Revenue::yearToDate();
    }
};
?>

<div>
    @island
        <div>
            Revenue: {{ $this->revenue }}

            <button type="button" wire:click="$refresh">Refresh</button>
        </div>
    @endisland

    <div>
        </div>
</div>

```

Ketika tombol "Refresh" diklik, hanya bagian **island** yang di-*render* ulang—bagian komponen lainnya tetap tidak tersentuh.

---

## Lazy loading islands

Tunda perenderan awal sebuah **island** sampai setelah halaman dimuat menggunakan parameter `lazy`:

```blade
@island(lazy: true)
    <div>
        Revenue: {{ $this->revenue }}
    </div>
@endisland

```

**Island** tersebut awalnya akan menampilkan status pemuatan (*loading state*), kemudian mengambil kontennya dalam permintaan (*request*) terpisah.

### Lazy vs Deferred

Secara default, `lazy` menunggu sampai **island** terlihat di dalam **viewport**. Gunakan `defer` untuk memuat segera setelah pemuatan halaman selesai:

```blade
{{-- Dimuat saat digulir ke area pandang (viewport) --}}
@island(lazy: true)
    @endisland

{{-- Dimuat segera setelah halaman selesai dimuat --}}
@island(defer: true)
    @endisland

```

---

## Custom loading states

Gunakan `@placeholder` untuk menyesuaikan apa yang ditampilkan selama proses pemuatan:

```blade
@island(lazy: true)
    @placeholder
        <div class="animate-pulse">
            <div class="h-32 bg-gray-200 rounded"></div>
        </div>
    @endplaceholder

    <div>
        Revenue: {{ $this->revenue }}
    </div>
@endisland

```

---

## Named islands

Berikan nama pada **island** untuk menargetkannya dari bagian lain di dalam komponen Anda:

```blade
@island(name: 'revenue')
    <div>Revenue: {{ $this->revenue }}</div>
@endisland

<button type="button" wire:click="$refresh" wire:island="revenue">
    Refresh revenue
</button>

```

Direktif `wire:island` membatasi cakupan pembaruan (*scope updates*) hanya pada **island** tertentu.

---

## Mengapa menggunakan islands?

**Islands** memberikan isolasi performa tanpa beban tambahan (*overhead*) seperti membuat **child components** terpisah, mengelola **props**, atau berurusan dengan komunikasi antar komponen.

**Gunakan islands saat:**

* Anda ingin mengisolasi komputasi yang berat.
* Anda membutuhkan wilayah pembaruan independen di dalam satu komponen.
* Anda menginginkan arsitektur yang lebih sederhana daripada **nested components**.

---

## Referensi

```blade
@island(
    ?string $name = null,
    bool $lazy = false,
    bool $defer = false,
)
    @endisland

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$name` | `?string` | `null` | Nama unik untuk menargetkan island dengan `wire:island`. |
| `$lazy` | `bool` | `false` | Menunda render sampai island terlihat di dalam viewport. |
| `$defer` | `bool` | `false` | Memuat segera setelah halaman dimuat tanpa menunggu visibilitas viewport. |
