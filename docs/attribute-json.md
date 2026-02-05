Atribut `#[Json]` menandai sebuah **action** sebagai endpoint JSON, yang mengembalikan data secara langsung ke JavaScript. Kesalahan validasi (*validation errors*) akan memicu penolakan *promise* (*promise rejection*) dengan data kesalahan yang terstruktur. Ini sangat ideal untuk **actions** yang dikonsumsi oleh JavaScript alih-alih di-*render* di Blade.

## Penggunaan dasar

Terapkan atribut `#[Json]` pada metode **action** apa pun yang mengembalikan data untuk dikonsumsi JavaScript:

```php
<?php // resources/views/components/⚡search.blade.php

use Livewire\Attributes\Json;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Json] // [tl! highlight]
    public function search($query)
    {
        return Post::where('title', 'like', "%{$query}%")
            ->limit(10)
            ->get();
    }
};

```

```blade
<div x-data="{ query: '', posts: [] }">
    <input
        type="text"
        x-model="query"
        x-on:input.debounce="$wire.search(query).then(data => posts = data)"
    >

    <ul>
        <template x-for="post in posts">
            <li x-text="post.title"></li>
        </template>
    </ul>
</div>

```

Metode `search()` mengembalikan postingan langsung ke Alpine, di mana data tersebut disimpan dalam array `posts` dan di-*render* di sisi klien (*client-side*).

---

## Menangani respons (Handling responses)

Metode JSON akan melakukan *resolve* dengan nilai balik jika berhasil, dan melakukan *reject* jika terjadi kegagalan validasi:

**Saat berhasil:**

```js
let data = await $wire.search('query')
// data = [ { id: 1, title: '...' }, ...]

```

**Saat gagal validasi:**

```js
try {
    let data = await $wire.save()
} catch (e) {
    // e.status = 422
    // e.errors = { name: ['The name field is required.'] }
}

```

Atau menggunakan `.catch()`:

```js
$wire.save()
    .then(data => {
        // Tangani keberhasilan
        console.log(data)
    })
    .catch(e => {
        if (e.status === 422) {
            // Tangani kesalahan validasi
            console.log(e.errors)
        }
    })

```

---

## Struktur Error Rejection

Ketika sebuah *promise* ditolak (*rejected*), objek *error* memiliki struktur sebagai berikut:

```js
{
    status: 422,    // Kode status HTTP (422 untuk kesalahan validasi)
    body: null,     // Body respons mentah (null untuk kesalahan validasi)
    json: null,     // JSON yang di-parse (null untuk kesalahan validasi)
    errors: {...}   // Objek kesalahan validasi
}

```

Untuk kesalahan HTTP (500, dll.), strukturnya sama tetapi dengan data respons yang sebenarnya:

```js
{
    status: 500,
    body: '<html>...</html>',
    json: null,
    errors: null
}

```

---

## Perilaku (Behavior)

Atribut `#[Json]` secara otomatis menerapkan dua perilaku:

1. **Melewatkan proses render** — **Component** tidak akan melakukan *re-render* setelah **action** selesai, karena respons dikonsumsi oleh JavaScript.
2. **Berjalan secara asinkron** — **Action** dieksekusi secara paralel tanpa memblokir *requests* lainnya.

Perilaku ini sesuai dengan apa yang Anda harapkan dari endpoint bergaya API.

---

## Kapan harus menggunakan

Gunakan `#[Json]` saat:

* **Membangun pencarian dinamis/autocomplete** — Mengambil hasil untuk daftar *dropdown* atau saran.
* **Memuat data ke dalam JavaScript** — Mengisi grafik (*charts*), peta, atau UI lain yang dikendalikan JS.
* **Mengirim formulir dengan penanganan JS** — Ketika Anda ingin menangani status sukses/error di dalam JavaScript.
* **Integrasi dengan library pihak ketiga** — Menyediakan data ke pustaka yang mengelola *rendering*-nya sendiri.

> [!warning] Kesalahan validasi terisolasi
> Kesalahan validasi dari metode JSON hanya dikembalikan melalui *promise rejection*. Mereka tidak akan muncul di `$wire.$errors` atau tas kesalahan (*error bag*) milik **component**. Ini disengaja—metode JSON bersifat mandiri dan tidak memengaruhi status tampilan **component**.

---

## Lihat juga

* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Pelajari tentang memanggil metode dan menerima nilai balik.
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Validasi sisi server untuk Livewire components.
* **[Async Attribute](https://www.google.com/search?q=/docs/4.x/attribute-async)** — Menjalankan actions secara paralel tanpa memblokir.
* **[Renderless Attribute](https://www.google.com/search?q=/docs/4.x/attribute-renderless)** — Melewatkan re-rendering setelah sebuah action.
