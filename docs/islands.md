Islands memungkinkan Anda membuat wilayah terisolasi di dalam sebuah **Livewire component** yang memperbarui dirinya secara independen. Saat sebuah **action** terjadi di dalam sebuah **island**, hanya **island** tersebut yang melakukan **re-render** — bukan seluruh **component**.

Ini memberikan keuntungan performa seperti memecah **components** menjadi bagian-bagian kecil tanpa beban tambahan saat membuat **child components** terpisah, mengelola **props**, atau berurusan dengan komunikasi antar **component**.

## Basic usage

Untuk membuat sebuah **island**, bungkus bagian mana pun dari **Blade template** Anda dengan **directive** `@island`:

```blade
<?php // resources/views/components/⚡dashboard.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Revenue;

new class extends Component {
    #[Computed]
    public function revenue()
    {
        // Kalkulasi atau query yang berat...
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

Saat tombol "Refresh" diklik, hanya **island** yang berisi kalkulasi pendapatan yang akan melakukan **re-render**. Sisa dari **component** tersebut tetap tidak tersentuh.

Karena kalkulasi berat berada di dalam sebuah **computed property**—yang dievaluasi sesuai permintaan (**on-demand**)—ia hanya akan dipanggil saat **island** melakukan **re-render**, bukan saat bagian lain dari halaman diperbarui. Namun, karena **islands** dimuat bersama halaman secara **default**, **property** `revenue` akan tetap dikalkulasi selama pemuatan halaman awal.

## Lazy loading

Terkadang Anda memiliki komputasi berat atau pemanggilan API yang lambat yang seharusnya tidak menghalangi pemuatan halaman awal Anda. Anda dapat menunda **initial render** sebuah **island** sampai setelah halaman dimuat menggunakan **parameter** `lazy`:

```blade
@island(lazy: true)
    <div>
        Revenue: {{ $this->revenue }}

        <button type="button" wire:click="$refresh">Refresh</button>
    </div>
@endisland

```

**Island** akan menampilkan **loading state** pada awalnya, kemudian mengambil dan merender kontennya dalam sebuah permintaan terpisah.

### Lazy vs Deferred loading

Secara **default**, `lazy` menggunakan **intersection observer** untuk memicu pemuatan saat **island** terlihat di **viewport**. Jika Anda ingin **island** dimuat segera setelah halaman dimuat (terlepas dari visibilitasnya), gunakan `defer`:

```blade
{{-- Dimuat saat di-scroll ke area pandang --}}
@island(lazy: true)
    @endisland

{{-- Dimuat segera setelah halaman dimuat --}}
@island(defer: true)
    @endisland

```

### Custom loading states

Anda dapat menyesuaikan apa yang ditampilkan saat **lazy island** sedang dimuat menggunakan **directive** `@placeholder`:

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

## Named islands

Untuk memicu sebuah **island** dari tempat lain di dalam **component** Anda, berikan sebuah nama dan referensikan menggunakan `wire:island`:

```blade
<div>
    @island(name: 'revenue')
        <div>
            Revenue: {{ $this->revenue }}
        </div>
    @endisland

    <button type="button" wire:click="$refresh" wire:island="revenue">
        Refresh revenue
    </button>
</div>

```

**Directive** `wire:island` bekerja bersama **action directives** seperti `wire:click`, `wire:submit`, dll. untuk membatasi pembaruan mereka ke **island** tertentu.

Ketika Anda memiliki beberapa **islands** dengan nama yang sama, mereka akan terhubung satu sama lain dan akan selalu dirender sebagai sebuah grup:

```blade
@island(name: 'revenue')
    <div class="sidebar">
        Revenue: {{ $this->revenue }}
    </div>
@endisland

@island(name: 'revenue')
    <div class="header">
        Revenue: {{ $this->revenue }}
    </div>
@endisland

<button type="button" wire:click="$refresh" wire:island="revenue">
    Refresh revenue
</button>

```

Kedua **islands** akan diperbarui bersamaan kapan pun salah satunya dipicu.

## Append and prepend modes

Alih-alih mengganti konten sepenuhnya, **islands** dapat menambahkan konten baru di akhir (**append**) atau di awal (**prepend**). Ini sangat cocok untuk **pagination**, **infinite scroll**, atau **real-time feeds**:

```blade
<div>
    @island(name: 'feed')
        @foreach ($this->activities as $activity)
            <x-activity-item wire:key="{{ $activity->id }}" :activity="$activity" />
        @endforeach
    @endisland

    <button type="button" wire:click="loadMore" wire:island.append="feed">
        Load more
    </button>
</div>

```

Mode yang tersedia:

* `wire:island.append` - Tambahkan ke akhir
* `wire:island.prepend` - Tambahkan ke awal

## Nested islands

**Islands** dapat bersarang (**nested**) satu sama lain. Saat **island** luar melakukan **re-render**, **islands** di dalamnya dilewati secara **default**:

```blade
@island(name: 'revenue')
    <div>
        Total revenue: {{ $this->revenue }}

        @island(name: 'breakdown')
            <div>
                Monthly breakdown: {{ $this->monthlyBreakdown }}
                <button type="button" wire:click="$refresh">Refresh breakdown</button>
            </div>
        @endisland

        <button type="button" wire:click="$refresh">Refresh revenue</button>
    </div>
@endisland

```

Mengklik "Refresh revenue" hanya memperbarui **island** luar, sementara "Refresh breakdown" hanya memperbarui **island** dalam.

## Always render with parent

Secara **default**, saat sebuah **component** melakukan **re-render**, **islands** akan dilewati. Gunakan **parameter** `always` untuk memaksa sebuah **island** diperbarui kapan pun **parent component** diperbarui:

```blade
@island(always: true)
    <div>
        Revenue: {{ $this->revenue }}
    </div>
@endisland

```

Dengan `always: true`, **island** akan melakukan **re-render** kapan pun bagian mana pun dari **component** diperbarui. Ini berguna untuk data krusial yang harus selalu selaras dengan **component state**.

## Skip initial render

**Parameter** `skip` mencegah sebuah **island** dirender pada awalnya, sangat cocok untuk konten **on-demand**:

```blade
@island(skip: true)
    @placeholder
        <button type="button" wire:click="$refresh">Load revenue details</button>
    @endplaceholder

    <div>
        Revenue: {{ $this->revenue }}
    </div>
@endisland

```

Konten **placeholder** akan ditampilkan pada awalnya. Saat dipicu, **island** dirender dan menggantikan **placeholder**.

## Island polling

Anda dapat menggunakan `wire:poll` di dalam sebuah **island** untuk menyegarkan hanya **island** tersebut pada interval tertentu:

```blade
@island
    <div wire:poll.3s>
        Revenue: {{ $this->revenue }}
    </div>
@endisland

```

**Polling** ini dibatasi pada **island** tersebut — hanya **island** tersebut yang akan menyegarkan setiap 3 detik, bukan seluruh **component**.

## Triggering islands from JavaScript

**Directive** `wire:island` hanya bekerja bersama **Livewire action directives** seperti `wire:click`. Untuk membatasi sebuah **action** ke sebuah **island** dari Alpine atau JavaScript, gunakan `$wire.$island()`:

```blade
<button type="button" x-on:click="$wire.$island('feed').loadMore()">
    Load more
</button>

```

Ini setara dengan `wire:click="loadMore" wire:island="feed"`. Mode **append** dan **prepend** juga didukung melalui parameter opsi:

```blade
<button type="button" x-on:click="$wire.$island('feed', { mode: 'append' }).loadMore()">
    Load more
</button>

```

Setiap **method** `$wire` bekerja dengan `$island()`, termasuk `$refresh()`, `$set()`, dan `$toggle()`.

## Considerations

Meskipun **islands** menyediakan isolasi yang kuat, harap diingat:

**Data scope**: **Islands** memiliki akses ke **properties** dan **methods** milik **component**, tetapi tidak ke **template variables** yang didefinisikan di luar **island**. Variabel `@php` atau variabel **loop** dari **parent template** tidak akan tersedia di dalam **island**.

**Islands tidak dapat digunakan dalam loops atau conditionals**: Karena **islands** tidak memiliki akses ke **loop variables** atau konteks kondisional, mereka tidak dapat digunakan di dalam `@foreach`, `@if`, atau struktur kontrol lainnya. Sebaliknya, letakkan **loop** atau kondisional tersebut di dalam **island**.

**State synchronization**: Meskipun permintaan **island** berjalan secara paralel, baik **islands** maupun **root component** dapat mengubah **state component** yang sama. Jika beberapa permintaan berjalan secara bersamaan, respons terakhir yang kembali akan memenangkan pertarungan **state**.

**Kapan menggunakan islands**: **Islands** paling bermanfaat untuk:

* Komputasi berat yang tidak boleh menghalangi pemuatan halaman awal.
* Wilayah independen dengan interaksi mereka sendiri.
* Pembaruan **real-time** yang hanya memengaruhi sebagian UI.
* Hambatan performa (**performance bottlenecks**) pada **components** yang besar.

---

## See also

* **[Nesting](https://www.google.com/search?q=/docs/4.x/nesting)** — Pendekatan alternatif menggunakan *child components*.
* **[Lazy Loading](https://www.google.com/search?q=/docs/4.x/lazy)** — Menunda pemuatan konten yang berat.
* **[Computed Properties](https://www.google.com/search?q=/docs/4.x/computed-properties)** — Optimalkan performa *island* dengan *memoization*.
* **[@island](https://www.google.com/search?q=/docs/4.x/directive-island)** — Dokumentasi direktif untuk membuat wilayah pembaruan terisolasi.
