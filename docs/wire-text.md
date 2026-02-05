`wire:text` adalah direktif yang memperbarui konten teks sebuah elemen secara dinamis berdasarkan properti atau ekspresi **component**. Berbeda dengan penggunaan sintaks `{{ }}` milik Blade, `wire:text` memperbarui konten tanpa memerlukan *network roundtrip* untuk me-*render* ulang **component**.

Jika Anda terbiasa dengan direktif `x-text` milik Alpine, keduanya pada dasarnya memiliki fungsi yang sama.

## Penggunaan dasar

Berikut adalah contoh penggunaan `wire:text` untuk menampilkan pembaruan properti Livewire secara optimis tanpa menunggu *network roundtrip*.

```php
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public $likes;

    public function mount()
    {
        $this->likes = $this->post->like_count;
    }

    public function like()
    {
        $this->post->like();

        $this->likes = $this->post->fresh()->like_count;
    }
}

```

```blade
<div>
    <button x-on:click="$wire.likes++" wire:click="like">❤️ Like</button>

    Likes: <span wire:text="likes"></span>
</div>

```

Ketika tombol diklik, `$wire.likes++` segera memperbarui angka yang ditampilkan melalui `wire:text`, sementara `wire:click="like"` tetap menjalankan proses penyimpanan perubahan ke database di latar belakang.

Pola ini menjadikan `wire:text` sangat cocok untuk membangun **optimistic UI** di Livewire.

---

## Referensi

```blade
wire:text="expression"

```

Direktif ini tidak memiliki **modifiers**.
