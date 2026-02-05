Atribut `#[Locked]` mencegah properti dimodifikasi di sisi klien (*client-side*), melindungi data sensitif seperti ID model dari manipulasi oleh pengguna.

## Penggunaan dasar

Terapkan atribut `#[Locked]` pada properti publik apa pun yang tidak boleh diubah dari frontend:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Locked;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Locked] // [tl! highlight]
    public $postId;

    public function mount($id)
    {
        $this->postId = $id;
    }

    public function delete()
    {
        Post::find($this->postId)->delete();

        return redirect('/posts');
    }
};

```

Jika pengguna mencoba memodifikasi properti yang terkunci melalui *browser DevTools* atau dengan memanipulasi *requests*, Livewire akan melempar pengecualian (*exception*) dan mencegah **action** tersebut dieksekusi.

> [!warning] Modifikasi di backend tetap diizinkan
> Properti dengan atribut `#[Locked]` masih dapat diubah di dalam kode PHP **component** Anda. Penguncian ini hanya mencegah manipulasi dari sisi klien. Berhati-hatilah untuk tidak memasukkan input pengguna yang tidak tepercaya ke properti yang terkunci di dalam metode Anda sendiri.

---

## Kapan harus menggunakan

Gunakan `#[Locked]` saat Anda perlu:

* Menyimpan ID model yang tidak boleh diubah oleh pengguna.
* Mempertahankan data sensitif terkait otorisasi sepanjang siklus hidup **component**.
* Melindungi properti publik apa pun yang berfungsi sebagai batasan keamanan (*security boundary*).

> [!tip] Properti model sudah aman secara default
> Jika Anda menyimpan model Eloquent dalam properti publik, Livewire secara otomatis memastikan ID-nya tidak dimanipulasi—atribut `#[Locked]` tidak diperlukan:
> ```php
> public Post $post; // Sudah terlindungi secara otomatis [tl! highlight]
> 
> ```
> 
> 

---

## Mengapa tidak menggunakan protected properties?

Anda mungkin bertanya-tanya mengapa tidak menggunakan properti `protected` saja untuk data sensitif.

Ingat, Livewire hanya mempertahankan properti `public` di antara *requests*. Properti `protected` berfungsi dengan baik untuk nilai statis yang ditulis langsung (*hard-coded*), tetapi data apa pun yang perlu disimpan saat *runtime* harus menggunakan properti `public` agar tetap tersimpan dengan benar di antara *requests*.

Di sinilah `#[Locked]` menjadi sangat penting: atribut ini memberikan kemampuan persistensi properti publik dengan perlindungan terhadap manipulasi di sisi klien.

---

## Bisakah Livewire melakukan ini secara otomatis?

Dalam dunia yang sempurna, Livewire akan mengunci properti secara default dan hanya mengizinkan modifikasi ketika `wire:model` digunakan pada properti tersebut.

Sayangnya, hal itu mengharuskan Livewire untuk membedah (*parse*) semua template Blade Anda untuk memahami apakah sebuah properti dimodifikasi oleh `wire:model` atau API serupa.

Hal ini tidak hanya akan menambah beban teknis dan performa, tetapi juga mustahil untuk mendeteksi jika sebuah properti diubah oleh sesuatu seperti Alpine atau JavaScript kustom lainnya.

Oleh karena itu, Livewire akan tetap membiarkan properti publik dapat diubah secara bebas secara default dan memberikan alat bagi pengembang untuk menguncinya sesuai kebutuhan.
