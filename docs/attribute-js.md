Atribut `#[Js]` menandai metode yang mengembalikan kode JavaScript untuk dieksekusi di sisi klien (*client-side*). Metode yang ditandai dengan `#[Js]` dapat dipanggil langsung dari template Anda tanpa melakukan *server request*.

## Penggunaan dasar

Terapkan atribut `#[Js]` pada metode yang mengembalikan ekspresi JavaScript:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Attributes\Js;
use Livewire\Component;

new class extends Component {
    public $title = '';
    public $content = '';

    #[Js] // [tl! highlight:start]
    public function resetForm()
    {
        return <<<'JS'
            $wire.title = ''
            $wire.content = ''
        JS;
    } // [tl! highlight:end]
};

```

```blade
<form wire:submit="save">
    <input wire:model="title" placeholder="Title">
    <textarea wire:model="content" placeholder="Content"></textarea>

    <button type="submit">Save</button>
    <button type="button" @click="$wire.resetForm()">Reset</button> </form>

```

Ketika `$wire.resetForm()` dipanggil, JavaScript akan dieksekusi langsung di browser — tidak terjadi *server round-trip*.

## Mengeksekusi JavaScript setelah server actions

Jika Anda perlu mengeksekusi JavaScript **setelah sebuah server action selesai**, gunakan metode `js()` sebagai gantinya:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public function save()
    {
        Post::create(['title' => $this->title]);

        $this->js("alert('Post saved successfully!')"); // [tl! highlight]
    }
};

```

Metode `js()` akan memasukkan JavaScript ke dalam antrean untuk dieksekusi saat respons server tiba.

## Mengakses $wire

Anda dapat mengakses objek `$wire` milik komponen di dalam ekspresi JavaScript:

```php
#[Js]
public function resetForm()
{
    return <<<'JS'
        $wire.title = ''
        $wire.content = ''
    JS;
}

```

## Kapan harus menggunakan

Gunakan `#[Js]` saat Anda perlu:

* Mengosongkan atau mereset bidang formulir tanpa beban server (*server overhead*)
* Memicu animasi atau transisi JavaScript
* Memperbarui *state* di sisi klien tanpa me-*render* ulang
* Mengeksekusi logika JavaScript yang dapat digunakan kembali dari berbagai tempat
* Berintegrasi dengan pustaka JavaScript pihak ketiga

## JavaScript actions vs #[Js] methods

Ada perbedaan penting yang perlu diperhatikan:

* **`#[Js]` methods** didefinisikan di PHP dan mengembalikan kode JavaScript. Metode ini dipanggil via `$wire.methodName()` tanpa melakukan *server request*.
* **JavaScript actions** (`$js.methodName`) didefinisikan sepenuhnya di JavaScript menggunakan blok `@script`.

Kedua pendekatan tersebut mengeksekusi JavaScript di klien tanpa *server round-trip*. Perbedaannya terletak pada di mana kode JavaScript tersebut didefinisikan.

```php
<?php // resources/views/components/⚡example.blade.php

use Livewire\Attributes\Js;
use Livewire\Component;

new class extends Component {
    public $count = 0;

    // JavaScript didefinisikan di PHP
    #[Js]
    public function showCount()
    {
        return "alert('Count is: {$this->count}')";
    }
};

```

```blade
<div>
    <button @click="$wire.showCount()">Show Count (dari PHP)</button>
    <button @click="$js.incrementLocal()">Increment Local (dari JS)</button>
</div>

@script
<script>
    // JavaScript didefinisikan di JavaScript
    $js('incrementLocal', () => {
        console.log('Tidak ada server request yang dilakukan')
    })
</script>
@endscript

```

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut tentang integrasi JavaScript di Livewire, lihat:

* [Dokumentasi JavaScript](https://www.google.com/search?q=/docs/4.x/javascript)
* [Dokumentasi JavaScript actions](https://www.google.com/search?q=/docs/4.x/actions%23javascript-actions)
