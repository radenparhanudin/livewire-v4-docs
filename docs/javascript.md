## Menggunakan JavaScript dalam komponen Livewire

Livewire dan Alpine menyediakan banyak utilitas untuk membangun komponen dinamis langsung di dalam HTML Anda, namun ada kalanya akan sangat membantu jika Anda keluar dari HTML dan mengeksekusi JavaScript murni untuk komponen Anda.

> [!warning] Class-based components memerlukan direktif @@script
> Contoh-contoh di halaman ini menggunakan tag `<script>` biasa, yang berfungsi untuk komponen **single-file** dan **multi-file**. Jika Anda menggunakan **class-based components** (di mana view Blade berada di file terpisah dari class PHP), Anda harus membungkus tag script Anda dengan direktif `@@script`:
> ```blade
> @@script
> <script>
>     // JavaScript Anda di sini...
> </script>
> @@endscript
> 
> ```
> 
> 
> Ini memberitahu Livewire untuk menangani waktu eksekusi dengan benar bagi class-based components.

### Mengeksekusi script

Anda dapat menambahkan tag `<script>` langsung di dalam template komponen Anda untuk mengeksekusi JavaScript saat komponen dimuat.

Karena script ini ditangani oleh Livewire, mereka dieksekusi pada waktu yang tepat—setelah halaman dimuat, tetapi sebelum komponen Livewire di-render. Ini berarti Anda tidak perlu lagi membungkus script Anda dalam `document.addEventListener('...')` agar dimuat dengan benar.

Ini juga berarti bahwa komponen Livewire yang dimuat secara *lazy* atau kondisional tetap dapat mengeksekusi JavaScript setelah halaman diinisialisasi.

```blade
<div>
    ...
</div>

<script>
    // Javascript ini akan dieksekusi setiap kali komponen ini dimuat ke halaman...
</script>

```

Berikut adalah contoh yang lebih lengkap di mana Anda dapat mendaftarkan JavaScript action yang digunakan dalam komponen Livewire Anda.

```blade
<div>
    <button wire:click="$js.increment">+</button>
</div>

<script>
    this.$js.increment = () => {
        console.log('increment')
    }
</script>

```

Untuk mempelajari lebih lanjut tentang JavaScript actions, [kunjungi dokumentasi actions](https://www.google.com/search?q=/docs/4.x/actions%23javascript-actions).

### Menggunakan `$wire` dari script

Saat Anda menambahkan tag `<script>` di dalam komponen, Anda secara otomatis memiliki akses ke objek `$wire` milik komponen Livewire tersebut.

Berikut adalah contoh penggunaan `setInterval` sederhana untuk me-refresh komponen setiap 2 detik (Anda dapat dengan mudah melakukannya dengan [`wire:poll`](https://www.google.com/search?q=/docs/4.x/wire-poll), tetapi ini adalah cara simpel untuk mendemonstrasikannya):

```blade
<script>
    setInterval(() => {
        $wire.$refresh()
    }, 2000)
</script>

```

---

## Objek `$wire`

Objek `$wire` adalah antarmuka JavaScript Anda menuju komponen Livewire. Ia menyediakan akses ke properti komponen, metode, dan utilitas untuk berinteraksi dengan server.

Di dalam script komponen, Anda dapat menggunakan `$wire` secara langsung. Berikut adalah metode-metode paling penting yang akan Anda gunakan:

```js
// Mengakses dan memodifikasi properti
$wire.count
$wire.count = 5
$wire.$set('count', 5)

// Memanggil metode komponen
$wire.save()
$wire.delete(postId)

// Me-refresh komponen
$wire.$refresh()

// Mengirimkan event (Dispatch)
$wire.$dispatch('post-created', { postId: 2 })

// Mendengarkan event (Listen)
$wire.$on('post-created', (event) => {
    console.log(event.postId)
})

// Mengakses elemen root
$wire.$el.querySelector('.modal')

```

> [!tip] Referensi lengkap $wire
> Untuk daftar komprehensif semua metode dan properti `$wire\`, lihat [referensi $wire](https://www.google.com/search?q=%23the-wire-object) di bagian bawah halaman ini.

---

## Memuat aset (Loading assets)

Tag `<script>` komponen berguna untuk mengeksekusi sedikit JavaScript setiap kali komponen Livewire dimuat, namun ada kalanya Anda mungkin ingin memuat seluruh aset script dan style ke halaman bersamaan dengan komponen tersebut.

Berikut adalah contoh penggunaan `@assets` untuk memuat library date picker bernama [Pikaday](https://github.com/Pikaday/Pikaday) dan menginisialisasinya di dalam komponen Anda:

```blade
<div>
    <input type="text" data-picker>
</div>

@assets
<script src="https://cdn.jsdelivr.net/npm/pikaday/pikaday.js" defer></script>
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/pikaday/css/pikaday.css">
@endassets

<script>
    new Pikaday({ field: $wire.$el.querySelector('[data-picker]') });
</script>

```

Ketika komponen ini dimuat, Livewire akan memastikan semua `@assets` dimuat di halaman tersebut sebelum mengevaluasi script. Selain itu, ia memastikan `@assets` yang disediakan hanya dimuat satu kali per halaman tidak peduli berapa banyak instansi komponen ini yang ada, berbeda dengan script komponen yang akan dievaluasi untuk setiap instansi komponen di halaman.

---

## Interceptors

Cegat permintaan Livewire pada tiga level: **action** (paling granular), **message** (per-komponen), dan **request** (level HTTP).

```js
// Action interceptors - dipicu untuk setiap pemanggilan action
$wire.intercept(callback)                 // Semua action pada komponen ini
$wire.intercept('save', callback)         // Hanya action 'save'
Livewire.interceptAction(callback)        // Global (semua komponen)

// Message interceptors - dipicu untuk setiap pesan komponen
$wire.interceptMessage(callback)          // Pesan dari komponen ini
$wire.interceptMessage('save', callback)   // Hanya saat pesan berisi 'save'
Livewire.interceptMessage(callback)       // Global (semua komponen)

// Request interceptors - dipicu untuk setiap HTTP request
$wire.interceptRequest(callback)          // Request yang melibatkan komponen ini
$wire.interceptRequest('save', callback)   // Hanya saat request berisi 'save'
Livewire.interceptRequest(callback)       // Global (semua request)

```

Semua interceptor mengembalikan fungsi unsubscribe:

```js
let unsubscribe = $wire.intercept(callback)
unsubscribe() // Menghapus interceptor

```

### Action interceptors

Action interceptors adalah yang paling granular. Mereka dipicu untuk setiap pemanggilan metode pada sebuah komponen.

```js
$wire.intercept(({ action, onSend, onCancel, onSuccess, onError, onFailure, onFinish }) => {
    // action.name        - Nama metode ('save', '$refresh', dll.)
    // action.params      - Parameter metode
    // action.component   - Instansi komponen
    // action.cancel()    - Membatalkan action ini

    onSend(({ call }) => {
        // call: { method, params, metadata }
    })

    onCancel(() => {})

    onSuccess((result) => {
        // result: Nilai kembalian dari metode PHP
    })

    onError(({ response, body, preventDefault }) => {
        preventDefault() // Mencegah modal error muncul
    })

    onFailure(({ error }) => {
        // error: Kesalahan jaringan
    })

    onFinish(() => {
        // Berjalan setelah DOM morph selesai (atau saat error/batal)
    })
})

```

### Message interceptors

Message interceptors dipicu untuk setiap pembaruan komponen. Sebuah pesan (*message*) berisi satu atau lebih action.

```js
$wire.interceptMessage(({ message, cancel, onSend, onCancel, onSuccess, onError, onFailure, onStream, onFinish }) => {
    // message.component  - Instansi komponen
    // message.actions    - Kumpulan action dalam pesan ini
    // cancel()           - Membatalkan pesan ini

    onSend(({ payload }) => {
        // payload: { snapshot, updates, calls }
    })

    onCancel(() => {})

    onSuccess(({ payload, onSync, onEffect, onMorph, onRender }) => {
        // payload: { snapshot, effects }

        onSync(() => {})    // Setelah status disinkronkan
        onEffect(() => {})  // Setelah efek diproses
        onMorph(async () => {})   // Setelah DOM di-morph (harus async)
        onRender(() => {})  // Setelah render selesai
    })

    onError(({ response, body, preventDefault }) => {
        preventDefault() // Mencegah modal error muncul
    })

    onFailure(({ error }) => {})

    onStream(({ json }) => {
        // json: Chunk stream yang sudah diparsing
    })

    onFinish(() => {
        // Berjalan setelah DOM morph selesai (atau saat error/batal)
    })
})

```

#### Timing

Urutan eksekusi **hook** untuk permintaan yang berhasil:

1. `onSuccess` - Segera setelah respons server diterima.
2. `onSync` - Setelah **state** digabungkan (*merged*).
3. `onEffect` - Setelah efek-efek diproses.
4. `onMorph` - Setelah DOM di-**morph**.
5. `onFinish` - Setelah proses **morph** selesai sepenuhnya.
6. `onRender` - Di dalam `requestAnimationFrame` (setelah proses *paint* browser).

**Action promises** (`.then()`) diselesaikan pada waktu yang sama dengan `onFinish` (setelah **morph**).

### Request interceptors

**Request interceptors** dipicu untuk setiap permintaan HTTP. Satu permintaan dapat berisi pesan dari beberapa komponen sekaligus.

```js
$wire.interceptRequest(({ request, onSend, onCancel, onSuccess, onError, onFailure, onResponse, onParsed, onStream, onRedirect, onDump, onFinish }) => {
    // request.messages   - Kumpulan pesan dalam permintaan ini
    // request.cancel()   - Membatalkan permintaan ini

    onSend(({ responsePromise }) => {})

    onCancel(() => {})

    onResponse(({ response }) => {
        // response: Fetch Response (sebelum body dibaca)
    })

    onParsed(({ response, body }) => {
        // body: Isi respons sebagai string
    })

    onSuccess(({ response, body, json }) => {})

    onError(({ response, body, preventDefault }) => {
        preventDefault() // Mencegah modal error muncul
    })

    onFailure(({ error }) => {})

    onStream(({ response }) => {})

    onRedirect(({ url, preventDefault }) => {
        preventDefault() // Mencegah pengalihan (redirect)
    })

    onDump(({ html, preventDefault }) => {
        preventDefault() // Mencegah modal dump muncul
    })

    onFinish(() => {})
})

```

### Contoh Penggunaan

**Loading state untuk sebuah komponen:**

```blade
<script>
    $wire.intercept(({ onSend, onFinish }) => {
        onSend(() => $wire.$el.classList.add('opacity-50'))
        onFinish(() => $wire.$el.classList.remove('opacity-50'))
    })
</script>

```

**Konfirmasi sebelum hapus (Confirm before delete):**

```blade
<script>
    $wire.intercept('delete', ({ action }) => {
        if (!confirm('Apakah Anda yakin?')) {
            action.cancel()
        }
    })
</script>

```

**Penanganan sesi kedaluwarsa secara global:**

```js
Livewire.interceptRequest(({ onError }) => {
    onError(({ response, preventDefault }) => {
        if (response.status === 419) {
            preventDefault()
            if (confirm('Sesi kedaluwarsa. Muat ulang halaman?')) {
                window.location.reload()
            }
        }
    })
})

```

**Notifikasi sukses khusus untuk action tertentu:**

```blade
<script>
    $wire.intercept('save', ({ onSuccess, onError }) => {
        onSuccess(() => showToast('Berhasil disimpan!'))
        onError(() => showToast('Gagal menyimpan', 'error'))
    })
</script>

```

---

## Global Livewire events

Livewire mengirimkan dua **browser events** yang berguna bagi Anda untuk mendaftarkan titik ekstensi kustom dari script eksternal:

```html
<script>
    document.addEventListener('livewire:init', () => {
        // Berjalan setelah Livewire dimuat tetapi sebelum diinisialisasi 
        // di halaman...
    })

    document.addEventListener('livewire:initialized', () => {
        // Berjalan segera setelah Livewire selesai menginisialisasi
        // di halaman...
    })
</script>

```

> [!info]
> Sering kali lebih baik untuk mendaftarkan [custom directives](https://www.google.com/search?q=%23registering-custom-directives) atau [lifecycle hooks](https://www.google.com/search?q=%23javascript-hooks) di dalam `livewire:init` agar tersedia sebelum Livewire mulai menginisialisasi halaman.

---

## Objek global `Livewire`

Objek global Livewire adalah titik awal terbaik untuk berinteraksi dengan Livewire dari script eksternal. Anda dapat mengakses objek JavaScript `Livewire` global pada `window` dari mana saja di dalam kode sisi klien Anda.

Sangat disarankan untuk menggunakan `window.Livewire` di dalam *event listener* `livewire:init`.

### Mengakses komponen

Anda dapat menggunakan metode berikut untuk mengakses komponen Livewire tertentu yang dimuat pada halaman saat ini:

```js
// Mengambil objek $wire untuk komponen pertama di halaman...
let component = Livewire.first()

// Mengambil objek $wire komponen tertentu berdasarkan ID-nya...
let component = Livewire.find(id)

// Mengambil array berisi objek $wire komponen berdasarkan namanya...
let components = Livewire.getByName(name)

// Mengambil objek $wire untuk setiap komponen di halaman...
let components = Livewire.all()

```

> [!info]
> Masing-masing metode ini mengembalikan objek `$wire` yang mewakili **state** komponen di Livewire. Anda dapat mempelajari lebih lanjut tentang objek-objek ini di [dokumentasi $wire](https://www.google.com/search?q=%23the-wire-object).

### Berinteraksi dengan event

Selain mengirim dan mendengarkan **event** dari masing-masing komponen di PHP, objek `Livewire` global memungkinkan Anda untuk berinteraksi dengan [sistem event Livewire](https://www.google.com/search?q=/docs/4.x/events) dari mana saja di aplikasi Anda:

```js
// Mengirim event ke komponen Livewire mana pun yang mendengarkan...
Livewire.dispatch('post-created', { postId: 2 })

// Mengirim event ke komponen Livewire tertentu berdasarkan namanya...
Livewire.dispatchTo('dashboard', 'post-created', { postId: 2 })

// Mendengarkan event yang dikirim dari komponen Livewire...
Livewire.on('post-created', ({ postId }) => {
    // ...
})

```

Dalam skenario tertentu, Anda mungkin perlu membatalkan pendaftaran **global Livewire events**. Misalnya, saat bekerja dengan komponen Alpine dan `wire:navigate`, beberapa *listener* mungkin terdaftar berulang kali karena `init` dipanggil saat berpindah antar halaman. Untuk mengatasinya, gunakan fungsi `destroy` yang secara otomatis dipanggil oleh Alpine.

```js
Alpine.data('MyComponent', () => ({
    listeners: [],
    init() {
        this.listeners.push(
            Livewire.on('post-created', (options) => {
                // Lakukan sesuatu...
            })
        );
    },
    destroy() {
        // Memanggil fungsi unsubscribe untuk setiap listener
        this.listeners.forEach((listener) => {
            listener();
        });
    }
}));

```

### Menggunakan lifecycle hooks

Livewire memungkinkan Anda untuk masuk ke berbagai bagian siklus hidup globalnya menggunakan `Livewire.hook()`:

```js
// Mendaftarkan callback untuk dieksekusi pada hook internal Livewire tertentu...
Livewire.hook('component.init', ({ component, cleanup }) => {
    // ...
})

```

Informasi lebih lanjut mengenai JavaScript **hooks** Livewire dapat [ditemukan di bawah](https://www.google.com/search?q=%23javascript-hooks).

### Mendaftarkan custom directives

Livewire memungkinkan Anda untuk mendaftarkan **custom directives** menggunakan `Livewire.directive()`.

Di bawah ini adalah contoh dari direktif kustom `wire:confirm` yang menggunakan dialog `confirm()` JavaScript untuk mengonfirmasi atau membatalkan sebuah **action** sebelum dikirim ke server:

```html
<button wire:confirm="Are you sure?" wire:click="delete">Delete post</button>

```

Berikut adalah implementasi `wire:confirm` menggunakan `Livewire.directive()`:

```js
Livewire.directive('confirm', ({ el, directive, component, cleanup }) => {
    let content =  directive.expression

    // Objek "directive" memberi Anda akses ke direktif yang telah diparsing.
    // Sebagai contoh, berikut adalah nilai-nilainya untuk: wire:click.prevent="deletePost(1)"
    //
    // directive.raw = wire:click.prevent
    // directive.value = "click"
    // directive.modifiers = ['prevent']
    // directive.expression = "deletePost(1)"

    let onClick = e => {
        if (! confirm(content)) {
            e.preventDefault()
            e.stopImmediatePropagation()
        }
    }

    el.addEventListener('click', onClick, { capture: true })

    // Daftarkan kode pembersihan (cleanup) di dalam `cleanup()` untuk kasus
    // di mana komponen Livewire dihapus dari DOM saat
    // halaman masih aktif.
    cleanup(() => {
        el.removeEventListener('click', onClick)
    })
})

```

---

## JavaScript hooks

Untuk pengguna tingkat lanjut, Livewire mengekspos sistem "**hook**" internal di sisi klien. Anda dapat menggunakan **hooks** berikut untuk memperluas fungsionalitas Livewire atau mendapatkan informasi lebih lanjut tentang aplikasi Livewire Anda.

### Inisialisasi komponen

Setiap kali komponen baru ditemukan oleh Livewire — baik pada saat pemuatan halaman awal atau setelahnya — event `component.init` akan dipicu. Anda dapat masuk ke `component.init` untuk mencegat atau menginisialisasi apa pun yang terkait dengan komponen baru tersebut:

```js
Livewire.hook('component.init', ({ component, cleanup }) => {
    //
})

```

Untuk informasi lebih lanjut, silakan merujuk pada [dokumentasi objek komponen](https://www.google.com/search?q=%23the-component-object).

### Inisialisasi elemen DOM

Selain memicu event saat komponen baru diinisialisasi, Livewire memicu event untuk setiap elemen DOM di dalam komponen Livewire tertentu.

Ini dapat digunakan untuk menyediakan atribut HTML Livewire kustom di dalam aplikasi Anda:

```js
Livewire.hook('element.init', ({ component, el }) => {
    //
})

```

### DOM Morph hooks

Selama fase **DOM morphing** — yang terjadi setelah Livewire menyelesaikan siklus komunikasi jaringan — Livewire memicu serangkaian event untuk setiap elemen yang berubah (**mutated**).

```js
Livewire.hook('morph.updating',  ({ el, component, toEl, skip, childrenOnly }) => {
    //
})

Livewire.hook('morph.updated', ({ el, component }) => {
    //
})

Livewire.hook('morph.removing', ({ el, component, skip }) => {
    //
})

Livewire.hook('morph.removed', ({ el, component }) => {
    //
})

Livewire.hook('morph.adding',  ({ el, component }) => {
    //
})

Livewire.hook('morph.added',  ({ el }) => {
    //
})

```

Selain event yang dipicu per elemen, event `morph` dan `morphed` juga dipicu untuk setiap komponen Livewire:

```js
Livewire.hook('morph',  ({ el, component }) => {
    // Berjalan tepat sebelum elemen anak dalam `component` di-morph (tidak termasuk partial morphing)
})

Livewire.hook('morphed',  ({ el, component }) => {
    // Berjalan setelah semua elemen anak dalam `component` selesai di-morph (tidak termasuk partial morphing)
})

```

---

## Evaluasi JavaScript di sisi server

Selain mengeksekusi JavaScript secara langsung di komponen Anda, Anda dapat menggunakan metode `js()` untuk mengevaluasi ekspresi JavaScript dari kode PHP di sisi server.

Ini umumnya berguna untuk melakukan semacam tindak lanjut di sisi klien setelah sebuah **action** di sisi server dilakukan.

Sebagai contoh, berikut adalah komponen `post.create` yang memicu dialog alert di sisi klien setelah postingan disimpan ke database:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public $title = '';

    public function save()
    {
        // Simpan postingan ke database...

        $this->js("alert('Post saved!')");
    }
};

```

Ekspresi JavaScript `alert('Post saved!')` akan dieksekusi pada klien setelah postingan berhasil disimpan ke database di server.

Anda dapat mengakses objek `$wire` komponen saat ini di dalam ekspresi tersebut:

```php
$this->js('$wire.$refresh()');
$this->js('$wire.$dispatch("post-created", { id: ' . $post->id . ' })');

```

---

## Pola Umum (Common patterns)

Berikut adalah beberapa pola umum penggunaan JavaScript dengan Livewire dalam aplikasi dunia nyata.

### Mengintegrasikan library pihak ketiga

Banyak library JavaScript perlu diinisialisasi saat elemen ditambahkan ke halaman. Gunakan script komponen untuk menginisialisasi library saat komponen Anda dimuat:

```blade
<div>
    <div id="map" style="height: 400px;"></div>
</div>

@assets
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_KEY"></script>
@endassets

<script>
    new google.maps.Map($wire.$el.querySelector('#map'), {
        center: { lat: {{ $latitude }}, lng: {{ $longitude }} },
        zoom: 12
    });
</script>

```

### Sinkronisasi dengan localStorage

Anda dapat menyinkronkan **state** komponen dengan **localStorage** menggunakan `$watch`:

```blade
<script>
    // Muat dari localStorage saat inisialisasi
    if (localStorage.getItem('draft')) {
        $wire.content = localStorage.getItem('draft');
    }

    // Simpan ke localStorage saat nilai berubah
    $wire.$watch('content', (value) => {
        localStorage.setItem('draft', value);
    });
</script>

```

### Menggunakan direktif `@js`

Jika Anda perlu mengeluarkan data PHP untuk digunakan langsung dalam JavaScript, Anda dapat menggunakan direktif `@js`.

```blade
<script>
    let posts = @js($posts)

    // "posts" sekarang akan menjadi array JavaScript yang berisi data post dari PHP.
</script>

```

## Praktik terbaik

### Component scripts vs global scripts

**Gunakan component scripts saat:**

* JavaScript tersebut bersifat spesifik untuk fungsionalitas komponen tersebut.
* Anda memerlukan akses ke `$wire` atau data spesifik komponen.
* Kode tersebut harus dijalankan setiap kali komponen dimuat.

**Gunakan global scripts saat:**

* Mendaftarkan custom directives atau hooks.
* Menyiapkan event listeners global.
* Menginisialisasi JavaScript yang berlaku untuk seluruh aplikasi.

### Menghindari kebocoran memori (Memory leaks)

Saat menambahkan event listeners di dalam component scripts, Livewire secara otomatis membersihkannya ketika komponen dihapus. Namun, jika Anda menggunakan interceptors atau hooks global, pastikan untuk membersihkannya secara manual jika diperlukan:

```js
// Tingkat Komponen - dibersihkan secara otomatis ✓
$wire.intercept(({ onSend }) => {
    onSend(() => console.log('Mengirim...'));
});

// Tingkat Global - tetap hidup selama siklus hidup halaman
Livewire.interceptMessage(({ onSend }) => {
    onSend(() => console.log('Mengirim...'));
});

```

### Tips debugging

**Akses komponen dari konsol browser:**

```js
// Ambil komponen pertama di halaman
let $wire = Livewire.first()

// Periksa state komponen
console.log($wire.count)

// Panggil methods
$wire.increment()

```

**Pantau semua request:**

```js
Livewire.interceptRequest(({ onSend }) => {
    onSend(() => {
        console.log('Request terkirim:', Date.now());
    });
});

```

**Lihat snapshots komponen:**

```js
let component = Livewire.first().__instance()
console.log(component.snapshot)

```

### Pertimbangan performa

* Gunakan `wire:ignore` pada elemen yang tidak boleh disentuh oleh DOM morphing Livewire.
* Gunakan **debounce** pada operasi yang berat menggunakan `wire:model.debounce` atau JavaScript debouncing.
* Gunakan **lazy loading** (parameter `lazy`) untuk komponen yang tidak langsung terlihat di layar.
* Pertimbangkan menggunakan **islands** untuk wilayah terisolasi yang diperbarui secara independen.

---

## Lihat juga

* **[Styles](https://www.google.com/search?q=/docs/4.x/styles)** — Tambahkan CSS lingkup terbatas (scoped) ke komponen Anda.
* **[Alpine](https://www.google.com/search?q=/docs/4.x/alpine)** — Gunakan Alpine untuk interaktivitas di sisi klien.
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Buat JavaScript actions di dalam komponen.
* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Akses properties dari JavaScript dengan $wire.
* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — Kirim dan dengarkan events di JavaScript.

---

## Referensi

Saat memperluas sistem JavaScript Livewire, penting untuk memahami berbagai objek yang mungkin Anda temui. Berikut adalah referensi lengkap dari properti internal Livewire yang relevan.

Sebagai pengingat, pengguna Livewire rata-rata mungkin tidak pernah berinteraksi dengan bagian ini. Sebagian besar objek ini tersedia untuk sistem internal Livewire atau pengguna tingkat lanjut.

### Objek `$wire`

Misalkan terdapat komponen `Counter` generik berikut:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}

```

Livewire mengekspos representasi JavaScript dari komponen sisi server dalam bentuk objek yang biasa disebut sebagai `$wire`:

```js
let $wire = {
    // Semua public properties komponen dapat diakses langsung di $wire...
    count: 0,

    // Semua public methods diekspos dan dapat dipanggil di $wire...
    increment() { ... },

    // Akses objek `$wire` dari komponen induk jika ada...
    $parent,

    // Akses elemen DOM root dari komponen Livewire...
    $el,

    // Akses ID dari komponen Livewire saat ini...
    $id,

    // Ambil nilai properti berdasarkan nama...
    // Penggunaan: $wire.$get('count')
    $get(name) { ... },

    // Atur properti pada komponen berdasarkan nama...
    // Penggunaan: $wire.$set('count', 5)
    $set(name, value, live = true) { ... },

    // Toggle nilai dari properti boolean...
    $toggle(name, live = true) { ... },

    // Panggil method...
    // Penggunaan: $wire.$call('increment')
    $call(method, ...params) { ... },

    // Definisikan JavaScript action...
    // Penggunaan: $wire.$js('increment', () => { ... })
    // Penggunaan: $wire.$js.increment = () => { ... }
    $js(name, callback) { ... },

    // [DEPRECATED] Entangle - Anda kemungkinan besar tidak membutuhkannya.
    // Gunakan $wire secara langsung untuk mengakses properti.
    // Penggunaan: <div x-data="{ count: $wire.$entangle('count') }">
    $entangle(name, live = false) { ... },

    // Pantau perubahan nilai properti...
    // Penggunaan: Alpine.$watch('count', (value, old) => { ... })
    $watch(name, callback) { ... },

    // Batasi cakupan action berikutnya ke island tertentu...
    // Mengembalikan $wire sehingga Anda bisa melakukan chaining method call.
    // Action yang dirangkai hanya akan me-render ulang island tersebut.
    // Penggunaan: $wire.$island('revenue').$refresh()
    $island(name, options = {}) { ... },

    // Refresh komponen dengan mengirim pesan ke server 
    // untuk me-render ulang HTML dan menukarnya di halaman...
    $refresh() { ... },

    // Identik dengan `$refresh` di atas. Hanya nama yang lebih teknis...
    $commit() { ... }, // Alias untuk $refresh()

    // Mendengarkan event yang dikirim dari komponen ini atau anak-anaknya...
    // Penggunaan: $wire.$on('post-created', () => { ... })
    $on(event, callback) { ... },

    // Mendengarkan lifecycle hook yang dipicu dari komponen ini atau request...
    // Penggunaan: $wire.$hook('message.sent', () => { ... })
    $hook(name, callback) { ... },

    // Kirim event dari komponen ini...
    // Penggunaan: $wire.$dispatch('post-created', { postId: 2 })
    $dispatch(event, params = {}) { ... },

    // Kirim event ke komponen lain...
    // Penggunaan: $wire.$dispatchTo('dashboard', 'post-created', { postId: 2 })
    $dispatchTo(otherComponentName, event, params = {}) { ... },

    // Kirim event ke komponen ini sendiri dan tidak ke yang lain...
    $dispatchSelf(event, params = {}) { ... },

    // API JS untuk mengunggah file langsung ke komponen
    // tanpa melalui `wire:model`...
    $upload(
        name, // Nama properti
        file, // Objek File JavaScript
        finish = () => { ... }, // Berjalan saat unggahan selesai...
        error = () => { ... }, // Berjalan jika terjadi error saat mengunggah...
        progress = (event) => { // Berjalan saat proses unggahan berlangsung...
            event.detail.progress // Integer dari 1-100...
        },
    ) { ... },

    // API untuk mengunggah banyak file sekaligus...
    $uploadMultiple(name, files, finish, error, progress) { },

    // Hapus unggahan setelah diunggah sementara tetapi belum disimpan...
    $removeUpload(name, tmpFilename, finish, error) { ... },

    // Daftarkan action interceptor untuk instansi komponen ini
    // Penggunaan: $wire.intercept(({ action, onSend, onCancel, onSuccess, onError, onFailure, onFinish }) => { ... })
    intercept(actionOrCallback, callback) { ... },

    // Alias untuk intercept
    interceptAction(actionOrCallback, callback) { ... },

    // Daftarkan message interceptor untuk instansi komponen ini
    interceptMessage(actionOrCallback, callback) { ... },

    // Daftarkan request interceptor untuk instansi komponen ini
    interceptRequest(actionOrCallback, callback) { ... },

    // Mengambil objek "component" yang mendasarinya (internal)...
    __instance() { ... },
}

```

Anda dapat mempelajari lebih lanjut tentang `$wire` dalam [dokumentasi Livewire tentang mengakses properti di JavaScript](https://www.google.com/search?q=/docs/4.x/properties%23accessing-properties-from-javascript).

### Objek `snapshot`

Di antara setiap permintaan jaringan, Livewire melakukan serialisasi komponen PHP ke dalam sebuah objek yang dapat dikonsumsi oleh JavaScript. **Snapshot** ini digunakan untuk melakukan unserialisasi kembali komponen tersebut menjadi objek PHP, sehingga memiliki mekanisme bawaan untuk mencegah manipulasi:

```js
let snapshot = {
    // State komponen yang diserialisasi (public properties)...
    data: { count: 0 },

    // Informasi jangka panjang tentang komponen...
    memo: {
        // ID unik komponen...
        id: '0qCY3ri9pzSSMIXPGg8F',

        // Nama komponen. Contoh: <livewire:[name] />
        name: 'counter',

        // URI, method, dan locale dari halaman web tempat
        // komponen pertama kali dimuat. Ini digunakan
        // untuk menerapkan kembali middleware dari permintaan asli
        // ke permintaan pembaruan komponen selanjutnya (messages)...
        path: '/',
        method: 'GET',
        locale: 'en',

        // Daftar komponen "anak" yang bersarang. Diindeks berdasarkan
        // ID template internal dengan ID komponen sebagai nilainya...
        children: [],

        // Apakah komponen ini dimuat secara "lazy loaded" atau tidak...
        lazyLoaded: false,

        // Daftar error validasi yang muncul pada
        // permintaan terakhir...
        errors: [],
    },

    // Hash terenkripsi yang aman dari snapshot ini. Dengan cara ini,
    // jika pengguna jahat memanipulasi snapshot dengan
    // tujuan mengakses sumber daya yang bukan miliknya di server,
    // validasi checksum akan gagal dan error akan
    // dilemparkan...
    checksum: '1bc274eea17a434e33d26bcaba4a247a4a7768bd286456a83ea6e9be2d18c1e7',
}

```

---

### Objek `component`

Setiap komponen pada halaman memiliki objek komponen terkait di balik layar yang melacak **state**-nya dan mengekspos fungsionalitas dasarnya. Ini adalah satu lapisan lebih dalam dari `$wire` dan hanya ditujukan untuk penggunaan tingkat lanjut.

Berikut adalah objek komponen aktual untuk komponen `Counter` di atas dengan deskripsi properti yang relevan dalam komentar JS:

```js
let component = {
    // Elemen HTML root dari komponen...
    el: HTMLElement,

    // ID unik dari komponen...
    id: '0qCY3ri9pzSSMIXPGg8F',

    // "Nama" komponen (<livewire:[name] />)...
    name: 'counter',

    // Objek "effects" terbaru. Effects adalah efek samping (side-effects) dari server.
    // Ini termasuk pengalihan (redirect), unduhan file, dll...
    effects: {},

    // State sisi server terakhir yang diketahui dari komponen...
    canonical: { count: 0 },

    // Objek data mutable yang mewakili state
    // sisi klien yang aktif (live)...
    ephemeral: { count: 0 },

    // Versi reaktif dari `this.ephemeral`. Perubahan pada
    // objek ini akan ditangkap oleh ekspresi AlpineJS...
    reactive: Proxy,

    // Objek Proxy yang biasanya digunakan di dalam ekspresi Alpine
    // sebagai `$wire`. Ini dimaksudkan untuk menyediakan antarmuka
    // objek JS yang ramah bagi komponen Livewire...
    $wire: Proxy,

    // Daftar komponen "anak" yang bersarang...
    children: [],

    // Representasi "snapshot" terakhir yang diketahui dari komponen ini.
    // Snapshot diambil dari komponen sisi server dan digunakan
    // untuk membuat ulang objek PHP di backend...
    snapshot: {...},

    // Versi yang belum diparsing dari snapshot di atas. Ini digunakan untuk dikirim kembali ke
    // server pada siklus berikutnya karena parsing JS sering mengacaukan encoding PHP
    // yang mengakibatkan ketidakcocokan checksum (checksum mismatch).
    snapshotEncoded: '{"data":{"count":0},"memo":{"id":"0qCY3ri9pzSSMIXPGg8F","name":"counter","path":"\/","method":"GET","children":[],"lazyLoaded":true,"errors":[],"locale":"en"},"checksum":"1bc274eea17a434e33d26bcaba4a247a4a7768bd286456a83ea6e9be2d18c1e7"}',
}

```

---

### Payload `message`

Ketika sebuah **action** dilakukan pada komponen Livewire di browser, permintaan jaringan akan dipicu. Permintaan tersebut berisi satu atau banyak komponen dan berbagai instruksi untuk server. Secara internal, payload jaringan komponen ini disebut sebagai "**messages**".

Sebuah "**message**" mewakili data yang dikirim dari frontend ke backend saat komponen perlu diperbarui. Komponen di-render dan dimanipulasi di frontend hingga sebuah **action** dilakukan yang mengharuskannya mengirim pesan berisi **state** dan pembaruan ke backend.

Anda dapat mengenali skema ini dari payload di tab jaringan (Network) pada DevTools browser Anda, atau melalui [JavaScript hooks Livewire](https://www.google.com/search?q=%23javascript-hooks):

```js
let message = {
    // Objek snapshot...
    snapshot: { ... },

    // Daftar pasangan kunci-nilai (key-value) dari properti
    // yang akan diperbarui di server...
    updates: {},

    // Array metode (dengan parameter) yang akan dipanggil di sisi server...
    calls: [
        { method: 'increment', params: [] },
    ],
}

```
