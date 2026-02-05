Ketika sebuah komponen Livewire memperbarui DOM browser, ia melakukannya dengan cara cerdas yang kami sebut "**morphing**". Istilah *morph* (mengubah bentuk) digunakan sebagai kontras dari kata *replace* (mengganti).

Alih-alih *mengganti* seluruh HTML komponen dengan HTML baru hasil render setiap kali komponen diperbarui, Livewire secara dinamis membandingkan HTML saat ini dengan HTML baru, mengidentifikasi perbedaan, dan melakukan perubahan secara "bedah" hanya pada bagian HTML yang memang memerlukan perubahan.

Hal ini memberikan keuntungan berupa terjaganya elemen-elemen yang sudah ada dan tidak berubah. Sebagai contoh, *event listeners*, *focus state*, dan nilai pada *input form* semuanya tetap terjaga di antara pembaruan Livewire. Tentu saja, *morphing* juga menawarkan peningkatan performa dibandingkan dengan menghapus dan me-render ulang seluruh DOM pada setiap pembaruan.

## Cara kerja morphing

Untuk memahami bagaimana Livewire menentukan elemen mana yang harus diperbarui, perhatikan komponen `Todos` sederhana berikut:

```php
class Todos extends Component
{
    public $todo = '';

    public $todos = [
        'first',
        'second',
    ];

    public function add()
    {
        $this->todos[] = $this->todo;
    }
}

```

```blade
<form wire:submit="add">
    <ul>
        @foreach ($todos as $item)
            <li wire:key="{{ $loop->index }}">{{ $item }}</li>
        @endforeach
    </ul>

    <input wire:model="todo">
</form>

```

Render awal dari komponen ini akan menghasilkan HTML berikut:

```html
<form wire:submit="add">
    <ul>
        <li>first</li>
        <li>second</li>
    </ul>

    <input wire:model="todo">
</form>

```

Sekarang, bayangkan Anda mengetik "third" ke dalam bidang input dan menekan tombol `[Enter]`. HTML baru yang di-render adalah:

```html
<form wire:submit="add">
    <ul>
        <li>first</li>
        <li>second</li>
        <li>third</li> </ul>

    <input wire:model="todo">
</form>

```

Saat Livewire memproses pembaruan komponen, ia melakukan *morph* pada DOM asli menjadi HTML yang baru di-render. Livewire menelusuri kedua pohon (tree) HTML secara bersamaan. Saat menemui setiap elemen di kedua pohon tersebut, ia membandingkannya untuk melihat adanya perubahan, penambahan, atau penghapusan. Jika terdeteksi, ia akan melakukan perubahan yang sesuai secara presisi.

---

## Kekurangan morphing

Berikut adalah skenario di mana algoritma *morphing* gagal mengidentifikasi perubahan pohon HTML dengan benar sehingga menyebabkan masalah pada aplikasi Anda.

### Menyisipkan elemen di tengah (Intermediate elements)

Perhatikan template Blade Livewire berikut untuk komponen `CreatePost`:

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>

    @if ($errors->has('title'))
        <div>{{ $errors->first('title') }}</div>
    @endif

    <div>
        <button>Save</button>
    </div>
</form>

```

Jika pengguna mencoba mengirim formulir tetapi mengalami error validasi, masalah berikut akan terjadi:

* Livewire menemui `<div>` pertama di kedua pohon. Keduanya sama, jadi ia lanjut.
* Livewire menemui `<div>` kedua di kedua pohon dan menganggap keduanya adalah `<div>` yang sama, hanya saja kontennya berubah. Jadi, alih-alih menyisipkan pesan error sebagai elemen baru, ia **mengubah `<button>` menjadi pesan error**.
* Setelah salah memodifikasi elemen tersebut, Livewire menyadari ada elemen tambahan di akhir perbandingan. Ia kemudian membuat dan menambahkan elemen baru setelah elemen sebelumnya.
* Akibatnya, elemen yang seharusnya hanya berpindah posisi justru dihapus lalu dibuat ulang.

Skenario ini adalah akar dari hampir semua bug terkait *morphing*. Dampak spesifiknya meliputi:

* *Event listeners* dan *state* elemen hilang.
* *Event listeners* tertukar pada elemen yang salah.
* Komponen Livewire bisa ter-reset atau terduplikasi.
* Komponen dan *state* Alpine bisa hilang atau salah tempat.

---

## Solusi Livewire

Livewire telah bekerja keras untuk memitigasi masalah ini menggunakan pendekatan berikut:

### Internal look-ahead

Livewire memiliki langkah tambahan dalam algoritma *morphing*-nya yang memeriksa elemen-elemen berikutnya beserta kontennya sebelum mengubah sebuah elemen. Ini mencegah skenario di atas terjadi di banyak kasus.

### Injecting morph markers

Di sisi backend, Livewire secara otomatis mendeteksi kondisional di dalam template Blade dan membungkusnya dengan penanda komentar HTML (*morph markers*) yang digunakan JavaScript Livewire sebagai panduan saat melakukan *morphing*.

```blade
<form wire:submit="save">
    @if ($errors->has('title'))
        <div>Error: {{ $errors->first('title') }}</div>
    @endif
    </form>

```

Fitur ini sangat bermanfaat, tetapi karena menggunakan regex untuk parsing template, terkadang ia gagal mendeteksi kondisional dengan benar. Jika fitur ini justru mengganggu, Anda dapat menonaktifkannya di `config/livewire.php`:

```php
'inject_morph_markers' => false,

```

### Membungkus kondisional (Wrapping)

Cara paling andal untuk menghindari masalah *morphing* adalah membungkus kondisional dan loop di dalam elemen mereka sendiri yang selalu ada (*persistent*).

```blade
<form wire:submit="save">
    <div> @if ($errors->has('title'))
            <div>{{ $errors->first('title') }}</div>
        @endif
    </div> </form>

```

Dengan membungkus kondisional dalam elemen yang menetap, Livewire akan melakukan *morph* pada pohon HTML dengan benar.

### Melewati morphing (Bypassing)

Jika Anda perlu melewati proses *morphing* sepenuhnya untuk suatu elemen, Anda dapat menggunakan [wire:replace](https://www.google.com/search?q=/docs/4.x/wire-replace) untuk memerintahkan Livewire mengganti seluruh anak elemen tersebut alih-alih mencoba melakukan *morph*.

---

## Lihat juga

* **[Hydration](https://www.google.com/search?q=/docs/4.x/hydration)** — Memahami siklus hidup permintaan Livewire.
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Bagaimana komponen di-render dan diperbarui.
* **[wire:replace](https://www.google.com/search?q=/docs/4.x/wire-replace)** — Melewati *morphing* untuk elemen spesifik.
