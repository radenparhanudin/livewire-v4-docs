Seperti banyak framework berbasis komponen lainnya, komponen Livewire dapat disarangkan (*nestable*) — artinya satu komponen dapat me-render beberapa komponen di dalam dirinya sendiri.

Namun, karena sistem penyarangan Livewire dibangun secara berbeda dari framework lain, ada implikasi dan batasan tertentu yang penting untuk disadari.

> [!tip] Pastikan Anda memahami hydration terlebih dahulu
> Sebelum mempelajari lebih lanjut tentang sistem penyarangan Livewire, sangat membantu jika Anda memahami sepenuhnya bagaimana Livewire melakukan *hydrate* pada komponen. Anda dapat mempelajari lebih lanjut dengan membaca [dokumentasi hydration](https://www.google.com/search?q=/docs/4.x/hydration).

## Setiap komponen bersifat independen {#every-component-is-an-island}

Di Livewire, setiap komponen pada sebuah halaman melacak statusnya dan melakukan pembaruan secara independen dari komponen lainnya.

Sebagai contoh, perhatikan komponen `Posts` dan komponen `ShowPost` yang disarangkan di dalamnya:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class Posts extends Component
{
    public $postLimit = 2;

    public function render()
    {
        return view('livewire.posts', [
            'posts' => Auth::user()->posts()
                ->limit($this->postLimit)->get(),
        ]);
    }
}

```

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :wire:key="$post->id">
    @endforeach
</div>

```

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function render()
    {
        return view('livewire.show-post');
    }
}

```

```blade
<div>
    <h1>{{ $post->title }}</h1>

    <p>{{ $post->content }}</p>

    <button wire:click="$refresh">Refresh post</button>
</div>

```

Berikut adalah tampilan HTML untuk seluruh pohon komponen pada saat pemuatan halaman awal:

```html
<div wire:id="123" wire:snapshot="...">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456" wire:snapshot="...">
        <h1>The first post</h1>
        <p>Post content</p>
        <button wire:click="$refresh">Refresh post</button>
    </div>

    <div wire:id="789" wire:snapshot="...">
        <h1>The second post</h1>
        <p>Post content</p>
        <button wire:click="$refresh">Refresh post</button>
    </div>
</div>

```

Perhatikan bahwa komponen induk berisi template yang di-render miliknya sendiri sekaligus template yang di-render dari semua komponen yang disarangkan di dalamnya.

Karena setiap komponen bersifat independen, masing-masing memiliki ID dan *snapshot* mereka sendiri (`wire:id` dan `wire:snapshot`) yang tertanam dalam HTML agar inti JavaScript Livewire dapat mengekstrak dan melacaknya.

Mari kita pertimbangkan beberapa skenario pembaruan yang berbeda untuk melihat perbedaan cara Livewire menangani berbagai level penyarangan.

### Memperbarui komponen anak (Child)

Jika Anda mengeklik tombol "Refresh post" di salah satu komponen anak `show-post`, berikut adalah apa yang akan dikirim ke server:

```js
{
    memo: { name: 'show-post', id: '456' },

    state: { ... },
}

```

Dan berikut adalah HTML yang akan dikirimkan kembali:

```html
<div wire:id="456">
    <h1>The first post</h1>

    <p>Post content</p>

    <button wire:click="$refresh">Refresh post</button>
</div>

```

Hal penting yang perlu dicatat di sini adalah ketika pembaruan dipicu pada komponen anak, hanya data komponen tersebut yang dikirim ke server, dan hanya komponen tersebut yang di-render ulang.

Sekarang mari kita lihat skenario yang kurang intuitif: memperbarui komponen induk (parent).

### Memperbarui komponen induk (Parent)

Sebagai pengingat, berikut adalah template Blade dari komponen induk `Posts`:

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :wire:key="$post->id">
    @endforeach
</div>

```

Jika pengguna mengubah nilai "Post Limit" dari `2` menjadi `1`, pembaruan hanya akan dipicu pada komponen induk saja.

Berikut adalah contoh bagaimana tampilan *payload* permintaan tersebut:

```js
{
    updates: { postLimit: 1 },

    snapshot: {
        memo: { name: 'posts', id: '123' },

        state: { postLimit: 2, ... },
    },
}

```

Seperti yang Anda lihat, hanya *snapshot* untuk komponen induk `Posts` yang dikirimkan ke server.

Pertanyaan penting yang mungkin Anda tanyakan adalah: apa yang terjadi ketika komponen induk me-render ulang dan menemui komponen anak `show-post`? Bagaimana ia akan me-render ulang anak-anak tersebut jika *snapshot* mereka tidak disertakan dalam *payload* permintaan?

Jawabannya adalah: mereka tidak akan di-render ulang.

Ketika Livewire me-render komponen `Posts`, ia akan me-render *placeholders* (penampung) untuk setiap komponen anak yang ditemuinya.

Berikut adalah contoh HTML hasil render untuk komponen `Posts` setelah pembaruan di atas:

```html
<div wire:id="123">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456"></div>
</div>

```

Hanya satu anak yang di-render karena `postLimit` diperbarui menjadi `1`. Namun, Anda juga akan menyadari bahwa alih-alih komponen anak yang lengkap, hanya ada `<div></div>` kosong dengan atribut `wire:id` yang cocok.

Ketika HTML ini diterima di frontend, Livewire akan melakukan *morph* pada HTML lama komponen ini menjadi HTML baru tersebut, namun secara cerdas akan melewati setiap *placeholder* komponen anak.

Efeknya adalah, setelah proses *morphing*, konten DOM akhir dari komponen induk `Posts` akan menjadi seperti berikut:

```html
<div wire:id="123">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456">
        <h1>The first post</h1>

        <p>Post content</p>

        <button wire:click="$refresh">Refresh post</button>
    </div>
</div>

```

---

## Implikasi Performa

Arsitektur komponen independen Livewire dapat memiliki implikasi positif maupun negatif bagi aplikasi Anda.

Keuntungan dari arsitektur ini adalah memungkinkan Anda untuk mengisolasi bagian aplikasi yang "mahal". Sebagai contoh, Anda dapat mengarantina kueri database yang lambat ke komponen independennya sendiri, dan beban performanya tidak akan memengaruhi bagian halaman lainnya.

Namun, kelemahan terbesar dari pendekatan ini adalah karena komponen sepenuhnya terpisah, komunikasi/ketergantungan antar komponen menjadi lebih sulit.

Sebagai contoh, jika Anda memiliki properti yang diteruskan dari komponen induk `Posts` di atas ke komponen `ShowPost` yang disarangkan, properti tersebut tidak akan bersifat "reaktif". Karena setiap komponen independen, jika sebuah permintaan ke komponen induk mengubah nilai properti yang diteruskan ke `ShowPost`, nilai tersebut tidak akan diperbarui di dalam `ShowPost`.

Livewire telah mengatasi sejumlah hambatan ini dan menyediakan API khusus untuk skenario tersebut seperti: [Reactive properties](https://www.google.com/search?q=/docs/4.x/nesting%23reactive-props), [Modelable components](https://www.google.com/search?q=/docs/4.x/nesting%23binding-to-child-data-using-wiremodel), dan [objek `$parent](https://www.google.com/search?q=/docs/4.x/nesting%23directly-accessing-the-parent-from-the-child)`.

Berbekal pengetahuan tentang bagaimana komponen Livewire yang disarangkan beroperasi, Anda akan dapat membuat keputusan yang lebih tepat tentang kapan dan bagaimana menyarangkan komponen dalam aplikasi Anda.
