Livewire **components** pada dasarnya adalah PHP **classes** dengan **properties** dan **methods** yang dapat dipanggil langsung dari sebuah Blade **template**. Kombinasi ampuh ini memungkinkan Anda membuat antarmuka *full-stack* yang interaktif dengan upaya dan kompleksitas yang jauh lebih kecil dibandingkan alternatif JavaScript modern.

Panduan ini mencakup segala hal yang perlu Anda ketahui tentang membuat, melakukan **rendering**, dan mengorganisir Livewire **components**. Anda akan mempelajari berbagai format **component** yang tersedia (**single-file**, **multi-file**, dan **class-based**), cara mengirimkan data antar **components**, dan cara menggunakan **components** sebagai **full pages**.

## Creating components

Anda dapat membuat sebuah **component** menggunakan **Artisan command** `make:livewire`:

```shell
php artisan make:livewire post.create

```

Ini akan membuat sebuah **single-file component** pada:

`resources/views/components/post/⚡create.blade.php`

```blade
<?php

use Livewire\Component;

new class extends Component {
    public $title = '';

    public function save()
    {
        // Save logic here...
    }
};
?>

<div>
    <input wire:model="title" type="text">
    <button wire:click="save">Save Post</button>
</div>

```

> [!info] Mengapa ada emoji ⚡?
> Anda mungkin bertanya-tanya tentang simbol petir pada nama file tersebut. Sentuhan kecil ini memiliki tujuan praktis: membuat Livewire **components** langsung dikenali di **file tree** editor Anda dan hasil pencarian. Karena ini adalah karakter Unicode, ia bekerja dengan mulus di semua platform — Windows, macOS, Linux, Git, dan **production servers** Anda.
> Emoji ini sepenuhnya opsional dan jika Anda merasa kurang nyaman, Anda dapat menonaktifkannya sepenuhnya di file `config/livewire.php` Anda:
> ```php
> 'make_command' => [
>      'emoji' => false,
> ],
> 
> ```
> 
> 

### Creating page components

Saat membuat **components** yang akan digunakan sebagai **full pages**, gunakan **namespace** `pages::` untuk mengorganisirnya dalam direktori khusus:

```shell
php artisan make:livewire pages::post.create

```

Ini membuat **component** tersebut pada `resources/views/pages/post/⚡create.blade.php`. Organisasi ini memperjelas mana **components** yang berfungsi sebagai halaman versus UI **components** yang dapat digunakan kembali (**reusable**).

Pelajari lebih lanjut tentang penggunaan **components** sebagai halaman di bagian [Page components section](https://www.google.com/search?q=%23page-components) di bawah. Anda juga dapat meregistrasi **custom namespaces** Anda sendiri — lihat [Component namespaces documentation](https://www.google.com/search?q=/docs/4.x/pages%23component-namespaces).

### Multi-file components

Seiring berkembangnya **component** atau proyek Anda, Anda mungkin merasa pendekatan **single-file** mulai membatasi. Livewire menawarkan alternatif **multi-file** yang membagi **component** Anda menjadi file-file terpisah untuk organisasi yang lebih baik dan dukungan IDE.

Untuk membuat sebuah **multi-file component**, tambahkan **flag** `--mfc`:

```shell
php artisan make:livewire post.create --mfc

```

Ini akan membuat sebuah direktori dengan semua file terkait dikelompokkan bersama:

```
resources/views/components/post/⚡create/
├── create.php          # PHP class
├── create.blade.php    # Blade template
├── create.js           # JavaScript (optional)
├── create.css          # Scoped styles (optional)
├── create.global.css   # Global styles (optional)
└── create.test.php     # Pest test (optional, dengan flag --test)

```

### Converting between formats

Livewire menyediakan **command** `livewire:convert` untuk mengubah **components** secara mulus antara format **single-file** dan **multi-file**.

**Auto-detect and convert:**

```shell
php artisan livewire:convert post.create
# Single-file → Multi-file (atau sebaliknya)

```

**Explicitly convert ke multi-file:**

```shell
php artisan livewire:convert post.create --mfc

```

Ini akan mem-**parse single-file component** Anda, membuat struktur direktori, membagi file-filenya, dan menghapus file aslinya.

**Explicitly convert ke single-file:**

```shell
php artisan livewire:convert post.create --sfc

```

Ini menggabungkan kembali semua file ke dalam satu file tunggal dan menghapus direktorinya.

> [!warning] Test files akan terhapus saat konversi ke single-file
> Jika **multi-file component** Anda memiliki file tes, Anda akan diminta konfirmasi sebelum konversi karena file tes tidak dapat dipertahankan dalam format **single-file**.

### When to use each format

**Single-file components (default):**

* Terbaik untuk sebagian besar **components**
* Menjaga kode yang berkaitan tetap bersama
* Mudah dipahami secara sekilas
* Sempurna untuk **components** kecil hingga menengah

**Multi-file components:**

* Lebih baik untuk **components** yang besar dan kompleks
* Dukungan IDE dan navigasi yang lebih baik
* Pemisahan yang lebih jelas ketika **components** memiliki banyak JavaScript

**Class-based components:**

* Familiar bagi pengembang dari Livewire v2/v3
* Tradisional Laravel **separation of concerns**
* Lebih baik untuk tim dengan konvensi yang sudah mapan
* Lihat [Class-based components](https://www.google.com/search?q=%23class-based-components) di bawah

---

## Rendering components

Anda dapat menyertakan sebuah Livewire **component** di dalam **Blade template** mana pun menggunakan sintaks `<livewire:component-name />`:

```blade
<livewire:component-name />

```

Jika **component** terletak di dalam sub-direktori, Anda dapat menunjukkannya menggunakan karakter titik (`.`):

`resources/views/components/post/⚡create.blade.php`

```blade
<livewire:post.create />

```

Untuk **namespaced components** — seperti `pages::` — gunakan awalan **namespace**:

```blade
<livewire:pages::post.create />

```

### Passing props

Untuk mengirimkan data ke dalam Livewire **component**, Anda dapat menggunakan **prop attributes** pada tag **component**:

```blade
<livewire:post.create title="Initial Title" />

```

Untuk nilai dinamis atau variabel, gunakan awalan titik dua (`:`):

```blade
<livewire:post.create :title="$initialTitle" />

```

Data yang dikirimkan ke dalam **components** diterima melalui **method** `mount()`:

```php
<?php

use Livewire\Component;

new class extends Component {
    public $title;

    public function mount($title = null)
    {
        $this->title = $title;
    }

    // ...
};

```

Anda dapat menganggap **method** `mount()` sebagai sebuah **class constructor**. Ia berjalan saat **component** diinisialisasi, tetapi tidak pada **requests** berikutnya dalam sesi halaman tersebut. Anda dapat mempelajari lebih lanjut tentang `mount()` dan **lifecycle hooks** lainnya di dalam [lifecycle documentation](https://www.google.com/search?q=/docs/4.x/lifecycle-hooks).

Untuk mengurangi kode **boilerplate**, Anda dapat mengabaikan **method** `mount()` dan Livewire akan secara otomatis mengisi **properties** yang namanya sesuai dengan nilai yang dikirimkan:

```php
<?php

use Livewire\Component;

new class extends Component {
    public $title; // Secara otomatis diisi dari prop

    // ...
};

```

> [!warning] Properties ini tidak reactive secara default
> **Property** `$title` tidak akan ter-**update** secara otomatis jika nilai luar `:title="$initialValue"` berubah setelah pemuatan halaman awal. Ini adalah poin kebingungan yang umum saat menggunakan Livewire, terutama bagi pengembang yang pernah menggunakan JavaScript **frameworks** seperti Vue atau React dan berasumsi parameter ini berperilaku seperti "**reactive props**". Tapi, jangan khawatir, Livewire memungkinkan Anda untuk memilih untuk [membuat props Anda menjadi reactive](https://www.google.com/search?q=/docs/4.x/nesting%23reactive-props).

### Passing route parameters sebagai props

Saat menggunakan **components** sebagai halaman, Anda dapat mengirimkan **route parameters** secara langsung ke **component** Anda. **Route parameters** secara otomatis dikirimkan ke **method** `mount()`:

```php
Route::livewire('/posts/{id}', 'pages::post.show');

```

```php
<?php // resources/views/pages/post/⚡show.blade.php

use Livewire\Component;

new class extends Component {
    public $postId;

    public function mount($id)
    {
        $this->postId = $id;
    }
};

```

Livewire juga mendukung Laravel **route model binding**:

```php
Route::livewire('/posts/{post}', 'pages::post.show');

```

```php
<?php // resources/views/pages/post/⚡show.blade.php

use App\Models\Post;
use Livewire\Component;

new class extends Component {
    public Post $post; // Secara otomatis di-bound dari route

    // Tidak butuh mount() - Livewire menanganinya secara otomatis
};

```

---

## Page components

**Components** dapat diarahkan secara langsung sebagai **full pages** menggunakan `Route::livewire()`. Ini adalah salah satu fitur paling kuat dari Livewire, yang memungkinkan Anda membangun seluruh halaman tanpa **controllers** tradisional.

```php
Route::livewire('/posts/create', 'pages::post.create');

```

Saat pengguna mengunjungi `/posts/create`, Livewire akan me-**render** komponen `pages::post.create` di dalam **layout file** aplikasi Anda.

**Page components** bekerja seperti **components** biasa, tetapi di-**render** sebagai halaman penuh dengan akses ke:

* **Custom layouts**
* **Page titles**
* **Route parameters** dan **model binding**
* **Named slots** untuk **layouts**

Untuk informasi lengkap mengenai **page components**, termasuk **layouts**, **titles**, dan **advanced routing**, lihat [Pages documentation](https://www.google.com/search?q=/docs/4.x/pages).

---

## Accessing data in views

Livewire menyediakan beberapa cara untuk mengirimkan data ke Blade **view** milik **component** Anda. Setiap pendekatan memiliki karakteristik performa dan keamanan yang berbeda.

### Component properties

Pendekatan paling sederhana adalah menggunakan **public properties**, yang secara otomatis tersedia di Blade **template** Anda:

```php
<?php

use Livewire\Component;

new class extends Component {
    public $title = 'My Post';
};

```

```blade
<div>
    <h1>{{ $title }}</h1>
</div>

```

**Protected properties** harus diakses menggunakan `$this->`:

```php
public $title = 'My Post';            // Tersedia sebagai {{ $title }}
protected $apiKey = 'secret-key';     // Tersedia sebagai {{ $this->apiKey }}

```

> [!info] Protected properties tidak dikirim ke client
> Berbeda dengan **public properties**, **protected properties** tidak pernah dikirim ke **frontend** dan tidak dapat dimanipulasi oleh pengguna. Hal ini membuatnya aman untuk data sensitif. Namun, mereka tidak **persisted** di antara **requests**, yang membatasi kegunaannya dalam banyak skenario Livewire. Mereka paling baik digunakan untuk nilai statis yang didefinisikan dalam deklarasi **property** yang tidak ingin Anda ekspos di sisi **client**.

Untuk informasi lengkap mengenai **properties**, termasuk perilaku **persistence** dan fitur lanjutan, lihat [properties documentation](https://www.google.com/search?q=/docs/4.x/properties).

### Computed properties

**Computed properties** adalah **methods** yang bertindak seperti **memoized properties**. Mereka sangat cocok untuk operasi yang berat seperti **database queries**:

```php
use Livewire\Attributes\Computed;

#[Computed]
public function posts()
{
    return Post::with('author')->latest()->get();
}

```

```blade
<div>
    @foreach ($this->posts as $post)
        <article wire:key="{{ $post->id }}">{{ $post->title }}</article>
    @endforeach
</div>

```

Perhatikan awalan `$this->` — ini memberitahu Livewire untuk memanggil **method** tersebut dan me-**cache** hasilnya hanya untuk **request** saat ini (bukan antar **requests**). Untuk detail lebih lanjut, lihat [computed properties section](https://www.google.com/search?q=/docs/4.x/properties%23computed-properties) di dokumentasi **properties**.

### Passing data dari render()

Mirip dengan sebuah **controller**, Anda dapat mengirimkan data secara langsung ke **view** menggunakan **method** `render()`:

```php
public function render()
{
    return $this->view([
        'author' => Auth::user(),
        'currentTime' => now(),
    ]);
}

```

Ingatlah bahwa `render()` berjalan pada setiap **component update**, jadi hindari operasi yang berat di sini kecuali Anda membutuhkan data segar pada setiap **update**.

---

## Organizing components

Meskipun Livewire secara otomatis menemukan **components** di direktori default `resources/views/components/`, Anda dapat menyesuaikan di mana Livewire mencari **components** dan mengorganisirnya menggunakan **namespaces**.

### Component namespaces

**Component namespaces** memungkinkan Anda mengorganisir **components** ke dalam direktori khusus dengan sintaks referensi yang bersih.

Secara **default**, Livewire menyediakan dua **namespaces**:

* `pages::` — Merujuk ke `resources/views/pages/`
* `layouts::` — Merujuk ke `resources/views/layouts/`

Anda dapat mendefinisikan **namespaces** tambahan di file `config/livewire.php` Anda:

```php
'component_namespaces' => [
    'layouts' => resource_path('views/layouts'),
    'pages' => resource_path('views/pages'),
    'admin' => resource_path('views/admin'),    // Custom namespace
    'widgets' => resource_path('views/widgets'), // Custom namespace lainnya
],

```

Kemudian gunakan mereka saat membuat, melakukan **rendering**, dan melakukan **routing**:

```shell
php artisan make:livewire admin::users-table

```

```blade
<livewire:admin::users-table />

```

```php
Route::livewire('/admin/users', 'admin::users-table');

```

### Additional component locations

Jika Anda ingin Livewire menemukan **components** di direktori tambahan di luar default, Anda dapat mengonfigurasinya di file `config/livewire.php` Anda:

```php
'component_locations' => [
    resource_path('views/components'),
    resource_path('views/admin/components'),
    resource_path('views/widgets'),
],

```

Sekarang Livewire akan secara otomatis menemukan **components** di semua direktori tersebut.

### Programmatic registration

Untuk skenario yang lebih dinamis (seperti pengembangan **package** atau **runtime configuration**), Anda dapat meregistrasi **components**, **locations**, dan **namespaces** secara terprogram di sebuah **service provider**:

**Register an individual component:**

```php
use Livewire\Livewire;

// Di dalam service provider boot() method (misal, App\Providers\AppServiceProvider)
Livewire::addComponent(
    name: 'custom-button',
    viewPath: resource_path('views/ui/button.blade.php')
);

```

**Register a component directory:**

```php
Livewire::addLocation(
    viewPath: resource_path('views/admin/components')
);

```

**Register a namespace:**

```php
Livewire::addNamespace(
    namespace: 'ui',
    viewPath: resource_path('views/ui')
);

```

#### Registering class-based components

Untuk **class-based components**, gunakan metode yang sama tetapi dengan parameter `class` alih-alih `path`:

```php
use Livewire\Livewire;

// Register individual class-based component
Livewire::addComponent(
    name: 'todos',
    class: \App\Livewire\Todos::class
);

// Register location untuk class-based components
Livewire::addLocation(
    classNamespace: 'App\\Admin\\Livewire'
);

// Create namespace untuk class-based components
Livewire::addNamespace(
    namespace: 'admin',
    classNamespace: 'App\\Admin\\Livewire',
    classPath: app_path('Admin/Livewire'),
    classViewPath: resource_path('views/admin/livewire')
);

```

---

## Class-based components

Untuk tim yang bermigrasi dari Livewire v3 atau mereka yang lebih menyukai struktur Laravel yang lebih tradisional, Livewire mendukung penuh **class-based components**. Pendekatan ini memisahkan PHP **class** dan Blade **view** ke dalam file yang berbeda di lokasi konvensional mereka.

### Creating class-based components

```shell
php artisan make:livewire CreatePost --class

```

Ini membuat dua file terpisah:

`app/Livewire/CreatePost.php`

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public function render()
	{
		return view('livewire.create-post');
	}
}

```

`resources/views/livewire/create-post.blade.php`

```blade
<div>
	{{-- ... --}}
</div>

```

### When to use class-based components

**Gunakan class-based components saat:**

* Bermigrasi dari Livewire v2/v3
* Tim Anda lebih menyukai struktur file tradisional
* Anda telah menetapkan konvensi di sekitar **class-based architecture**

**Gunakan single-file atau multi-file components saat:**

* Memulai proyek baru Livewire v4
* Anda menginginkan **component colocation** yang lebih baik
* Anda ingin menggunakan konvensi Livewire terbaru

---

## Customizing component stubs

Anda dapat menyesuaikan file (atau *stubs*) yang digunakan Livewire untuk men-**generate components** baru dengan menjalankan:

```shell
php artisan livewire:stubs

```

Ini membuat file **stub** di aplikasi Anda yang dapat Anda modifikasi, seperti:

* `stubs/livewire-sfc.stub` — Untuk **single-file components**
* `stubs/livewire.stub` — Untuk **class-based components** (PHP class)

Setelah di-**publish**, Livewire akan secara otomatis menggunakan **custom stubs** Anda saat membuat **components** baru.

---

## Troubleshooting

### Component not found

**Symptom:** Pesan kesalahan seperti "Component [post.create] not found"
**Solutions:**

* Verifikasi file **component** ada di **path** yang diharapkan.
* Pastikan nama **component** di **view** Anda sesuai dengan struktur file (gunakan titik untuk sub-direktori).
* Untuk **namespaced components**, pastikan **namespace** didefinisikan di `config/livewire.php`.

### Component shows blank or doesn't render

**Common causes:**

* Kehilangan **root element** di Blade **template** Anda (Livewire membutuhkan tepat satu **root element**).
* Kesalahan sintaks pada bagian PHP di **component** Anda.

### Class name conflicts

**Symptom:** Kesalahan mengenai nama **class** yang duplikat saat menggunakan **single-file components**.
**Solution:** Terjadi jika Anda memiliki beberapa **single-file components** dengan nama yang sama di direktori berbeda. Ganti nama salah satu komponen agar unik atau gunakan **namespace**.

---

## See also

* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Mengelola **state** dan data komponen
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Menangani interaksi pengguna dengan **methods**
* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Menggunakan komponen sebagai halaman penuh dengan **routing**
* **[Nesting](https://www.google.com/search?q=/docs/4.x/nesting)** — Menyusun komponen bersama dan mengirim data di antaranya
* **[Lifecycle Hooks](https://www.google.com/search?q=/docs/4.x/lifecycle-hooks)** — Menjalankan kode pada titik spesifik dalam siklus hidup komponen
