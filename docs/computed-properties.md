**Computed properties** adalah cara untuk membuat properti "turunan" (*derived*) di Livewire. Seperti *accessors* pada model Eloquent, **computed properties** memungkinkan Anda mengakses nilai dan melakukan *memoize* (penyimpanan sementara) terhadap nilai tersebut untuk akses di masa mendatang selama satu **request**.

**Computed properties** sangat berguna jika dikombinasikan dengan properti publik **component**.

## Penggunaan dasar

Untuk membuat **computed property**, Anda dapat menambahkan atribut `#[Computed]` di atas metode apa pun dalam **component** Livewire Anda. Setelah atribut ditambahkan ke metode tersebut, Anda dapat mengaksesnya seperti properti lainnya.

> [!warning] Pastikan Anda mengimpor class attribute
> Pastikan Anda mengimpor setiap class **attribute**. Sebagai contoh, atribut `#[Computed]` di bawah ini memerlukan impor berikut: `use Livewire\Attributes\Computed;`.

Sebagai contoh, berikut adalah **component** `show-user` yang menggunakan **computed property** bernama `user()` untuk mengakses model Eloquent `User` berdasarkan properti bernama `$userId`:

```php
<?php // resources/views/components/⚡show-user.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $userId;

    #[Computed]
    public function user()
    {
        return User::find($this->userId);
    }

    public function follow()
    {
        Auth::user()->follow($this->user);
    }
};

```

```blade
<div>
    <h1>{{ $this->user->name }}</h1>

    <span>{{ $this->user->email }}</span>

    <button wire:click="follow">Follow</button>
</div>

```

Karena atribut `#[Computed]` telah ditambahkan ke metode `user()`, nilainya dapat diakses di metode lain dalam **component** tersebut maupun di dalam **template** Blade.

> [!info] Harus menggunakan `$this` di template Anda
> Berbeda dengan properti biasa, **computed properties** tidak tersedia secara langsung di dalam **template component** Anda. Sebaliknya, Anda harus mengaksesnya melalui objek `$this`. Misalnya, **computed property** bernama `posts()` harus diakses melalui `$this->posts` di dalam **template** Anda.

> [!warning] Computed properties tidak didukung pada objek `Livewire\Form`.
> Mencoba menggunakan **Computed property** di dalam [Form](https://livewire.laravel.com/docs/forms) akan menyebabkan error saat Anda mencoba mengakses properti tersebut di Blade menggunakan sintaks `$form->property`.

## Keunggulan performa

Anda mungkin bertanya-tanya: mengapa harus menggunakan **computed properties**? Mengapa tidak memanggil metodenya secara langsung saja?

Mengakses metode sebagai **computed property** menawarkan keunggulan performa dibandingkan memanggil metode biasa. Secara internal, saat **computed property** dijalankan untuk pertama kalinya, Livewire melakukan *memoize* terhadap nilai yang dikembalikan. Dengan cara ini, akses berikutnya dalam satu **request** yang sama akan mengembalikan nilai yang sudah disimpan tersebut alih-alih mengeksekusi metode berkali-kali.

Hal ini memungkinkan Anda bebas mengakses nilai turunan tanpa perlu khawatir tentang implikasi performanya.

> [!warning] Computed properties hanya di-memoize untuk satu request
> Ada kesalahpahaman umum bahwa Livewire menyimpan (*memoize*) **computed properties** selama seluruh masa hidup **component** Livewire Anda di sebuah halaman. Namun, kenyataannya tidak demikian. Livewire hanya menyimpan hasilnya selama durasi satu **request component** saja (tidak bertahan di antara **requests**). Ini berarti jika metode **computed property** Anda berisi kueri database yang berat, kueri tersebut akan tetap dijalankan setiap kali **component** Livewire Anda melakukan pembaruan (*update*).

### Membersihkan memo

Pertimbangkan skenario bermasalah berikut:

1. Anda mengakses **computed property** yang bergantung pada properti tertentu atau status database.
2. Properti dasar atau status database tersebut berubah.
3. Nilai yang tersimpan (*memoize*) untuk properti tersebut menjadi usang (*stale*) dan perlu dihitung ulang.

Untuk membersihkan, atau melakukan "bust" pada memo yang tersimpan, Anda dapat menggunakan fungsi `unset()` bawaan PHP.

Di bawah ini adalah contoh sebuah **action** bernama `createPost()` yang, dengan membuat postingan baru, menyebabkan **computed** `posts()` menjadi usang — artinya **computed property** `posts()` perlu dihitung ulang untuk menyertakan postingan yang baru ditambahkan:

```php
<?php // resources/views/components/⚡show-posts.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    public function createPost()
    {
        if ($this->posts->count() > 10) {
            throw new \Exception('Maximum post count exceeded');
        }

        Auth::user()->posts()->create(...);

        unset($this->posts); // [tl! highlight]
    }

    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    // ...
};

```

Pada **component** di atas, **computed property** sudah tersimpan sebelum postingan baru dibuat karena metode `createPost()` mengakses `$this->posts` sebelum pembuatan data. Untuk memastikan bahwa `$this->posts` berisi konten terbaru saat diakses di dalam **view**, memo dibersihkan menggunakan `unset($this->posts)`.

### Caching di antara requests

> [!tip] Memoization vs Caching
> *Memoization* yang kita bahas sejauh ini hanya bertahan selama satu **request**. Jika Anda membutuhkan nilai yang tetap ada di antara beberapa **requests**, Anda memerlukan fitur **caching** Laravel yang sesungguhnya.

Terkadang Anda ingin menyimpan (*cache*) nilai dari **computed property** selama masa hidup sebuah **component** Livewire, alih-alih dibersihkan setiap kali **request** selesai. Dalam kasus ini, Anda dapat menggunakan [utilitas caching Laravel](https://laravel.com/docs/cache#retrieve-store).

Berikut adalah contoh **computed property** bernama `user()`, di mana alih-alih mengeksekusi kueri Eloquent secara langsung, kita membungkus kueri tersebut dalam `Cache::remember()` untuk memastikan bahwa **requests** di masa mendatang mengambilnya dari **cache** Laravel alih-alih mengeksekusi ulang kueri:

```php
<?php // resources/views/components/⚡show-user.blade.php

use Illuminate\Support\Facades\Cache;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $userId;

    #[Computed]
    public function user()
    {
        $key = 'user'.$this->getId();
        $seconds = 3600; // 1 jam...

        return Cache::remember($key, $seconds, function () {
            return User::find($this->userId);
        });
    }

    // ...
};

```

Karena setiap instansi unik dari **component** Livewire memiliki ID yang unik, kita dapat menggunakan `$this->getId()` untuk menghasilkan **cache key** unik yang hanya akan diterapkan pada **requests** mendatang untuk instansi **component** yang sama ini.

Namun, seperti yang mungkin Anda perhatikan, sebagian besar kode ini dapat diprediksi dan diabstraksi dengan mudah. Oleh karena itu, atribut `#[Computed]` Livewire menyediakan parameter `persist` yang sangat membantu. Dengan menerapkan `#[Computed(persist: true)]` pada sebuah metode, Anda dapat mencapai hasil yang sama tanpa kode tambahan:

```php
use Livewire\Attributes\Computed;
use App\Models\User;

#[Computed(persist: true)]
public function user()
{
    return User::find($this->userId);
}

```

Pada contoh di atas, saat `$this->user` diakses dari **component** Anda, nilainya akan terus disimpan dalam **cache** selama durasi **component** Livewire tersebut ada di halaman. Ini berarti kueri Eloquent yang sebenarnya hanya akan dijalankan satu kali saja.

Livewire menyimpan nilai yang di-*persist* selama 3600 detik (satu jam). Anda dapat mengubah nilai default ini dengan meneruskan parameter `seconds` tambahan ke atribut `#[Computed]`:

```php
#[Computed(persist: true, seconds: 7200)]

```

> [!tip] Memanggil `unset()` akan membersihkan memo dan cache
> Seperti yang dibahas sebelumnya, Anda dapat menghapus memo **computed property** menggunakan metode `unset()` PHP. Ini juga berlaku untuk **computed properties** yang menggunakan parameter `persist: true`. Saat memanggil `unset()` pada properti tersebut, Livewire tidak hanya akan menghapus memo dalam **request**, tetapi juga menghapus nilai terkait di dalam **cache** Laravel.

## Caching di semua components

Alih-alih menyimpan nilai **computed property** hanya selama siklus hidup satu **component** saja, Anda dapat menyimpan nilai tersebut di seluruh **components** dalam aplikasi Anda menggunakan parameter `cache: true` yang disediakan oleh atribut `#[Computed]`:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true)]
public function posts()
{
    return Post::all();
}

```

Pada contoh di atas, sampai **cache** kedaluwarsa atau dibersihkan, setiap instansi dari **component** ini di seluruh aplikasi Anda akan berbagi nilai **cache** yang sama untuk `$this->posts`.

Jika Anda perlu membersihkan **cache** secara manual untuk sebuah **computed property**, Anda dapat menetapkan **cache key** kustom menggunakan parameter `key`:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true, key: 'homepage-posts')]
public function posts()
{
    return Post::all();
}

```

## Kapan sebaiknya menggunakan computed properties?

Selain menawarkan keuntungan performa, ada beberapa skenario lain di mana **computed properties** sangat membantu.

Secara khusus, saat meneruskan data ke dalam **template** Blade **component** Anda, ada beberapa kondisi di mana **computed property** menjadi alternatif yang lebih baik. Di bawah ini adalah contoh metode `render()` pada **component** sederhana yang meneruskan koleksi `posts` ke **template** Blade:

```php
public function render()
{
    return view('livewire.show-posts', [
        'posts' => Post::all(),
    ]);
}

```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            </div>
    @endforeach
</div>

```

Meskipun cara ini cukup untuk banyak kasus penggunaan, berikut adalah tiga skenario di mana **computed property** akan menjadi alternatif yang lebih baik:

### Mengakses nilai secara kondisional

Jika Anda mengakses suatu nilai secara kondisional yang proses pengambilannya memakan biaya komputasi besar di dalam **template** Blade, Anda dapat mengurangi beban performa menggunakan **computed property**.

Pertimbangkan **template** berikut tanpa **computed property**:

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($posts as $post)
            <div wire:key="{{ $post->id }}">
                </div>
        @endforeach
    @endif
</div>

```

Jika seorang pengguna dilarang melihat postingan, kueri database untuk mengambil postingan tersebut tetap dijalankan, padahal postingan tersebut tidak pernah digunakan di dalam **template**.

Berikut adalah versi dari skenario di atas menggunakan **computed property**:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed]
public function posts()
{
    return Post::all();
}

public function render()
{
    return view('livewire.show-posts');
}

```

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($this->posts as $post)
            <div wire:key="{{ $post->id }}">
                </div>
        @endforeach
    @endif
</div>

```

Sekarang, karena kita menyediakan postingan ke **template** menggunakan **computed property**, kueri database hanya akan dieksekusi saat data tersebut benar-benar dibutuhkan.

### Menggunakan inline templates

Skenario lain di mana **computed properties** sangat membantu adalah saat menggunakan [inline templates](https://www.google.com/search?q=/docs/4.x/components%23inline-components) pada **component** Anda.

Di bawah ini adalah contoh **inline component** di mana, karena kita mengembalikan string **template** secara langsung di dalam `render()`, kita tidak pernah memiliki kesempatan untuk meneruskan data ke dalam **view**:

```php
<?php // resources/views/components/⚡show-posts.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Post::all();
    }

    public function render()
    {
        return <<<HTML
        <div>
            @foreach ($this->posts as $post)
                <div wire:key="{{ $post->id }}">
                    </div>
            @endforeach
        </div>
        HTML;
    }
};

```

Pada contoh di atas, tanpa **computed property**, kita tidak akan memiliki cara untuk meneruskan data secara eksplisit ke dalam **template** Blade.

### Meniadakan metode render

Di Livewire, cara lain untuk mengurangi kode repetitif (*boilerplate*) pada **component** Anda adalah dengan meniadakan metode `render()` sepenuhnya. Saat ditiadakan, Livewire akan menggunakan metode `render()` miliknya sendiri yang mengembalikan **Blade view** yang sesuai berdasarkan konvensi.

Dalam kasus ini, Anda jelas tidak memiliki metode `render()` sebagai tempat untuk meneruskan data ke dalam **Blade view**.

Daripada menghadirkan kembali metode `render()` ke dalam **component** Anda, Anda dapat menyediakan data tersebut ke **view** melalui **computed properties**:

```php
<?php // resources/views/components/⚡show-posts.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Post::all();
    }
};

```

```blade
<div>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">
            </div>
    @endforeach
</div>

```

---

## Alternatif: Session properties

Jika Anda perlu mempertahankan nilai sederhana saat halaman dimuat ulang tanpa perlu **caching** antar-request, pertimbangkan untuk menggunakan [atribut `#[Session]](https://www.google.com/search?q=/docs/4.x/attribute-session)` alih-alih **computed properties**.

**Session properties** berguna ketika:

* Anda ingin nilai spesifik pengguna tetap ada setelah halaman dimuat ulang (seperti filter pencarian atau preferensi UI).
* Anda tidak ingin nilai tersebut dapat dibagikan melalui URL.
* Nilainya sederhana dan tidak memakan biaya komputasi besar untuk disimpan.

Contohnya, menyimpan kueri pencarian di dalam **session**:

```php
use Livewire\Attributes\Session;

#[Session]
public $search = '';

```

Ini menjaga nilai pencarian tetap ada saat halaman di-*refresh* tanpa menggunakan parameter URL atau **caching** pada **computed property**.

[Learn more about session properties →](https://www.google.com/search?q=/docs/4.x/attribute-session)

---

## See also

* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Pahami pengelolaan properti dasar.
* **[Islands](https://www.google.com/search?q=/docs/4.x/islands)** — Optimalkan performa dengan nilai *lazy computed*.
* **[Computed Attribute](https://www.google.com/search?q=/docs/4.x/attribute-computed)** — Gunakan `#[Computed]` untuk *memoization*.
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Akses *computed properties* di dalam *views*.
