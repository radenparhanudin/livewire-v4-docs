Direktif `wire:show` milik Livewire memudahkan Anda untuk menampilkan dan menyembunyikan elemen berdasarkan hasil dari sebuah ekspresi.

Direktif `wire:show` berbeda dengan penggunaan `@if` di Blade karena ia beralih visibilitas elemen menggunakan CSS (`display: none`) alih-alih menghapus elemen dari DOM sepenuhnya. Ini berarti elemen tetap ada di halaman tetapi tersembunyi, memungkinkan transisi yang lebih mulus tanpa memerlukan *server round-trip*.

## Penggunaan dasar

Berikut adalah contoh praktis penggunaan `wire:show` untuk beralih modal "Create Post":

```php
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $showModal = false;

    public $content = '';

    public function save()
    {
        Post::create(['content' => $this->content]);

        $this->reset('content');

        $this->showModal = false;
    }
}

```

```blade
<div>
    <button x-on:click="$wire.showModal = true">New Post</button>

    <div wire:show="showModal">
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>

            <button type="submit">Save Post</button>
        </form>
    </div>
</div>

```

Ketika tombol "New Post" diklik, modal akan muncul tanpa *server round-trip*. Setelah berhasil menyimpan postingan, modal akan disembunyikan dan formulir disetel ulang (*reset*).

## Menggunakan transisi

Anda dapat menggabungkan `wire:show` dengan transisi Alpine.js untuk membuat animasi tampil/sembunyi yang mulus. Karena `wire:show` hanya beralih properti CSS `display`, direktif `x-transition` milik Alpine bekerja dengan sempurna bersamanya:

```blade
<div>
    <button x-on:click="$wire.showModal = true">New Post</button>

    <div wire:show="showModal" x-transition.duration.500ms>
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>
            <button type="submit">Save Post</button>
        </form>
    </div>
</div>

```

Class transisi Alpine.js di atas akan membuat efek *fade* dan *scale* saat modal ditampilkan dan disembunyikan.

[Lihat dokumentasi lengkap x-transition →](https://alpinejs.dev/directives/transition)

---

## Referensi

```blade
wire:show="expression"

```

Direktif ini tidak memiliki **modifiers**.
