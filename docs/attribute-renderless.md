Atribut `#[Renderless]` melewatkan fase pemuatan ulang (*rendering phase*) dalam siklus hidup Livewire ketika sebuah **action** dipanggil, sehingga meningkatkan performa untuk **actions** yang tidak memodifikasi tampilan (*view*) dari komponen tersebut.

## Penggunaan dasar

Terapkan atribut `#[Renderless]` pada metode **action** apa pun yang tidak memerlukan perenderan ulang komponen:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Renderless;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    #[Renderless] // [tl! highlight]
    public function incrementViewCount()
    {
        $this->post->incrementViewCount();
    }
};

```

```blade
<div>
    <h1>{{ $post->title }}</h1>
    <p>{{ $post->content }}</p>

    <div wire:intersect="incrementViewCount"></div>
</div>

```

Contoh di atas menggunakan `wire:intersect` untuk memanggil `incrementViewCount()` saat pengguna menggulir ke bawah. Karena `#[Renderless]` diterapkan, jumlah tampilan dicatat tetapi template tidak di-*render* ulang—tidak ada bagian halaman yang terpengaruh.

---

## Kapan harus menggunakan

Gunakan `#[Renderless]` ketika sebuah **action**:

* Hanya melakukan operasi backend (logging, analitik, pelacakan).
* Tidak memodifikasi properti apa pun yang memengaruhi tampilan yang di-*render*.
* Perlu dijalankan sering tanpa menyebabkan proses *re-render* yang tidak perlu.

Kasus penggunaan yang umum meliputi:

* Melacak interaksi pengguna (klik, gulir, waktu di halaman).
* Mengirim *events* analitik.
* Memperbarui penghitung atau metrik.
* Menjalankan operasi latar belakang.

---

## Pendekatan alternatif

### Menggunakan skipRender()

Jika Anda perlu melewatkan perenderan secara kondisional atau lebih suka tidak menggunakan atribut, Anda dapat memanggil `skipRender()` langsung di dalam **action** Anda:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function incrementViewCount()
    {
        $this->post->incrementViewCount();

        $this->skipRender(); // [tl! highlight]
    }
};

```

### Menggunakan modifier .renderless

Anda juga dapat melewatkan perenderan langsung dari elemen menggunakan **modifier** `.renderless`:

```blade
<button type="button" wire:click.renderless="incrementViewCount">
    Track View
</button>

```

Pendekatan ini berguna untuk kasus satu kali di mana Anda tidak ingin menambahkan atribut pada metode tersebut.

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut tentang **actions** dan optimalisasi performa, lihat [dokumentasi Actions](https://www.google.com/search?q=/docs/4.x/actions%23skipping-re-renders).
