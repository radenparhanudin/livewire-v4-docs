Menggunakan Livewire terasa seperti menempelkan *class* PHP sisi server langsung ke browser web. Hal-hal seperti memanggil fungsi server secara langsung melalui penekanan tombol mendukung ilusi ini. Namun pada kenyataannya, itu hanyalah sebuah ilusi.

Di balik layar, Livewire sebenarnya berperilaku jauh lebih mirip dengan aplikasi web standar. Ia me-*render* HTML statis ke browser, mendengarkan *event* browser, lalu membuat permintaan AJAX untuk memanggil kode di sisi server.

Karena setiap permintaan AJAX yang dibuat Livewire ke server bersifat "**stateless**" (artinya tidak ada proses backend yang berjalan lama untuk menjaga status komponen tetap hidup), Livewire harus membuat ulang status terakhir yang diketahui dari sebuah komponen sebelum melakukan pembaruan apa pun.

Livewire melakukan ini dengan mengambil "**snapshot**" dari komponen PHP setelah setiap pembaruan di sisi server sehingga komponen tersebut dapat dibuat ulang atau *dilanjutkan* (*resumed*) pada permintaan berikutnya.

Dalam dokumentasi ini, kami akan merujuk proses pengambilan *snapshot* sebagai "**dehydration**" dan proses pembuatan ulang komponen dari sebuah *snapshot* sebagai "**hydration**".

## Dehydrating

Saat Livewire melakukan *dehydrate* pada komponen di sisi server, ia melakukan dua hal:

* Me-*render* template komponen menjadi HTML
* Membuat *snapshot* JSON dari komponen tersebut

### Rendering HTML

Setelah komponen di-*mount* atau pembaruan telah dilakukan, Livewire memanggil metode `render()` komponen untuk mengubah template Blade menjadi HTML mentah.

Ambil contoh komponen `counter` berikut:

```php
<?php

use Livewire\Component;

new class extends Component {
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            Count: {{ $count }}

            <button wire:click="increment">+</button>
        </div>
        HTML;
    }
};

```

Setelah setiap *mount* atau pembaruan, Livewire akan me-*render* komponen `counter` di atas menjadi HTML berikut:

```html
<div>
    Count: 1

    <button wire:click="increment">+</button>
</div>

```

### The snapshot

Agar dapat membuat ulang komponen `counter` di server selama permintaan berikutnya, sebuah *snapshot* JSON dibuat untuk menangkap sebanyak mungkin status komponen:

```js
{
    state: {
        count: 1,
    },

    memo: {
        name: 'counter',

        id: '1526456',
    },
}

```

Perhatikan dua bagian berbeda dari *snapshot* tersebut: `memo` dan `state`.

Bagian `memo` digunakan untuk menyimpan informasi yang diperlukan untuk mengidentifikasi dan membuat ulang komponen, sedangkan bagian `state` menyimpan nilai dari semua properti publik komponen.

> [!info]
> Snapshot di atas adalah versi ringkas dari snapshot asli di Livewire. Dalam aplikasi nyata, snapshot berisi jauh lebih banyak informasi, seperti error validasi, daftar komponen anak, lokal, dan banyak lagi. Untuk melihat detail objek snapshot, Anda dapat merujuk ke [dokumentasi skema snapshot](https://www.google.com/search?q=/docs/4.x/javascript%23the-snapshot-object).

### Menyematkan snapshot di dalam HTML

Saat komponen pertama kali di-*render*, Livewire menyimpan *snapshot* sebagai JSON di dalam atribut HTML bernama `wire:snapshot`. Dengan cara ini, inti JavaScript Livewire dapat mengekstrak JSON tersebut dan mengubahnya menjadi objek *run-time*:

```html
<div wire:id="..." wire:snapshot="{ state: {...}, memo: {...} }">
    Count: 1

    <button wire:click="increment">+</button>
</div>

```

## Hydrating

Ketika pembaruan komponen dipicu, misalnya tombol "+" ditekan pada komponen `counter`, sebuah *payload* seperti berikut dikirim ke server:

```js
{
    calls: [
        { method: 'increment', params: [] },
    ],

    snapshot: {
        state: {
            count: 1,
        },

        memo: {
            name: 'counter',

            id: '1526456',
        },
    }
}

```

Sebelum Livewire dapat memanggil metode `increment`, ia harus terlebih dahulu membuat instansi `counter` baru dan mengisinya dengan status dari *snapshot* tersebut.

Berikut adalah *pseudo-code* PHP yang mencapai hasil ini:

```php
$state = request('snapshot.state');
$memo = request('snapshot.memo');

$instance = Livewire::new($memo['name'], $memo['id']);

foreach ($state as $property => $value) {
    $instance[$property] = $value;
}

```

Jika Anda mengikuti skrip di atas, Anda akan melihat bahwa setelah membuat objek `counter`, properti publiknya diatur berdasarkan status yang disediakan dari *snapshot*.

## Advanced hydration

Contoh `counter` di atas berfungsi dengan baik untuk mendemonstrasikan konsep *hydration*; namun, itu hanya menunjukkan bagaimana Livewire menangani *hydration* nilai sederhana seperti integer (`1`).

Seperti yang Anda ketahui, Livewire mendukung lebih banyak tipe properti yang canggih di luar integer. Mari kita lihat contoh yang sedikit lebih kompleks - komponen `todos`:

```php
<?php

use Livewire\Component;

new class extends Component {
    public $todos;

    public function mount() {
        $this->todos = collect([
            'first',
            'second',
            'third',
        ]);
    }
};

```

Di sini kita mengatur properti `$todos` menjadi sebuah [Laravel collection](https://laravel.com/docs/collections#main-content). Karena JSON tidak memiliki cara untuk merepresentasikan Laravel collection secara langsung, Livewire membuat polanya sendiri untuk mengasosiasikan metadata dengan data murni di dalam *snapshot*.

Berikut adalah objek status *snapshot* untuk komponen `todos` ini:

```js
state: {
    todos: [
        [ 'first', 'second', 'third' ],
        { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
    ],
},

```

Livewire mendukung sintaks status alternatif dalam bentuk **tuple** (array berisi dua item):

```js
todos: [
    [ 'first', 'second', 'third' ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],

```

Ketika Livewire menemui sebuah tuple saat melakukan *hydrating* pada status komponen, ia menggunakan informasi yang disimpan di elemen kedua tuple untuk melakukan *hydrating* status yang disimpan di elemen pertama secara lebih cerdas.

### Deeply nested tuples

Salah satu keuntungan nyata dari pendekatan ini adalah kemampuan untuk melakukan *dehydrate* dan *hydrate* pada properti yang bersarang sangat dalam (*deeply nested*).

Misalnya, jika item ketiga dalam koleksi adalah sebuah [Laravel Stringable](https://laravel.com/docs/helpers#method-str):

```php
$this->todos = collect([
    'first',
    'second',
    str('third'),
]);

```

Snapshot yang di-*dehydrate* untuk status komponen ini sekarang akan terlihat seperti ini:

```js
todos: [
    [
        'first',
        'second',
        [ 'third', { s: 'str' } ],
    ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],

```

Seperti yang Anda lihat, item ketiga dalam koleksi telah di-*dehydrate* menjadi tuple metadata. Elemen pertama adalah nilai string biasa, dan elemen kedua adalah tanda (*flag*) yang memberitahu Livewire bahwa string ini adalah sebuah *stringable*.

### Mendukung tipe properti kustom

Secara internal, Livewire memiliki dukungan *hydration* untuk tipe PHP dan Laravel yang paling umum. Namun, jika Anda ingin mendukung tipe yang belum didukung, Anda dapat melakukannya menggunakan [Synthesizers](https://www.google.com/search?q=/docs/4.x/synthesizers) — mekanisme internal Livewire untuk proses *hydrating/dehydrating* tipe properti non-primitif.

---

## Lihat juga

* **[Lifecycle Hooks](https://www.google.com/search?q=/docs/4.x/lifecycle-hooks)** — Gunakan hook hydrate() dan dehydrate()
* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Bagaimana properti dijaga di antara permintaan
* **[Morphing](https://www.google.com/search?q=/docs/4.x/morphing)** — Bagaimana Livewire memperbarui DOM
* **[Synthesizers](https://www.google.com/search?q=/docs/4.x/synthesizers)** — Kustomisasi serialisasi properti
