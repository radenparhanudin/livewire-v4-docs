Atribut `#[Computed]` memungkinkan Anda membuat properti turunan yang disimpan dalam cache selama sebuah *request*, memberikan keuntungan performa saat mengakses operasi yang berat (*expensive operations*) berkali-kali.

## Penggunaan dasar

Terapkan atribut `#[Computed]` ke metode apa pun untuk mengubahnya menjadi properti yang di-cache:

```php
<?php // resources/views/components/user/⚡show.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $userId;

    #[Computed] // [tl! highlight]
    public function user()
    {
        return User::find($this->userId);
    }

    public function follow()
    {
        Auth::user()->follow($this->user);
    }
};

```

```blade
<div>
    <h1>{{ $this->user->name }}</h1>
    <span>{{ $this->user->email }}</span>

    <button wire:click="follow">Follow</button>
</div>

```

Metode `user()` diakses seperti properti menggunakan `$this->user`. Pertama kali dipanggil, hasilnya akan di-cache dan digunakan kembali untuk sisa *request* tersebut.

> [!info] Harus menggunakan `$this` di dalam templates
> Berbeda dengan properti normal, **computed properties** harus diakses melalui `$this` di dalam template Anda. Contohnya, gunakan `$this->posts` alih-alih `$posts`.

---

## Keuntungan Performa

**Computed properties** menyimpan hasilnya di cache selama durasi satu *request*. Jika Anda mengakses `$this->posts` beberapa kali, metode di baliknya hanya akan dieksekusi sekali:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    #[Computed] // [tl! highlight]
    public function posts()
    {
        return Auth::user()->posts; // Hanya melakukan query database satu kali
    }
};

```

Ini memungkinkan Anda mengakses nilai turunan secara bebas tanpa khawatir tentang implikasi performa.

---

## Membersihkan cache (Busting the cache)

Jika data yang mendasarinya berubah selama *request*, Anda dapat membersihkan cache menggunakan `unset()`:

```php
public function createPost()
{
    if ($this->posts->count() > 10) {
        throw new \Exception('Maximum post count exceeded');
    }

    Auth::user()->posts()->create(...);

    unset($this->posts); // Membersihkan cache [tl! highlight]
}

```

Setelah membuat postingan baru, `unset($this->posts)` akan menghapus cache sehingga akses berikutnya akan mengambil data yang sudah diperbarui.

---

## Caching antar request

Secara default, **computed properties** hanya menyimpan cache dalam satu *request*. Untuk menyimpan cache di beberapa *request* yang berbeda, gunakan parameter `persist`:

```php
#[Computed(persist: true)] // [tl! highlight]
public function user()
{
    return User::find($this->userId);
}

```

Ini akan menyimpan nilai di cache selama 3600 detik (1 jam). Anda dapat menyesuaikan durasinya:

```php
#[Computed(persist: true, seconds: 7200)] // 2 jam [tl! highlight]

```

---

## Caching di semua component

Untuk berbagi nilai cache ke seluruh instansi **component** di aplikasi Anda, gunakan parameter `cache`:

```php
#[Computed(cache: true)] // [tl! highlight]
public function posts()
{
    return Post::all();
}

```

Anda juga dapat menetapkan kunci cache kustom secara opsional:

```php
#[Computed(cache: true, key: 'homepage-posts')] // [tl! highlight]

```

---

## Kapan harus menggunakan

**Computed properties** sangat berguna ketika:

* **Mengakses data berat secara kondisional** — Hanya melakukan query ke database jika nilainya benar-benar digunakan di template.
* **Menggunakan inline templates** — Tidak ada kesempatan untuk mengirim data melalui `render()`.
* **Menghilangkan metode render** — Mengikuti konvensi *single-file component* versi 4.
* **Mengakses nilai yang sama berkali-kali** — Caching otomatis mencegah query yang mubazir.

---

## Batasan

> [!warning] Tidak didukung pada Form objects
> **Computed properties** tidak dapat digunakan pada objek `Livewire\Form`. Mencoba mengaksesnya melalui `$form->property` akan menyebabkan error.

---

## Pelajari lebih lanjut

Untuk informasi mendalam tentang **computed properties**, strategi caching, dan kasus penggunaan tingkat lanjut, lihat [dokumentasi Computed Properties](https://www.google.com/search?q=/docs/4.x/computed-properties).

---

## Referensi

```php
#[Computed(
    bool $persist = false,
    int $seconds = 3600,
    bool $cache = false,
    ?string $key = null,
    mixed $tags = null,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$persist` | `bool` | `false` | Menyimpan nilai di cache lintas *requests* untuk instansi komponen yang sama. |
| `$seconds` | `int` | `3600` | Durasi dalam detik untuk menyimpan nilai di cache. |
| `$cache` | `bool` | `false` | Menyimpan nilai di cache untuk semua instansi komponen. |
| `$key` | `?string` | `null` | Kunci cache kustom (dibuat otomatis jika tidak diisi). |
| `$tags` | `mixed` | `null` | Tag cache (membutuhkan *cache driver* yang mendukung tag). |
