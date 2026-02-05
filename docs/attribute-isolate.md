Atribut `#[Isolate]` mencegah *request* sebuah **component** digabungkan (*bundled*) dengan pembaruan **component** lainnya, sehingga memungkinkannya untuk dieksekusi secara paralel.

## Mengapa bundling itu penting

Setiap pembaruan **component** di Livewire memicu sebuah *network request*. Secara default, ketika beberapa **component** memicu pembaruan pada saat yang bersamaan, mereka akan digabungkan menjadi satu *request* tunggal.

Hal ini menghasilkan lebih sedikit koneksi jaringan ke server dan dapat secara drastis mengurangi beban server. Selain keuntungan performa, hal ini juga mengaktifkan fitur internal yang membutuhkan kolaborasi antar beberapa **component** ([Reactive Properties](https://www.google.com/search?q=/docs/4.x/nesting%23reactive-props), [Modelable Properties](https://www.google.com/search?q=/docs/4.x/nesting%23binding-to-child-data-using-wiremodel), dll.)

Namun, ada kalanya menonaktifkan *bundling* ini diperlukan karena alasan performa. Di situlah `#[Isolate]` berperan.

## Penggunaan dasar

Terapkan atribut `#[Isolate]` pada **component** apa pun yang harus mengirimkan *request* secara terisolasi:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Isolate;
use Livewire\Component;
use App\Models\Post;

new #[Isolate] class extends Component { // [tl! highlight]
    public Post $post;

    public function refreshStats()
    {
        // Operasi yang berat (expensive)...
        $this->post->recalculateStatistics();
    }
};

```

Dengan `#[Isolate]`, *request* dari **component** ini tidak lagi akan digabungkan dengan pembaruan **component** lain, sehingga memungkinkan mereka untuk dieksekusi secara paralel.

> [!tip] Kapan bundling membantu vs menghambat
> *Bundling* sangat bagus untuk sebagian besar skenario, tetapi jika sebuah **component** melakukan operasi yang berat, *bundling* dapat memperlambat seluruh *request*. Mengisolasi **component** tersebut memungkinkannya berjalan secara paralel dengan pembaruan lainnya.

---

## Kapan harus menggunakan

Gunakan `#[Isolate]` ketika:

* **Component** melakukan operasi yang berat (query kompleks, API calls, komputasi berat).
* Beberapa **component** menggunakan `wire:poll` dan Anda menginginkan interval polling yang independen.
* **Component** mendengarkan *events* dan Anda tidak ingin satu **component** yang lambat memblokir yang lainnya.
* **Component** tidak perlu berkoordinasi dengan **component** lain di halaman tersebut.

## Contoh: Polling components

Berikut adalah contoh praktis dengan beberapa **polling components**:

```php
<?php // resources/views/components/⚡system-status.blade.php

use Livewire\Attributes\Isolate;
use Livewire\Component;

new #[Isolate] class extends Component { // [tl! highlight]
    public function checkStatus()
    {
        // API call eksternal yang berat...
        return ExternalService::getStatus();
    }
};

```

```blade
<div wire:poll.5s>
    Status: {{ $this->checkStatus() }}
</div>

```

Tanpa `#[Isolate]`, panggilan API yang lambat dari **component** ini akan menunda **component** lain di halaman. Dengan atribut ini, **component** melakukan polling secara independen tanpa memblokir yang lain.

---

## Lazy components terisolasi secara default

Saat menggunakan atribut `#[Lazy]`, **component** secara otomatis diisolasi untuk dimuat secara paralel. Anda dapat menonaktifkan perilaku ini jika diperlukan:

```php
<?php // resources/views/components/⚡revenue.blade.php

use Livewire\Attributes\Lazy;
use Livewire\Component;

new #[Lazy(isolate: false)] class extends Component { // [tl! highlight]
    // ...
};

```

Sekarang, beberapa komponen `revenue` akan menggabungkan *request* lazy-load mereka ke dalam satu *network request* tunggal.

---

## Pertimbangan (Trade-offs)

**Keuntungan:**

* Mencegah **component** yang lambat memblokir pembaruan lainnya.
* Memungkinkan eksekusi paralel yang sesungguhnya untuk operasi yang berat.
* Polling dan penanganan *event* yang independen.

**Kekurangan:**

* Lebih banyak *network requests* ke server.
* Tidak dapat berkoordinasi dengan **component** lain dalam *request* yang sama.
* *Overhead* server yang sedikit lebih tinggi dari banyak koneksi.
