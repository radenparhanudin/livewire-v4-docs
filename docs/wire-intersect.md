Direktif `wire:intersect` milik Livewire memungkinkan Anda menjalankan sebuah **action** saat sebuah elemen masuk atau keluar dari **viewport**. Ini sangat berguna untuk pemuatan konten secara malas (*lazy loading*), memicu analitik, atau membuat interaksi berbasis gulir (*scroll*).

## Penggunaan dasar

Bentuk paling sederhana menjalankan sebuah **action** saat sebuah elemen menjadi terlihat:

```blade
<div wire:intersect="loadMore">
    </div>

```

Ketika elemen masuk ke dalam **viewport**, **action** `loadMore` akan dipanggil pada **component** Anda.

## Event enter dan leave

Anda dapat menentukan apakah akan menjalankan **action** saat masuk (*enter*), keluar (*leave*), atau keduanya:

```blade
<div wire:intersect="trackView">...</div>

<div wire:intersect:enter="trackView">...</div>

<div wire:intersect:leave="pauseVideo">...</div>

```

## Visibility modifiers

Kontrol seberapa banyak bagian elemen yang harus terlihat sebelum memicu **action**:

```blade
<div wire:intersect="load">...</div>

<div wire:intersect.half="load">...</div>

<div wire:intersect.full="load">...</div>

<div wire:intersect.threshold.25="load">...</div>

```

## Margin

Tambahkan margin di sekitar **viewport** untuk memicu **action** sebelum/sesudah elemen masuk:

```blade
<div wire:intersect.margin.200px="loadMore">...</div>

<div wire:intersect.margin.10%="loadMore">...</div>

<div wire:intersect.margin.10%.25px.25px.25px="loadMore">...</div>

```

## Fire once

Gunakan **modifier** `.once` untuk memastikan **action** hanya dipicu pada persinggungan pertama kali saja:

```blade
<div wire:intersect.once="trackImpression">
    </div>

```

Ini sangat berguna untuk analitik atau pelacakan saat Anda hanya ingin mencatat pertama kali pengguna melihat sesuatu.

## Menggabungkan modifier

Anda dapat menggabungkan beberapa **modifier** untuk menciptakan perilaku yang presisi:

```blade
<div wire:intersect.once.half.margin.100px="loadSection">
    </div>

```

---

## Contoh penggunaan umum

### Infinite scroll

```blade
<?php

use Livewire\Component;

new class extends Component {
    public $page = 1;
    public $posts = [];

    public function mount()
    {
        $this->loadPosts();
    }

    public function loadPosts()
    {
        $newPosts = Post::latest()
            ->skip(($this->page - 1) * 10)
            ->take(10)
            ->get();

        $this->posts = array_merge($this->posts, $newPosts->toArray());
        $this->page++;
    }
};
?>

<div>
    @foreach ($posts as $post)
        <div>{{ $post['title'] }}</div>
    @endforeach

    <div wire:intersect="loadPosts">
        Loading more posts...
    </div>
</div>

```

### Lazy loading images

```blade
<?php

use Livewire\Component;

new class extends Component {
    public $imageLoaded = false;

    public function loadImage()
    {
        $this->imageLoaded = true;
    }
};
?>

<div>
    @if ($imageLoaded)
        <img src="/path/to/image.jpg" alt="Product">
    @else
        <div wire:intersect.once="loadImage" class="bg-gray-200 h-64">
            </div>
    @endif
</div>

```

---

## Perbandingan dengan x-intersect Alpine

Jika Anda terbiasa dengan Alpine.js, `wire:intersect` bekerja mirip dengan `x-intersect` tetapi memicu **Livewire actions** alih-alih ekspresi Alpine. **Modifiers** dan perilakunya dirancang agar terasa akrab bagi pengguna Alpine.

---

## Referensi

```blade
wire:intersect="action"
wire:intersect:enter="action"
wire:intersect:leave="action"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.once` | Hanya pemicu **action** pada persinggungan pertama |
| `.half` | Pemicu saat setengah bagian elemen terlihat |
| `.full` | Pemicu saat seluruh bagian elemen terlihat |
| `.threshold.[0-100]` | Pemicu pada persentase ambang batas visibilitas kustom |
| `.margin.[value]` | Menambahkan margin di sekitar **viewport** (misal: `.margin.200px`, `.margin.10%`) |
