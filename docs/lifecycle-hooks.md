Livewire menyediakan berbagai **lifecycle hooks** yang memungkinkan Anda menjalankan kode pada titik tertentu selama siklus hidup sebuah **component**. **Hooks** ini memungkinkan Anda untuk melakukan **actions** sebelum atau sesudah peristiwa tertentu, seperti menginisialisasi **component**, memperbarui properti, atau merender **template**.

Berikut adalah daftar semua **component lifecycle hooks** yang tersedia:

| Hook Method | Deskripsi |
| --- | --- |
| `mount()` | Dipanggil saat sebuah **component** dibuat |
| `hydrate()` | Dipanggil saat sebuah **component** di-**re-hydrated** pada awal **request** berikutnya |
| `boot()` | Dipanggil pada awal setiap **request**. Baik awal (initial), maupun berikutnya |
| `updating()` | Dipanggil sebelum memperbarui sebuah properti **component** |
| `updated()` | Dipanggil setelah memperbarui sebuah properti |
| `rendering()` | Dipanggil sebelum `render()` dipanggil |
| `rendered()` | Dipanggil setelah `render()` dipanggil |
| `dehydrate()` | Dipanggil pada akhir setiap **request** **component** |
| `exception($e, $stopPropagation)` | Dipanggil ketika sebuah **exception** dilemparkan |

## Mount

Dalam kelas PHP standar, sebuah **constructor** (`__construct()`) menerima parameter luar dan menginisialisasi **state** objek. Namun, di Livewire, Anda menggunakan metode `mount()` untuk menerima parameter dan menginisialisasi **state** dari **component** Anda.

**Components** Livewire tidak menggunakan `__construct()` karena **components** Livewire di-**re-constructed** pada **network requests** berikutnya, dan kita hanya ingin menginisialisasi **component** satu kali saat pertama kali dibuat.

Berikut adalah contoh penggunaan metode `mount()` untuk menginisialisasi properti `name` dan `email` dari sebuah **component** `profile.edit`:

```php
<?php // resources/views/components/profile/⚡edit.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

new class extends Component {
    public $name;

    public $email;

    public function mount()
    {
        $this->name = Auth::user()->name;

        $this->email = Auth::user()->email;
    }

    // ...
};

```

Seperti yang disebutkan sebelumnya, metode `mount()` menerima data yang diteruskan ke dalam **component** sebagai parameter metode:

```php
<?php // resources/views/components/post/⚡edit.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title;

    public $content;

    public function mount(Post $post)
    {
        $this->title = $post->title;

        $this->content = $post->content;
    }

    // ...
};

```

> [!tip] Anda dapat menggunakan dependency injection dengan semua metode hook
> Livewire memungkinkan Anda untuk menyelesaikan dependensi dari [Laravel's service container](https://laravel.com/docs/container#automatic-injection) dengan melakukan **type-hinting** pada parameter metode di **lifecycle hooks**.

Metode `mount()` adalah bagian krusial dalam menggunakan Livewire. Dokumentasi berikut menyediakan contoh lebih lanjut penggunaan metode `mount()` untuk menyelesaikan tugas-tugas umum:

* [Initializing properties](https://www.google.com/search?q=/docs/4.x/properties%23initializing-properties)
* [Receiving data from parent components](https://www.google.com/search?q=/docs/4.x/nesting%23passing-props-to-children)
* [Accessing route parameters](https://www.google.com/search?q=/docs/4.x/pages%23accessing-route-parameters)

## Boot

Meskipun `mount()` sangat membantu, ia hanya berjalan satu kali per siklus hidup **component**, dan Anda mungkin ingin menjalankan logika pada awal setiap **request** ke server untuk **component** tertentu.

Untuk kasus ini, Livewire menyediakan metode `boot()` di mana Anda dapat menulis kode pengaturan **component** yang ingin Anda jalankan setiap kali kelas **component** di-**boot**: baik saat inisialisasi maupun pada **requests** berikutnya.

Metode `boot()` dapat berguna untuk hal-hal seperti menginisialisasi properti **protected**, yang tidak dipertahankan (**persisted**) di antara **requests**. Di bawah ini adalah contoh menginisialisasi properti **protected** sebagai **Eloquent model**:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Locked;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Locked]
    public $postId = 1;

    protected Post $post;

    public function boot() // [tl! highlight:3]
    {
        $this->post = Post::find($this->postId);
    }

    // ...
};

```

Anda dapat menggunakan teknik ini untuk memiliki kontrol penuh atas inisialisasi properti **component** di dalam **component** Livewire Anda.

> [!tip] Seringkali, Anda dapat menggunakan computed property sebagai gantinya
> Teknik yang digunakan di atas memang kuat; namun, seringkali lebih baik menggunakan [Livewire's computed properties](https://www.google.com/search?q=/docs/4.x/computed-properties) untuk menyelesaikan kasus penggunaan ini.

> [!warning] Selalu kunci (lock) properti publik yang sensitif
> Seperti yang Anda lihat di atas, kita menggunakan atribut `#[Locked]` pada properti `$postId`. Dalam skenario seperti di atas, di mana Anda ingin memastikan properti `$postId` tidak dirusak oleh pengguna di sisi klien, penting untuk mengotorisasi nilai properti sebelum menggunakannya atau menambahkan `#[Locked]` ke properti tersebut untuk memastikan nilainya tidak pernah diubah.
> Untuk informasi lebih lanjut, lihat [dokumentasi atribut Locked](https://www.google.com/search?q=/docs/4.x/attribute-locked).

## Update

Pengguna di sisi klien dapat memperbarui properti publik dengan berbagai cara, paling umum dengan memodifikasi **input** yang memiliki `wire:model` di atasnya.

Livewire menyediakan **hooks** yang nyaman untuk mencegat pembaruan properti publik sehingga Anda dapat memvalidasi atau mengotorisasi sebuah nilai sebelum ditetapkan, atau memastikan sebuah properti ditetapkan dalam format tertentu.

Di bawah ini adalah contoh penggunaan `updating` untuk mencegah modifikasi properti `$postId`.

Perlu dicatat bahwa untuk contoh khusus ini, dalam aplikasi yang sebenarnya, Anda seharusnya menggunakan [atribut `#[Locked]](https://www.google.com/search?q=/docs/4.x/attribute-locked)` sebagai gantinya, seperti pada contoh di atas.

```php
<?php // resources/views/components/post/⚡show.blade.php

use Exception;
use Livewire\Component;

new class extends Component {
    public $postId = 1;

    public function updating($property, $value)
    {
        // $property: Nama properti saat ini yang sedang diperbarui
        // $value: Nilai yang akan ditetapkan ke properti tersebut

        if ($property === 'postId') {
            throw new Exception;
        }
    }

    // ...
};

```

Metode `updating()` di atas berjalan sebelum properti diperbarui, memungkinkan Anda untuk menangkap **input** yang tidak valid dan mencegah properti diperbarui. Di bawah ini adalah contoh penggunaan `updated()` untuk memastikan nilai properti tetap konsisten:

```php
<?php // resources/views/components/user/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public $username = '';

    public $email = '';

    public function updated($property)
    {
        // $property: Nama properti saat ini yang telah diperbarui

        if ($property === 'username') {
            $this->username = strtolower($this->username);
        }
    }

    // ...
};

```

Sekarang, setiap kali properti `$username` diperbarui di sisi klien, kita akan memastikan bahwa nilainya akan selalu berupa huruf kecil.

Karena Anda seringkali menargetkan properti tertentu saat menggunakan **update hooks**, Livewire memungkinkan Anda untuk menentukan nama properti secara langsung sebagai bagian dari nama metode. Berikut adalah contoh yang sama dari atas tetapi ditulis ulang menggunakan teknik ini:

```php
<?php // resources/views/components/user/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public $username = '';

    public $email = '';

    public function updatedUsername()
    {
        $this->username = strtolower($this->username);
    }

    // ...
};

```

Tentu saja, Anda juga dapat menerapkan teknik ini pada **hook** `updating`.

### Arrays

Properti **array** memiliki argumen tambahan `$key` yang diteruskan ke fungsi-fungsi ini untuk menentukan elemen yang berubah.

Perhatikan bahwa ketika **array** itu sendiri diperbarui alih-alih kunci tertentu, argumen `$key` bernilai **null**.

```php
<?php // resources/views/components/preferences/⚡edit.blade.php

use Livewire\Component;

new class extends Component {
    public $preferences = [];

    public function updatedPreferences($value, $key)
    {
        // $value = 'dark'
        // $key   = 'theme'
    }

    // ...
};

```

## Hydrate & Dehydrate

**Hydrate** dan **dehydrate** adalah **hooks** yang kurang dikenal dan kurang dimanfaatkan. Namun, ada skenario tertentu di mana mereka bisa menjadi sangat kuat.

Istilah "dehydrate" dan "hydrate" merujuk pada **component** Livewire yang diserialisasi menjadi JSON untuk sisi klien dan kemudian di-**unserialized** kembali menjadi objek PHP pada **request** berikutnya.

Kami sering menggunakan istilah "hydrate" dan "dehydrate" untuk merujuk pada proses ini di seluruh basis kode dan dokumentasi Livewire. Jika Anda ingin kejelasan lebih lanjut tentang istilah-istilah ini, Anda dapat mempelajari lebih lanjut dengan [berkonsultasi pada dokumentasi hidrasi kami](https://www.google.com/search?q=/docs/4.x/hydration).

Mari kita lihat contoh yang menggunakan `mount()`, `hydrate()`, dan `dehydrate()` sekaligus untuk mendukung penggunaan **data transfer object (DTO)** kustom alih-alih **Eloquent model** untuk menyimpan data **post** di dalam **component**:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Component;

new class extends Component {
    public $post;

    public function mount($title, $content)
    {
        // Berjalan pada awal request inisial pertama...

        $this->post = new PostDto([
            'title' => $title,
            'content' => $content,
        ]);
    }

    public function hydrate()
    {
        // Berjalan pada awal setiap request "berikutnya"...
        // Ini tidak berjalan pada request inisial ("mount" yang berjalan)...

        $this->post = new PostDto($this->post);
    }

    public function dehydrate()
    {
        // Berjalan pada akhir setiap request...

        $this->post = $this->post->toArray();
    }

    // ...
};

```

Sekarang, dari **actions** dan tempat lain di dalam **component** Anda, Anda dapat mengakses objek `PostDto` alih-alih data primitif.

Contoh di atas terutama mendemonstrasikan kemampuan dan sifat dari **hooks** `hydrate()` dan `dehydrate()`. Namun, direkomendasikan agar Anda menggunakan [Wireables atau Synthesizers](https://www.google.com/search?q=/docs/4.x/properties%23supporting-custom-types) untuk mencapai hal ini sebagai gantinya.

## Render

Jika Anda ingin masuk ke dalam proses perenderan **Blade view** sebuah **component**, Anda dapat melakukannya menggunakan **hooks** `rendering()` dan `rendered()`:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public function render()
    {
        return $this->view([
            'post' => Post::all(),
        ]);
    }

    public function rendering($view, $data)
    {
        // Berjalan SEBELUM view yang disediakan dirender...
        //
        // $view: View yang akan dirender
        // $data: Data yang disediakan untuk view tersebut
    }

    public function rendered($view, $html)
    {
        // Berjalan SESUDAH view yang disediakan dirender...
        //
        // $view: View yang telah dirender
        // $html: HTML akhir yang telah dirender
    }

    // ...
};

```

## Exception

Terkadang sangat membantu untuk mencegat dan menangkap kesalahan (**errors**), misalnya: untuk menyesuaikan pesan kesalahan atau mengabaikan jenis **exceptions** tertentu. **Hook** `exception()` memungkinkan Anda melakukan hal tersebut: Anda dapat melakukan pemeriksaan pada `$error`, dan menggunakan parameter `$stopPropagation` untuk menangkap masalahnya.
Ini juga membuka pola yang kuat ketika Anda ingin menghentikan eksekusi kode lebih lanjut (**return early**), begitulah cara metode internal seperti `validate()` bekerja.

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Component;

new class extends Component {
    public function mount() // [tl! highlight:3]
    {
        $this->post = Post::find($this->postId);
    }

    public function exception($e, $stopPropagation) {
        if ($e instanceof NotFoundException) {
            $this->notify('Post is not found');
            $stopPropagation();
        }
    }

    // ...
};

```

## Menggunakan hooks di dalam sebuah trait

**Traits** adalah cara yang membantu untuk menggunakan kembali kode di berbagai **components** atau mengekstrak kode dari satu **component** ke dalam file khusus.

Untuk menghindari konflik antar **traits** saat mendeklarasikan metode **lifecycle hook**, Livewire mendukung pemberian awalan (**prefixing**) pada metode **hook** dengan nama **camelCased** dari **trait** saat ini yang mendeklarasikannya.

Dengan cara ini, Anda dapat memiliki beberapa **traits** yang menggunakan **lifecycle hooks** yang sama dan menghindari definisi metode yang berkonflik.

Di bawah ini adalah contoh sebuah **component** yang mereferensikan sebuah **trait** bernama `HasPostForm`:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    use HasPostForm;

    // ...
};

```

Sekarang inilah **trait** `HasPostForm` yang sebenarnya berisi semua **prefixed hooks** yang tersedia:

```php
trait HasPostForm
{
    public $title = '';

    public $content = '';

    public function mountHasPostForm()
    {
        // ...
    }

    public function hydrateHasPostForm()
    {
        // ...
    }

    public function bootHasPostForm()
    {
        // ...
    }

    public function updatingHasPostForm()
    {
        // ...
    }

    public function updatedHasPostForm()
    {
        // ...
    }

    public function renderingHasPostForm()
    {
        // ...
    }

    public function renderedHasPostForm()
    {
        // ...
    }

    public function dehydrateHasPostForm()
    {
        // ...
    }

    // ...
}

```

## Menggunakan hooks di dalam sebuah form object

**Form objects** di Livewire mendukung **property update hooks**. **Hooks** ini bekerja mirip dengan [component update hooks](https://www.google.com/search?q=%23update), memungkinkan Anda melakukan **actions** ketika properti di dalam **form object** berubah.

Di bawah ini adalah contoh sebuah **component** yang menggunakan sebuah **form object** `PostForm`:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public PostForm $form;

    // ...
};

```

Berikut adalah **form object** `PostForm` yang berisi semua **hooks** yang tersedia:

```php
namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    public $title = '';

    public $tags = [];

    public function updating($property, $value)
    {
        // ...
    }

    public function updated($property, $value)
    {
        // ...
    }

    public function updatingTitle($value)
    {
        // ...
    }

    public function updatedTitle($value)
    {
        // ...
    }

    public function updatingTags($value, $key)
    {
        // ...
    }

    public function updatedTags($value, $key)
    {
        // ...
    }

    // ...
}

```

---

## See also

* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Menginisialisasi properti di dalam `mount()` dan `boot()`
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Memahami kapan **hooks** berjalan selama pembuatan **component**
* **[Pages](https://www.google.com/search?q=/docs/4.x/pages)** — Menggunakan `mount()` untuk menerima parameter **route**
* **[Hydration](https://www.google.com/search?q=/docs/4.x/hydration)** — Memahami **hooks** `hydrate()` dan `dehydrate()`
