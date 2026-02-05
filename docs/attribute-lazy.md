Atribut `#[Lazy]` membuat sebuah **component** hanya dimuat ketika ia menjadi terlihat di dalam **viewport**, mencegah **component** yang lambat menghambat proses *render* halaman awal.

## Penggunaan dasar

Terapkan atribut `#[Lazy]` pada **component** apa pun yang ingin dimuat secara *lazy-load*:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Lazy;
use Livewire\Component;
use App\Models\Transaction;

new #[Lazy] class extends Component { // [tl! highlight]
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

Dengan `#[Lazy]`, **component** awalnya akan di-*render* sebagai `<div></div>` kosong, lalu dimuat saat ia masuk ke dalam **viewport**—biasanya saat pengguna menggulir halaman ke arahnya.

---

## Lazy vs Defer

Livewire menyediakan dua cara untuk menunda pemuatan **component**:

* **Lazy loading (`#[Lazy]`)** – **Component** dimuat saat terlihat di dalam **viewport** (saat pengguna menggulir ke arahnya).
* **Deferred loading (`#[Defer]`)** – **Component** dimuat segera setelah pemuatan halaman awal selesai.

Gunakan *lazy loading* untuk **component** yang berada di bawah lipatan (*below the fold*) di mana pengguna mungkin tidak akan menggulir ke sana. Gunakan *defer* untuk **component** yang selalu terlihat tetapi Anda ingin memuatnya setelah halaman selesai di-*render*.

[Image comparing lazy vs defer loading triggers in a web page]

---

## Rendering placeholders

Secara default, Livewire me-*render* `<div></div>` kosong sebelum **component** dimuat. Anda dapat menyediakan **placeholder** kustom menggunakan metode `placeholder()`:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Lazy;
use Livewire\Component;
use App\Models\Transaction;

new #[Lazy] class extends Component {
    public $amount;

    public function mount()
    {
        $this->amount = Transaction::monthToDate()->sum('amount');
    }

    public function placeholder() // [tl! highlight:start]
    {
        return <<<'HTML'
        <div>
            <div class="animate-pulse bg-gray-200 h-20 rounded"></div>
        </div>
        HTML;
    } // [tl! highlight:end]
};
?>

<div>
    Revenue this month: {{ $amount }}
</div>

```

Pengguna akan melihat **placeholder** berupa *skeleton* sampai **component** masuk ke dalam **viewport** dan dimuat.

> [!warning] Samakan tipe elemen root placeholder
> Jika elemen *root* pada **placeholder** Anda adalah `<div>`, maka **component** Anda juga harus menggunakan elemen `<div>`.

---

## Bundling requests

Secara default, **lazy components** dimuat secara paralel dengan *network requests* yang independen. Untuk menggabungkan beberapa **lazy components** ke dalam satu permintaan (*request*), gunakan parameter `bundle`:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Lazy;
use Livewire\Component;

new #[Lazy(bundle: true)] class extends Component { // [tl! highlight]
    // ...
};

```

Sekarang, jika ada sepuluh komponen `revenue` di halaman, kesepuluhnya akan dimuat melalui satu *bundled network request* alih-alih sepuluh *requests* paralel.

---

## Pendekatan alternatif

### Menggunakan parameter lazy

Alih-alih menggunakan atribut, Anda dapat melakukan *lazy-load* pada instansi **component** tertentu menggunakan parameter `lazy`:

```blade
<livewire:revenue lazy />

```

Ini berguna ketika Anda hanya ingin instansi tertentu dari sebuah **component** yang dimuat secara *lazy-load*.

### Mengabaikan atribut (Overriding)

Jika sebuah **component** memiliki `#[Lazy]` tetapi Anda ingin memuatnya segera dalam kasus tertentu, Anda dapat mengabaikannya:

```blade
<livewire:revenue :lazy="false" />

```

---

## Kapan harus menggunakan

Gunakan `#[Lazy]` ketika:

* **Component** mengandung operasi lambat (query database, API calls) yang akan menunda pemuatan halaman.
* **Component** berada di bawah lipatan (*below the fold*) dan pengguna mungkin tidak akan menggulir ke sana.
* Anda ingin meningkatkan persepsi performa dengan menampilkan halaman lebih cepat.
* Anda memiliki banyak **component** yang memakan sumber daya besar dalam satu halaman.

---

## Referensi

```php
#[Lazy(
    bool|null $bundle = null,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$bundle` | `bool|null` | `null` | Menggabungkan beberapa lazy components ke dalam satu network request. |
