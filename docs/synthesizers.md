Karena komponen Livewire didehidrasi (**dehydrated**) atau diserialisasi menjadi JSON, kemudian dihidrasi (**hydrated**) atau di-unserialisasi kembali menjadi komponen PHP di antara setiap permintaan (*requests*), properti di dalamnya harus bersifat *JSON-serializable*.

Secara bawaan, PHP menyerialisasi sebagian besar nilai primitif ke JSON dengan mudah. Namun, agar komponen Livewire dapat mendukung tipe properti yang lebih canggih (seperti **models**, **collections**, **carbon instances**, dan **stringables**), diperlukan sistem yang lebih kokoh.

Oleh karena itu, Livewire menyediakan titik ekstensi yang disebut "**Synthesizers**" yang memungkinkan pengguna untuk mendukung tipe properti kustom apa pun yang mereka inginkan.

> [!tip] Pastikan Anda memahami hidrasi terlebih dahulu
> Sebelum menggunakan **Synthesizers**, sangat disarankan untuk memahami sepenuhnya sistem hidrasi Livewire. Anda dapat mempelajari lebih lanjut dengan membaca [dokumentasi hidrasi](https://www.google.com/search?q=/docs/4.x/hydration).

---

## Memahami Synthesizers

Sebelum mengeksplorasi pembuatan **Synthesizer** kustom, mari kita lihat terlebih dahulu **Synthesizer** internal yang digunakan Livewire untuk mendukung [Laravel Stringables](https://laravel.com/docs/strings).

Misalkan aplikasi Anda berisi komponen `CreatePost` berikut:

```php
class CreatePost extends Component
{
    public $title = '';
}

```

Di antara permintaan, Livewire mungkin menyerialisasi **state** komponen ini ke dalam objek JSON seperti berikut:

```js
state: { title: '' },

```

Sekarang, pertimbangkan contoh yang lebih lanjut di mana nilai properti `$title` adalah sebuah **stringable**, bukan string biasa:

```php
class CreatePost extends Component
{
    public $title = '';

    public function mount()
    {
        $this->title = str($this->title);
    }
}

```

JSON hasil dehidrasi yang merepresentasikan **state** komponen ini sekarang berisi sebuah [metadata tuple](https://www.google.com/search?q=/docs/4.x/hydration%23deeply-nested-tuples) alih-alih string kosong biasa:

```js
state: { title: ['', { s: 'str' }] },

```

Livewire sekarang dapat menggunakan **tuple** ini untuk menghidrasi kembali properti `$title` menjadi **stringable** pada permintaan berikutnya.

Setelah Anda melihat efek luar-dalam dari **Synthesizers**, berikut adalah kode sumber aktual untuk **stringable synth** internal Livewire:

```php
use Illuminate\Support\Stringable;

class StringableSynth extends Synth
{
    public static $key = 'str';

    public static function match($target)
    {
        return $target instanceof Stringable;
    }

    public function dehydrate($target)
    {
        return [$target->__toString(), []];
    }

    public function hydrate($value)
    {
        return str($value);
    }
}

```

Mari kita bedah bagian demi bagian.

Pertama adalah properti `$key`:

```php
public static $key = 'str';

```

Setiap **synth** harus berisi properti statis `$key` yang digunakan Livewire untuk mengubah [metadata tuple](https://www.google.com/search?q=/docs/4.x/hydration%23deeply-nested-tuples) seperti `['', { s: 'str' }]` kembali menjadi **stringable**. Seperti yang Anda perhatikan, setiap **metadata tuple** memiliki kunci `s` yang mereferensikan kunci ini.

Sebaliknya, ketika Livewire mendehidrasi sebuah properti, ia akan menggunakan fungsi statis `match()` milik **synth** untuk mengidentifikasi apakah **Synthesizer** ini merupakan kandidat yang tepat untuk mendehidrasi properti saat ini (`$target` adalah nilai properti saat ini):

```php
public static function match($target)
{
    return $target instanceof Stringable;
}

```

Jika `match()` mengembalikan nilai `true`, metode `dehydrate()` akan digunakan untuk mengambil nilai PHP properti tersebut sebagai input dan mengembalikan **metadata tuple** yang dapat di-JSON-kan:

```php
public function dehydrate($target)
{
    return [$target->__toString(), []];
}

```

Kemudian, pada awal permintaan berikutnya, setelah **Synthesizer** ini dicocokkan oleh kunci `{ s: 'str' }` dalam **tuple**, metode `hydrate()` akan dipanggil dan diberikan representasi JSON mentah dari properti tersebut, dengan harapan metode ini mengembalikan nilai yang kompatibel dengan PHP untuk ditetapkan kembali ke properti.

```php
public function hydrate($value)
{
    return str($value);
}

```

---

## Mendaftarkan Synthesizer kustom

Untuk mendemonstrasikan bagaimana Anda dapat menulis **Synthesizer** sendiri untuk mendukung properti kustom, kita akan menggunakan contoh komponen `UpdateProperty` berikut:

```php
class UpdateProperty extends Component
{
    public Address $address;

    public function mount()
    {
        $this->address = new Address();
    }
}

```

Berikut adalah kode sumber untuk kelas `Address`:

```php
namespace App\Dtos\Address;

class Address
{
    public $street = '';
    public $city = '';
    public $state = '';
    public $zip = '';
}

```

Untuk mendukung properti bertipe `Address`, kita dapat menggunakan **Synthesizer** berikut:

```php
use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }
}

```

Agar tersedia secara global di aplikasi Anda, Anda dapat menggunakan metode `propertySynthesizer` Livewire untuk mendaftarkan **synthesizer** tersebut dari metode `boot` di **service provider** Anda:

```php
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::propertySynthesizer(AddressSynth::class);
    }
}

```

## Mendukung data binding

Menggunakan contoh `UpdateProperty` dari atas, kemungkinan besar Anda ingin mendukung **binding** `wire:model` secara langsung ke properti dari objek `Address`. **Synthesizers** memungkinkan Anda untuk mendukung hal ini menggunakan metode `get()` dan `set()`:

```php
use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }

    public function get(&$target, $key) // [tl! highlight:8]
    {
        return $target->{$key};
    }

    public function set(&$target, $key, $value)
    {
        $target->{$key} = $value;
    }
}

```
