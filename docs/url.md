Livewire memungkinkan Anda menyimpan properti **component** di dalam **URL query string**. Sebagai contoh, Anda mungkin ingin properti `$search` di dalam **component** Anda disertakan dalam URL: `https://example.com/users?search=bob`. Hal ini sangat berguna untuk fitur-fitur seperti penyaringan (*filtering*), pengurutan (*sorting*), dan **pagination**, karena memungkinkan pengguna untuk membagikan atau menyimpan tautan (*bookmark*) status tertentu dari suatu halaman.

## Penggunaan dasar

Di bawah ini adalah **component** `show-users` yang memungkinkan Anda mencari pengguna berdasarkan nama melalui input teks sederhana:

```php
<?php // resources/views/components/⚡show-users.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $search = '';

    #[Computed]
    public function users()
    {
        return User::search($this->search)->get();
    }
};

```

```blade
<div>
    <input type="text" wire:model.live="search">

    <ul>
        @foreach ($this->users as $user)
            <li wire:key="{{ $user->id }}">{{ $user->name }}</li>
        @endforeach
    </ul>
</div>

```

Seperti yang Anda lihat, karena input teks menggunakan `wire:model.live="search"`, saat pengguna mengetik, permintaan jaringan akan dikirim untuk memperbarui properti `$search` dan menampilkan daftar pengguna yang telah difilter di halaman.

Namun, jika pengunjung memuat ulang (*refresh*) halaman, nilai pencarian dan hasilnya akan hilang.

Untuk mempertahankan nilai pencarian saat halaman dimuat ulang agar pengunjung dapat membagikan URL tersebut, kita dapat menyimpan nilai pencarian di **URL query string** dengan menambahkan atribut `#[Url]` di atas properti `$search` seperti berikut:

```php
<?php // resources/views/components/⚡show-users.blade.php

use Livewire\Attributes\Computed;
use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    #[Url] // [tl! highlight]
    public $search = '';

    #[Computed]
    public function users()
    {
        return User::search($this->search)->get();
    }
};

```

Sekarang, jika pengguna mengetik "bob" ke dalam kolom pencarian, bilah URL di browser akan menampilkan:

```
https://example.com/users?search=bob

```

Jika mereka memuat URL ini dari jendela browser baru, "bob" akan otomatis terisi di kolom pencarian, dan hasil pengguna akan difilter sesuai dengan nilai tersebut.

---

## Menginisialisasi properti dari URL

Seperti yang Anda lihat pada contoh sebelumnya, ketika sebuah properti menggunakan `#[Url]`, ia tidak hanya menyimpan nilainya yang diperbarui ke dalam **query string**, tetapi juga mengambil nilai **query string** yang sudah ada saat halaman dimuat.

Sebagai contoh, jika pengguna mengunjungi URL `https://example.com/users?search=bob`, Livewire akan mengatur nilai awal `$search` menjadi "bob".

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public $search = ''; // Akan diatur menjadi "bob"...

    // ...
}

```

### Nullable properties

Secara default, jika sebuah halaman dimuat dengan entri **query string** kosong seperti `?search=`, Livewire akan menganggap nilai tersebut sebagai string kosong (`""`). Dalam banyak kasus ini sudah sesuai, namun terkadang Anda ingin `?search=` dianggap sebagai `null`.

Dalam kasus tersebut, Anda dapat menggunakan **nullable typehint** seperti ini:

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public ?string $search; // [tl! highlight]

    // ...
}

```

Karena adanya tanda `?` pada **typehint** di atas, Livewire akan melihat `?search=` dan mengatur `$search` menjadi `null` alih-alih string kosong.

Ini juga berlaku sebaliknya: jika Anda mengatur `$this->search = null` di dalam aplikasi, hal tersebut akan direpresentasikan di **query string** sebagai `?search=`.

---

## Menggunakan alias

Livewire memberi Anda kontrol penuh atas nama apa yang ditampilkan di **URL query string**. Misalnya, Anda memiliki properti `$search` tetapi ingin menyamarkan nama properti aslinya atau menyingkatnya menjadi `q`.

Anda dapat menentukan alias **query string** dengan memberikan parameter `as` pada atribut `#[Url]`:

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(as: 'q')]
    public $search = '';

    // ...
}

```

Sekarang, saat pengguna mengetik "bob", URL akan menampilkan `?q=bob` alih-alih `?search=bob`.

---

## Mengecualikan nilai tertentu

Secara default, Livewire hanya akan memasukkan entri ke dalam **query string** ketika nilainya telah berubah dari nilai saat inisialisasi. Namun, ada skenario di mana Anda ingin kontrol lebih lanjut tentang nilai mana yang dikecualikan Livewire dari **query string**. Dalam kasus ini, Anda dapat menggunakan parameter `except`.

Contohnya, pada **component** di bawah, nilai awal `$search` diubah di dalam `mount()`. Untuk memastikan browser hanya akan mengecualikan `search` dari **query string** jika nilainya adalah string kosong, parameter `except` telah ditambahkan ke `#[Url]`:

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(except: '')]
    public $search = '';

    public function mount() {
        $this->search = auth()->user()->username;
    }

    // ...
}

```

Tanpa `except`, Livewire akan menghapus entri `search` dari URL setiap kali nilainya sama dengan nilai awal dari `auth()->user()->username`. Dengan menggunakan `except: ''`, Livewire akan tetap menampilkan nilai di URL kecuali jika `search` benar-benar kosong.

---

## Menampilkan saat halaman dimuat

Secara default, Livewire hanya akan menampilkan nilai di **query string** setelah nilai tersebut diubah di halaman. Jika nilai default `$search` adalah string kosong `""`, maka saat kolom pencarian kosong, tidak ada nilai yang muncul di URL.

Jika Anda ingin entri `?search` selalu disertakan dalam URL bahkan saat nilainya kosong, Anda dapat menggunakan parameter `keep`:

```php
#[Url(keep: true)]
public $search = '';

```

Sekarang, saat halaman dimuat, URL akan otomatis berubah menjadi: `https://example.com/users?search=`

---

## Menyimpan dalam riwayat (history)

Secara default, Livewire menggunakan [`history.replaceState()`](https://www.google.com/search?q=%5Bhttps://developer.mozilla.org/en-US/docs/Web/API/History/replaceState%5D(https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState)) untuk mengubah URL. Artinya, saat Livewire memperbarui **query string**, ia memodifikasi entri saat ini di riwayat browser alih-alih menambahkannya sebagai entri baru.

Karena Livewire "mengganti" (*replace*) riwayat saat ini, menekan tombol "kembali" (*back*) di browser akan membawa pengguna ke halaman sebelumnya, bukan ke nilai `?search=` sebelumnya.

Untuk memaksa Livewire menggunakan `history.pushState`, Anda dapat memberikan parameter `history: true`:

```php
#[Url(history: true)]
public $search = '';

```

Dengan ini, jika pengguna mengubah pencarian dari "bob" ke "frank" lalu menekan tombol kembali, nilai pencarian (dan URL) akan kembali ke "bob".

---

## Menggunakan metode queryString

**Query string** juga dapat didefinisikan sebagai metode di dalam **component**. Ini berguna jika beberapa properti memiliki opsi yang dinamis.

```php
protected function queryString()
{
    return [
        'search' => [
            'as' => 'q',
        ],
    ];
}

```

---

## Trait hooks

Livewire juga menawarkan [hooks](https://www.google.com/search?q=/docs/4.x/lifecycle-hooks) untuk **query strings** di dalam **trait**.

```php
trait WithSorting
{
    protected function queryStringWithSorting()
    {
        return [
            'sortBy' => ['as' => 'sort'],
            'sortDirection' => ['as' => 'direction'],
        ];
    }
}

```

---

## See also

* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Sinkronisasi properti dengan parameter URL
* **[Navigate](https://www.google.com/search?q=/docs/4.x/navigate)** — Mempertahankan status URL selama navigasi SPA
* **[Url Attribute](https://www.google.com/search?q=/docs/4.x/attribute-url)** — Menghubungkan properti ke URL query strings
* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Bekerja dengan parameter route
