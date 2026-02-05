Atribut `#[Defer]` membuat sebuah **component** dimuat segera setelah pemuatan halaman awal selesai, mencegah **component** yang lambat menghambat proses *render* halaman.

## Penggunaan dasar

Terapkan atribut `#[Defer]` pada **component** apa pun yang ingin ditunda pemuatannya:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Defer;
use Livewire\Component;
use App\Models\Transaction;

new #[Defer] class extends Component { // [tl! highlight]
    public $amount;

    public function mount()
    {
        // Query database yang lambat...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }
};
?>

<div>
    Revenue this month: {{ $amount }}
</div>

```

Dengan `#[Defer]`, **component** awalnya akan di-*render* sebagai `<div></div>` kosong, lalu dimuat segera setelah halaman selesai dimuat—tanpa menunggu elemen tersebut masuk ke dalam **viewport**.

---

## Lazy vs Defer

Livewire menyediakan dua cara untuk menunda pemuatan **component**:

* **Lazy loading (`#[Lazy]`)** – **Component** dimuat saat terlihat di dalam **viewport** (saat pengguna menggulir ke arah elemen tersebut).
* **Deferred loading (`#[Defer]`)** – **Component** dimuat segera setelah pemuatan halaman awal selesai.

Keduanya mencegah **component** yang lambat menghambat *render* halaman awal Anda, namun berbeda pada kapan **component** tersebut benar-benar dimuat.

[Image comparing lazy loading versus deferred loading triggers]

---

## Rendering placeholders

Secara default, Livewire me-*render* `<div></div>` kosong sebelum **component** dimuat. Anda dapat menyediakan **placeholder** kustom menggunakan metode `placeholder()`:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Defer;
use Livewire\Component;
use App\Models\Transaction;

new #[Defer] class extends Component {
    public $amount;

    public function mount()
    {
        $this->amount = Transaction::monthToDate()->sum('amount');
    }

    public function placeholder() // [tl! highlight:start]
    {
        return <<<'HTML'
        <div>
            <svg></svg>
        </div>
        HTML;
    } // [tl! highlight:end]
};
?>

<div>
    Revenue this month: {{ $amount }}
</div>

```

Pengguna akan melihat *loading spinner* sampai **component** dimuat sepenuhnya.

> [!warning] Samakan tipe elemen root placeholder
> Jika elemen *root* pada **placeholder** Anda adalah `<div>`, maka **component** Anda juga harus menggunakan elemen `<div>`.

---

## Bundling requests

Secara default, **deferred components** dimuat secara paralel dengan *network requests* yang independen. Untuk menggabungkan beberapa **deferred components** ke dalam satu permintaan (*request*), gunakan parameter `bundle`:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Defer;
use Livewire\Component;

new #[Defer(bundle: true)] class extends Component { // [tl! highlight]
    // ...
};

```

Sekarang, jika ada sepuluh komponen `revenue` di halaman, kesepuluhnya akan dimuat melalui satu *bundled network request* alih-alih sepuluh *requests* paralel.

---

## Pendekatan alternatif

### Menggunakan parameter defer

Alih-alih menggunakan atribut, Anda dapat menunda instansi **component** tertentu menggunakan parameter `defer`:

```blade
<livewire:revenue defer />

```

Ini berguna ketika Anda hanya ingin instansi tertentu dari sebuah **component** yang ditunda pemuatannya.

### Mengabaikan atribut (Overriding)

Jika sebuah **component** memiliki `#[Defer]` tetapi Anda ingin memuatnya segera dalam kasus tertentu, Anda dapat mengabaikannya:

```blade
<livewire:revenue :defer="false" />

```

---

## Kapan harus menggunakan

Gunakan `#[Defer]` ketika:

* **Component** mengandung operasi lambat (query database, API calls) yang akan menunda pemuatan halaman.
* **Component** selalu terlihat pada pemuatan halaman awal (jika berada di bawah lipatan/*below the fold*, gunakan `#[Lazy]`).
* Anda ingin meningkatkan persepsi performa dengan menampilkan halaman lebih cepat.

---

## Referensi

```php
#[Defer(
    bool|null $bundle = null,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$bundle` | `bool|null` | `null` | Menggabungkan beberapa deferred components ke dalam satu network request. |
