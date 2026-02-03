**Properties** menyimpan dan mengelola **state** di dalam Livewire **components** Anda. Mereka didefinisikan sebagai **public properties** pada **component classes** dan dapat diakses serta dimodifikasi baik di sisi **server** maupun **client-side**.

## Initializing properties

Anda dapat menetapkan nilai awal untuk **properties** di dalam **method** `mount()` pada **component** Anda.

Perhatikan contoh berikut:

```php
<?php // resources/views/components/⚡todos.blade.php

use Livewire\Component;

new class extends Component {
    public $todos = [];

    public $todo = '';

    public function mount()
    {
        $this->todos = ['Beli belanjaan', 'Jalan-jalan dengan anjing', 'Menulis kode']; 
    }

    // ...
};

```

Dalam contoh ini, kita telah mendefinisikan **array** `todos` yang kosong dan menginisialisasinya dengan daftar tugas default di dalam **method** `mount()`. Sekarang, saat **component** di-**render** untuk pertama kalinya, tugas-tugas awal ini akan ditampilkan kepada pengguna.

## Bulk assignment

Terkadang menginisialisasi banyak **properties** di dalam **method** `mount()` bisa terasa sangat panjang (**verbose**). Untuk membantu hal ini, Livewire menyediakan cara praktis untuk menetapkan beberapa **properties** sekaligus melalui **method** `fill()`. Dengan mengirimkan **associative array** yang berisi nama **property** dan nilainya masing-masing, Anda dapat mengatur beberapa **properties** secara simultan dan mengurangi pengulangan baris kode di `mount()`.

Sebagai contoh:

```php
<?php // resources/views/components/post/⚡edit.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $post;

    public $title;

    public $description;

    public function mount(Post $post)
    {
        $this->post = $post;

        $this->fill( 
            $post->only('title', 'description'), 
        ); 
    }

    // ...
};

```

Karena `$post->only(...)` mengembalikan sebuah **associative array** dari **model attributes** dan nilainya berdasarkan nama yang Anda masukkan, **property** `$title` dan `$description` akan diatur di awal sesuai dengan `title` dan `description` dari **model** `$post` di database tanpa harus menetapkan masing-masing secara manual.

## Data binding

Livewire mendukung **two-way data binding** melalui **HTML attribute** `wire:model`. Ini memungkinkan Anda untuk mensinkronisasi data antara **component properties** dan **HTML inputs** dengan mudah, menjaga agar antarmuka pengguna (**user interface**) dan **component state** tetap selaras.

Mari gunakan **directive** `wire:model` untuk menghubungkan **property** `$todo` di dalam komponen `todos` ke elemen **input** dasar:

```php
<?php // resources/views/components/⚡todos.blade.php

use Livewire\Component;

new class extends Component {
    public $todos = [];

    public $todo = '';

    public function add()
    {
        $this->todos[] = $this->todo;

        $this->todo = '';
    }

    // ...
};

```

```blade
<div>
    <input type="text" wire:model="todo" placeholder="Todo..."> 

    <button wire:click="add">Tambah Todo</button>

    <ul>
        @foreach ($todos as $todo)
            <li wire:key="{{ $loop->index }}">{{ $todo }}</li>
        @endforeach
    </ul>
</div>

```

Pada contoh di atas, nilai dari **text input** akan sinkron dengan **property** `$todo` di **server** saat tombol "Tambah Todo" diklik.

Ini hanyalah bagian dasar dari `wire:model`. Untuk informasi lebih mendalam tentang **data binding**, silakan periksa [dokumentasi tentang forms](https://www.google.com/search?q=/docs/4.x/forms).

## Resetting properties

Terkadang, Anda mungkin perlu mengembalikan **properties** Anda kembali ke keadaan awal setelah sebuah aksi dilakukan oleh pengguna. Dalam kasus ini, Livewire menyediakan **method** `reset()` yang menerima satu atau lebih nama **property** dan mengembalikan nilainya ke keadaan awal.

Pada contoh di bawah, kita dapat menghindari duplikasi kode dengan menggunakan `$this->reset()` untuk mengosongkan kolom `todo` setelah tombol "Tambah Todo" diklik:

```php
<?php // resources/views/components/⚡todos.blade.php

use Livewire\Component;

new class extends Component {
    public $todos = [];

    public $todo = '';

    public function addTodo()
    {
        $this->todos[] = $this->todo;

        $this->reset('todo'); 
    }

    // ...
};

```

Setelah pengguna mengklik "Tambah Todo", kolom **input** yang menampung tugas yang baru saja ditambahkan akan kosong, memungkinkan pengguna untuk menulis tugas baru.

> [!warning] `reset()` tidak akan bekerja pada nilai yang diatur di `mount()`
> `reset()` akan mengembalikan sebuah **property** ke keadaannya **sebelum** **method** `mount()` dipanggil. Jika Anda menginisialisasi **property** di dalam `mount()` dengan nilai yang berbeda, Anda harus mengembalikan nilai **property** tersebut secara manual.

## Pulling properties

Alternatif lainnya, Anda dapat menggunakan **method** `pull()` untuk mengembalikan nilai ke awal (**reset**) sekaligus mengambil nilainya dalam satu operasi.

Berikut adalah contoh yang sama seperti di atas, tetapi disederhanakan dengan `pull()`:

```php
public function addTodo()
{
    // Mengambil nilai $todo untuk dimasukkan ke array, lalu me-reset $todo
    $this->todos[] = $this->pull('todo'); 
}

```

`pull()` juga dapat digunakan untuk me-**reset** dan mengambil beberapa atau semua **properties**:

```php
// Sama seperti $this->all() kemudian $this->reset();
$this->pull();

// Sama seperti $this->only(...) kemudian $this->reset(...);
$this->pull(['title', 'content']);

```

---

## Supported property types

Livewire mendukung sekumpulan tipe **property** yang terbatas karena pendekatan uniknya dalam mengelola data komponen di antara **server requests**.

Setiap **property** dalam Livewire **component** akan di-**serialized** atau di-"**dehydrated**" menjadi JSON di antara **requests**, kemudian di-"**hydrated**" dari JSON kembali menjadi PHP untuk **request** berikutnya.

### Primitive types

Livewire mendukung tipe primitif seperti: `Array`, `String`, `Integer`, `Float`, `Boolean`, dan `Null`.

### Common PHP types

Livewire juga mendukung tipe objek PHP umum yang digunakan dalam aplikasi Laravel. Tipe-tipe ini akan di-**dehydrated** menjadi JSON dan di-**hydrated** kembali menjadi PHP pada setiap **request**.

| Type | Full Class Name |
| --- | --- |
| BackedEnum | `BackedEnum` |
| Collection | `Illuminate\Support\Collection` |
| Eloquent Collection | `Illuminate\Database\Eloquent\Collection` |
| Model | `Illuminate\Database\Eloquent\Model` |
| DateTime | `DateTime` |
| Carbon | `Carbon\Carbon` |
| Stringable | `Illuminate\Support\Stringable` |

> [!warning] Eloquent Collections dan Models
> Saat menyimpan **Eloquent Collections** dan **Models** di Livewire **properties**, sadarilah batasan berikut:
> * **Query constraints tidak dipertahankan**: Batasan kueri tambahan seperti `select(...)` tidak akan diterapkan kembali pada **requests** berikutnya.
> * **Dampak Performa**: Menyimpan koleksi Eloquent yang besar dapat menyebabkan masalah performa. Pertimbangkan untuk menggunakan [computed properties](https://www.google.com/search?q=/docs/4.x/computed-properties).
> 
> 

---

## Accessing properties dari JavaScript

Karena Livewire **properties** juga tersedia di browser melalui JavaScript, Anda dapat mengakses dan memanipulasi representasi JavaScript mereka dari [AlpineJS](https://alpinejs.dev/).

### Accessing properties

Livewire menyediakan **magic object** `$wire` ke Alpine. Anda dapat memperlakukan `$wire` seperti versi JavaScript dari Livewire **component** Anda.

Contoh menampilkan panjang karakter secara **real-time**:

```blade
<div>
    <input type="text" wire:model="todo">
    Panjang karakter: <h2 x-text="$wire.todo.length"></h2>
</div>

```

### Manipulating properties

Anda juga dapat mengubah nilai **property** di JavaScript menggunakan `$wire`:

```blade
<button x-on:click="$wire.todo = ''">Hapus</button>

```

---

## Security concerns

Selalu perlakukan **public properties** sebagai **user input** — seolah-olah itu adalah **request input** dari **endpoint** tradisional. Sangat penting untuk melakukan validasi dan otorisasi sebelum menyimpannya ke database.

### Don't trust property values

Karena kita menyimpan `id` sebagai **public property**, seseorang bisa saja mengubahnya melalui **DevTools** browser untuk memanipulasi data orang lain.

Untuk mencegah serangan ini, gunakan strategi berikut:

#### 1. Authorizing the input

Gunakan **Laravel authorization** di dalam **method** Anda:

```php
public function update()
{
    $post = Post::findOrFail($this->id);
    $this->authorize('update', $post); 
    $post->update(...);
}

```

#### 2. Locking the property

Gunakan **attribute** `#[Locked]` untuk mencegah **property** dimodifikasi dari sisi **client**:

```php
use Livewire\Attributes\Locked;

new class extends Component {
    #[Locked] 
    public $id;
};

```

### Eloquent constraints tidak dipertahankan

Gunakan **Computed Properties** jika Anda menggunakan kueri yang memiliki batasan khusus (seperti `select()`).

Contoh dengan `#[Computed]`:

```php
use Livewire\Attributes\Computed;

#[Computed]
public function todos()
{
    return Auth::user()->todos()->select(['title', 'content'])->get();
}

```

Akses di Blade dengan `$this->todos`.

---

## See also (Lihat juga)

* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Menghubungkan properti ke form input dengan `wire:model`.
* **[Computed Properties](https://www.google.com/search?q=/docs/4.x/computed-properties)** — Membuat nilai turunan dengan **memoization** otomatis.
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Memvalidasi nilai properti sebelum disimpan.
* **[Locked Attribute](https://www.google.com/search?q=/docs/4.x/attribute-locked)** — Mencegah properti dimanipulasi dari sisi client.
* **[Alpine](https://www.google.com/search?q=/docs/4.x/alpine)** — Mengakses dan memanipulasi properti dari JavaScript.
