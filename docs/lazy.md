Livewire memungkinkan Anda untuk melakukan **lazy load** pada **components** yang jika tidak dilakukan akan memperlambat pemuatan halaman awal.

## Lazy vs Defer

Livewire menyediakan dua cara untuk menunda pemuatan **component**:

* **Lazy loading (`lazy`)**: **Components** dimuat saat mereka terlihat di dalam **viewport** (ketika pengguna menggulir ke arah mereka).
* **Deferred loading (`defer`)**: **Components** dimuat segera setelah pemuatan halaman awal selesai.

Kedua pendekatan ini mencegah **components** yang lambat menghalangi **initial page render** Anda, tetapi berbeda dalam hal kapan **component** tersebut benar-benar dimuat.

## Basic example

Sebagai contoh, bayangkan Anda memiliki **component** `revenue` yang berisi **database query** yang lambat di dalam `mount()`:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    public $amount;

    public function mount()
    {
        // Database query yang lambat...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }
};
?>

<div>
    Revenue this month: {{ $amount }}
</div>

```

Tanpa **lazy loading**, **component** ini akan menunda pemuatan seluruh halaman dan membuat seluruh aplikasi Anda terasa lambat.

Untuk mengaktifkan **lazy loading**, Anda dapat meneruskan **parameter** `lazy` ke dalam **component**:

```blade
<livewire:revenue lazy />

```

Sekarang, alih-alih memuat **component** secara langsung, Livewire akan melewati **component** ini, memuat halaman tanpanya. Kemudian, ketika **component** tersebut terlihat di dalam **viewport**, Livewire akan melakukan **network request** untuk memuat **component** ini sepenuhnya pada halaman.

> [!info] Permintaan lazy dan deferred diisolasi secara default
> Berbeda dengan **network requests** lainnya di Livewire, pembaruan **component** yang bersifat **lazy** dan **deferred** diisolasi satu sama lain saat dikirim ke server. Ini menjaga pemuatan tetap cepat dengan memuat setiap **component** secara paralel. [Baca lebih lanjut tentang bundling components →](https://www.google.com/search?q=%23bundling-multiple-lazy-components)

## Rendering placeholder HTML

Secara **default**, Livewire akan menyisipkan sebuah `<div></div>` kosong untuk **component** Anda sebelum ia dimuat sepenuhnya. Karena **component** tersebut awalnya tidak akan terlihat oleh pengguna, hal ini bisa terasa janggal saat **component** tiba-tiba muncul di halaman.

Untuk memberi sinyal kepada pengguna bahwa **component** sedang dimuat, Anda dapat merender **placeholder HTML** seperti **loading spinners** dan **skeleton placeholders**.

### Menggunakan directive @placeholder

Untuk **single-file** dan **multi-file components**, Anda dapat menggunakan **directive** `@placeholder` secara langsung di dalam **view** Anda untuk menentukan konten **placeholder**:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    public $amount;

    public function mount()
    {
        // Database query yang lambat...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }
};
?>

@placeholder
    <div>
        <svg>...</svg>
    </div>
@endplaceholder

<div>
    Revenue this month: {{ $amount }}
</div>

```

Konten di dalam `@placeholder` dan `@endplaceholder` akan ditampilkan saat **component** sedang dimuat, lalu diganti dengan konten **component** yang sebenarnya setelah dimuat.

> [!tip] Directive placeholder hanya untuk view-based components
> **Directive** `@placeholder` hanya tersedia untuk **view-based components** (**single-file** dan **multi-file components**). Untuk **class-based components**, gunakan **method** `placeholder()` sebagai gantinya.

> [!warning] Placeholder dan component harus berbagi tipe elemen yang sama
> Misalnya, jika tipe **root element** dari **placeholder** Anda adalah 'div', maka **component** Anda juga harus menggunakan elemen 'div'.

### Menggunakan method placeholder()

Untuk **class-based components**, atau jika Anda lebih suka kontrol secara programatik, Anda dapat mendefinisikan **method** `placeholder()` yang mengembalikan HTML:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Transaction;

class Revenue extends Component
{
    public $amount;

    public function mount()
    {
        // Database query yang lambat...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }

    public function placeholder()
    {
        return <<<'HTML'
        <div>
            <svg>...</svg>
        </div>
        HTML;
    }

    public function render()
    {
        return view('livewire.revenue');
    }
}

```

Untuk pemuat yang lebih kompleks (seperti **skeletons**), Anda dapat mengembalikan sebuah `view` dari **method** `placeholder()`:

```php
public function placeholder(array $params = [])
{
    return view('livewire.placeholders.skeleton', $params);
}

```

Setiap **parameters** dari **component** yang sedang di-**lazy load** akan tersedia sebagai argumen `$params` yang diteruskan ke **method** `placeholder()`.

## Loading immediately after page load

Secara **default**, **lazy-loaded components** tidak akan dimuat sepenuhnya sampai mereka masuk ke dalam **viewport** browser, misalnya saat pengguna menggulir ke arahnya.

Jika Anda lebih suka memuat **components** segera setelah halaman dimuat, tanpa menunggu mereka masuk ke dalam **viewport**, Anda dapat menggunakan **parameter** `defer` sebagai gantinya:

```blade
<livewire:revenue defer />

```

Sekarang **component** ini akan dimuat segera setelah halaman siap tanpa menunggu ia terlihat di dalam **viewport**.

Anda juga dapat menggunakan **attribute** `#[Defer]` untuk membuat sebuah **component** menjadi **defer-loaded** secara **default**:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Defer;

#[Defer]
class Revenue extends Component
{
    // ...
}

```

> [!tip] Sintaks legacy on-load
> Anda juga dapat menggunakan `lazy="on-load"` yang berperilaku sama dengan `defer`. **Parameter** `defer` direkomendasikan untuk kode baru.

## Passing in props

Secara umum, Anda dapat memperlakukan **lazy components** sama seperti **components** normal, karena Anda masih dapat meneruskan data ke dalamnya dari luar.

Sebagai contoh, berikut adalah skenario di mana Anda mungkin meneruskan interval waktu ke dalam **component** `Revenue` dari sebuah **parent component**:

```blade
<input type="date" wire:model="start">
<input type="date" wire:model="end">

<livewire:revenue lazy :$start :$end />

```

Anda dapat menerima data ini di dalam `mount()` sama seperti **component** lainnya:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    public $amount;

    public function mount($start, $end)
    {
        // Database query yang mahal...
        $this->amount = Transactions::between($start, $end)->sum('amount');
    }
};
?>

@placeholder
    <div>
        <svg>...</svg>
    </div>
@endplaceholder

<div>
    Revenue this month: {{ $amount }}
</div>

```

Namun, berbeda dengan pemuatan **component** normal, sebuah **lazy component** harus melakukan **serialize** atau "**dehydrate**" pada setiap **properties** yang diteruskan dan menyimpannya sementara di sisi **client-side** sampai **component** tersebut dimuat sepenuhnya.

Sebagai contoh, Anda mungkin ingin meneruskan sebuah **Eloquent model** ke dalam **component** `revenue` seperti ini:

```blade
<livewire:revenue lazy :$user />

```

Dalam **component** normal, model PHP `$user` asli yang ada di memori akan diteruskan ke **method** `mount()` dari `revenue`. Namun, karena kita tidak akan menjalankan `mount()` sampai **network request** berikutnya, Livewire secara internal akan mengubah `$user` menjadi JSON (**serialize**) dan kemudian melakukan **re-query** dari database sebelum permintaan berikutnya ditangani.

Biasanya, **serialization** ini tidak akan menyebabkan perbedaan perilaku apa pun dalam aplikasi Anda.

## Enforcing lazy atau defer by default

Jika Anda ingin memastikan bahwa semua penggunaan sebuah **component** akan di-**lazy-load** atau di-**deferred**, Anda dapat menambahkan **attribute** `#[Lazy]` atau `#[Defer]` di atas **class component**:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Lazy;

#[Lazy]
class Revenue extends Component
{
    // ...
}

```

Atau untuk **deferred loading**:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Defer;

#[Defer]
class Revenue extends Component
{
    // ...
}

```

Anda dapat menimpa (**override**) pengaturan **default** ini saat merender sebuah **component**:

```blade
{{-- Menonaktifkan lazy loading --}}
<livewire:revenue :lazy="false" />

{{-- Menonaktifkan deferred loading --}}
<livewire:revenue :defer="false" />

```

## Bundling multiple lazy components

Secara **default**, jika terdapat beberapa **lazy-loaded components** pada halaman, setiap **component** akan membuat **network request** independen secara paralel. Hal ini sering kali diinginkan untuk performa karena setiap **component** dimuat secara mandiri.

Namun, jika Anda memiliki banyak **lazy components** pada satu halaman, Anda mungkin ingin menyatukan mereka (**bundle**) ke dalam satu **network request** untuk mengurangi beban server (**overhead**).

### Menggunakan parameter bundle

Anda dapat mengaktifkan **bundling** menggunakan **parameter** `bundle: true`:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Lazy;

#[Lazy(bundle: true)]
class Revenue extends Component
{
    // ...
}

```

Sekarang, jika terdapat sepuluh **components** `Revenue` pada halaman yang sama, saat halaman dimuat, kesepuluh pembaruan tersebut akan disatukan dan dikirim ke server sebagai satu **network request** tunggal.

### Menggunakan modifier bundle

Anda juga dapat mengaktifkan **bundling** secara **inline** saat merender **component** menggunakan **modifier** `bundle`:

```blade
<livewire:revenue lazy.bundle />

```

Ini juga berlaku untuk **deferred components**:

```blade
<livewire:revenue defer.bundle />

```

Atau menggunakan **attribute**:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Defer;

#[Defer(bundle: true)]
class Revenue extends Component
{
    // ...
}

```

### Kapan menggunakan bundling

**Gunakan bundling saat:**

* Anda memiliki banyak (5+) **lazy** atau **deferred components** pada satu halaman.
* **Components** tersebut memiliki kompleksitas dan waktu muat yang serupa.
* Anda ingin mengurangi beban server dan jumlah koneksi HTTP.

**Jangan gunakan bundling saat:**

* **Components** memiliki waktu muat yang sangat berbeda (**component** yang lambat akan menghambat yang cepat).
* Anda ingin **components** muncul segera setelah masing-masing siap.
* Anda hanya memiliki sedikit **lazy components** pada halaman.

> [!tip] Sintaks legacy isolate
> Anda juga dapat menggunakan `isolate: false` yang berperilaku sama dengan `bundle: true`. **Parameter** `bundle` direkomendasikan untuk kode baru karena lebih eksplisit mengenai tujuannya.

## Full-page lazy loading

Anda dapat melakukan **lazy load** atau **defer** pada **full-page Livewire components** menggunakan **route methods**.

### Lazy loading halaman penuh

Gunakan `->lazy()` untuk memuat **component** saat ia memasuki **viewport**:

```php
Route::livewire('/dashboard', 'pages::dashboard')->lazy();

```

### Deferring halaman penuh

Gunakan `->defer()` untuk memuat **component** segera setelah halaman dimuat:

```php
Route::livewire('/dashboard', 'pages::dashboard')->defer();

```

### Menonaktifkan lazy/defer loading

Jika sebuah **component** diatur **lazy** atau **defer** secara **default** (melalui **attribute** `#[Lazy]` atau `#[Defer]`), Anda dapat membatalkannya menggunakan `enabled: false`:

```php
Route::livewire('/dashboard', 'pages::dashboard')->lazy(enabled: false);
Route::livewire('/dashboard', 'pages::dashboard')->defer(enabled: false);

```

## Default placeholder view

Jika Anda ingin mengatur **default placeholder view** untuk semua **components** Anda, Anda dapat melakukannya dengan mereferensikan **view** tersebut di dalam file konfigurasi `/config/livewire.php`:

```php
'component_placeholder' => 'livewire.placeholder',

```

Sekarang, ketika sebuah **component** di-**lazy-load** dan tidak ada `placeholder()` yang didefinisikan, Livewire akan menggunakan **Blade view** yang telah dikonfigurasi (dalam hal ini `livewire.placeholder`).

## Menonaktifkan lazy loading untuk tests

Saat melakukan **unit testing** pada **lazy component**, atau halaman dengan **nested lazy components**, Anda mungkin ingin menonaktifkan perilaku "lazy" agar Anda dapat memastikan perilaku render akhirnya. Jika tidak, **components** tersebut akan dirender sebagai **placeholders** selama pengujian Anda.

Anda dapat dengan mudah menonaktifkan **lazy loading** menggunakan **testing helper** `Livewire::withoutLazyLoading()` seperti berikut:

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\Dashboard;
use Livewire\Livewire;
use Tests\TestCase;

class DashboardTest extends TestCase
{
    public function test_renders_successfully()
    {
        Livewire::withoutLazyLoading() // [tl! highlight]
            ->test(Dashboard::class)
            ->assertSee(...);
    }
}

```

Sekarang, ketika **dashboard component** dirender untuk pengujian ini, ia akan melewati proses render `placeholder()` dan sebaliknya merender **component** secara penuh seolah-olah **lazy loading** tidak diterapkan sama sekali.

---

## See also

* **[Islands](https://www.google.com/search?q=/docs/4.x/islands)** — Mengisolasi pembaruan di dalam satu **component**.
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Menampilkan **placeholders** saat **components** dimuat.
* **[@placeholder](https://www.google.com/search?q=/docs/4.x/directive-placeholder)** — Mendefinisikan konten **placeholder**.
* **[Lazy Attribute](https://www.google.com/search?q=/docs/4.x/attribute-lazy)** — Menandai **components** untuk **lazy loading**.
