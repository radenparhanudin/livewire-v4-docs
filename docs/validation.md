Livewire bertujuan untuk membuat proses memvalidasi input pengguna dan memberikan umpan balik kepada mereka semenyenangkan mungkin. Dengan membangun di atas fitur validasi Laravel, Livewire memanfaatkan pengetahuan Anda yang sudah ada sekaligus memberi Anda fitur tambahan yang kuat seperti validasi *real-time*.

Berikut adalah contoh komponen `CreatePost` yang menunjukkan alur kerja validasi paling dasar di Livewire:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        $validated = $this->validate([ // [tl! highlight:3]
            'title' => 'required|min:3',
            'content' => 'required|min:3',
        ]);

        Post::create($validated);

        return redirect()->to('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}

```

```blade
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>@error('title') {{ $message }} @enderror</div>

    <textarea wire:model="content"></textarea>
    <div>@error('content') {{ $message }} @enderror</div>

    <button type="submit">Save</button>
</form>

```

Seperti yang Anda lihat, Livewire menyediakan metode `validate()` yang dapat Anda panggil untuk memvalidasi properti komponen Anda. Metode ini mengembalikan set data yang telah divalidasi sehingga Anda dapat memasukkannya dengan aman ke dalam database.

Di sisi *frontend*, Anda dapat menggunakan direktif Blade milik Laravel yang sudah ada untuk menampilkan pesan validasi kepada pengguna Anda.

Untuk informasi lebih lanjut, lihat [dokumentasi Laravel tentang merender kesalahan validasi di Blade](https://laravel.com/docs/blade#validation-errors).

## Validate attributes

Jika Anda lebih suka menempatkan aturan validasi komponen langsung bersama propertinya, Anda dapat menggunakan atribut `#[Validate]` dari Livewire.

Dengan menghubungkan aturan validasi dengan properti menggunakan `#[Validate]`, Livewire akan secara otomatis menjalankan aturan validasi properti tersebut sebelum setiap pembaruan (*update*). Namun, Anda tetap harus menjalankan `$this->validate()` sebelum menyimpan data ke database agar properti yang belum diperbarui juga ikut divalidasi.

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3')] // [tl! highlight]
    public $title = '';

    #[Validate('required|min:3')] // [tl! highlight]
    public $content = '';

    public function save()
    {
        $this->validate();

        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }

    // ...
}

```

> [!info] Atribut Validate tidak mendukung objek Rule
> PHP Attributes dibatasi pada sintaks tertentu seperti string biasa dan array. Jika Anda ingin menggunakan sintaks *run-time* seperti objek Rule Laravel (`Rule::exists(...)`), Anda harus [mendefinisikan metode `rules()](https://www.google.com/search?q=%23defining-a-rules-method)` di komponen Anda.
> Pelajari lebih lanjut di dokumentasi tentang [menggunakan objek Laravel Rule dengan Livewire](https://www.google.com/search?q=%23using-laravel-rule-objects).

Jika Anda menginginkan kontrol lebih besar kapan properti divalidasi, Anda dapat meneruskan parameter `onUpdate: false` ke atribut `#[Validate]`. Ini akan menonaktifkan validasi otomatis dan berasumsi Anda ingin memvalidasi properti secara manual menggunakan metode `$this->validate()`:

```php
#[Validate('required|min:3', onUpdate: false)]
public $title = '';

```

### Nama atribut kustom

Jika Anda ingin menyesuaikan nama atribut yang dimasukkan ke dalam pesan validasi, Anda dapat menggunakan parameter `as: `:

```php
use Livewire\Attributes\Validate;

#[Validate('required', as: 'tanggal lahir')]
public $dob;

```

Ketika validasi gagal pada potongan kode di atas, Laravel akan menggunakan "tanggal lahir" alih-alih "dob" sebagai nama bidang dalam pesan validasi. Pesan yang dihasilkan akan menjadi "Isian tanggal lahir wajib diisi" alih-alih "Isian dob wajib diisi".

### Pesan validasi kustom

Untuk melewati pesan validasi bawaan Laravel dan menggantinya dengan pesan Anda sendiri, Anda dapat menggunakan parameter `message: ` di atribut `#[Validate]`:

```php
#[Validate('required', message: 'Mohon masukkan judul postingan')]
public $title;

```

Jika Anda ingin menambahkan pesan yang berbeda untuk aturan yang berbeda, Anda cukup memberikan beberapa atribut `#[Validate]`:

```php
#[Validate('required', message: 'Mohon masukkan judul postingan')]
#[Validate('min:3', message: 'Judul ini terlalu pendek')]
public $title;

```

### Keluar dari lokalisasi (Localization)

Secara default, pesan aturan dan atribut Livewire dilokalisasi menggunakan pembantu terjemahan Laravel: `trans()`.

Anda dapat memilih untuk tidak menggunakan lokalisasi dengan meneruskan parameter `translate: false` ke atribut `#[Validate]`:

```php
#[Validate('required', message: 'Please provide a post title', translate: false)]
public $title;

```

### Key kustom

Saat menerapkan aturan validasi langsung ke properti menggunakan atribut `#[Validate]`, Livewire berasumsi bahwa *validation key* haruslah nama properti itu sendiri. Namun, terkadang Anda mungkin ingin menyesuaikan *validation key* tersebut.

Contohnya, Anda mungkin ingin memberikan aturan validasi terpisah untuk properti array dan elemen-elemen di dalamnya. Dalam kasus ini, alih-alih meneruskan aturan validasi sebagai argumen pertama ke atribut `#[Validate]`, Anda dapat meneruskan array pasangan *key-value*:

```php
#[Validate([
    'todos' => 'required',
    'todos.*' => [
        'required',
        'min:3',
        new Uppercase,
    ],
])]
public $todos = [];

```

## Form objects

Seiring bertambahnya properti dan aturan validasi dalam sebuah komponen Livewire, komponen tersebut bisa mulai terasa terlalu penuh. Untuk meringankan hal ini dan menyediakan abstraksi yang membantu penggunaan kembali kode, Anda dapat menggunakan *Form Objects* Livewire untuk menyimpan properti dan aturan validasi Anda.

Di bawah ini adalah contoh `CreatePost` yang sama, tetapi sekarang properti dan aturannya telah dipisahkan ke *form object* khusus bernama `PostForm`:

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:3')]
    public $title = '';

    #[Validate('required|min:3')]
    public $content = '';
}

```

`PostForm` di atas sekarang dapat didefinisikan sebagai properti pada komponen `CreatePost`:

```php
class CreatePost extends Component
{
    public PostForm $form;

    public function save()
    {
        Post::create(
            $this->form->all()
        );

        return redirect()->to('/posts');
    }
}

```

Seperti yang Anda lihat, alih-alih mencantumkan setiap properti satu per satu, kita dapat mengambil semua nilai properti menggunakan metode `->all()` pada objek form.

Selain itu, saat mereferensikan nama properti di template, Anda harus menambahkan awalan `form.` pada setiap instansi:

```blade
<input type="text" wire:model="form.title">
<div>@error('form.title') {{ $message }} @enderror</div>

```

Saat menggunakan *form objects*, validasi atribut `#[Validate]` akan dijalankan setiap kali properti diperbarui. Namun, jika Anda menonaktifkan perilaku ini dengan menentukan `onUpdate: false` pada atribut, Anda dapat menjalankan validasi objek form secara manual menggunakan `$this->form->validate()`.

*Form objects* adalah abstraksi yang berguna untuk kumpulan data yang lebih besar dan memiliki berbagai fitur tambahan yang membuatnya lebih kuat. Untuk informasi lebih lanjut, lihat [dokumentasi form object](https://www.google.com/search?q=/docs/4.x/forms%23extracting-a-form-object) yang komprehensif.

## Real-time validation

*Real-time validation* adalah istilah yang digunakan ketika Anda memvalidasi input pengguna saat mereka mengisi formulir, alih-alih menunggu hingga formulir tersebut dikirimkan (*submission*).

Dengan menggunakan atribut `#[Validate]` langsung pada properti Livewire, setiap kali ada *network request* yang dikirim untuk memperbarui nilai properti di server, aturan validasi yang diberikan akan langsung diterapkan.

Ini berarti untuk menyediakan pengalaman *real-time validation* kepada pengguna pada input tertentu, tidak diperlukan pekerjaan *backend* tambahan. Satu-satunya hal yang diperlukan adalah menggunakan `wire:model.live` atau `wire:model.live.blur` untuk menginstruksikan Livewire agar memicu *network request* saat kolom diisi.

Pada contoh di bawah ini, `wire:model.live.blur` telah ditambahkan ke input teks. Sekarang, ketika pengguna mengetik di kolom tersebut lalu menekan tab atau mengklik ke luar kolom, *network request* akan dipicu dengan nilai terbaru dan aturan validasi akan dijalankan:

```blade
<form wire:submit="save">
    <input type="text" wire:model.live.blur="title">

    </form>

```

Jika Anda menggunakan metode `rules()` untuk mendeklarasikan aturan validasi properti alih-alih menggunakan atribut `#[Validate]`, Anda tetap bisa menyertakan atribut `#[Validate]` tanpa parameter untuk mempertahankan perilaku *real-time validation*:

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate] // [tl! highlight]
    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => 'required|min:5',
            'content' => 'required|min:5',
        ];
    }

    public function save()
    {
        $validated = $this->validate();

        Post::create($validated);

        return redirect()->to('/posts');
    }
}

```

Sekarang, pada contoh di atas, meskipun `#[Validate]` kosong, ia akan memberitahu Livewire untuk menjalankan validasi kolom yang disediakan oleh `rules()` setiap kali properti diperbarui.

## Menyesuaikan pesan kesalahan (Customizing error messages)

Secara bawaan, Laravel menyediakan pesan validasi yang masuk akal seperti "The title field is required." jika properti `$title` memiliki aturan `required`.

Namun, Anda mungkin perlu menyesuaikan bahasa dari pesan kesalahan ini agar lebih sesuai dengan aplikasi dan pengguna Anda.

### Nama atribut kustom

Terkadang properti yang Anda validasi memiliki nama yang tidak cocok untuk ditampilkan kepada pengguna. Misalnya, jika Anda memiliki kolom database bernama `dob` yang merupakan singkatan dari "Date of birth", Anda tentu ingin menampilkan "The date of birth field is required" alih-alih "The dob field is required".

Livewire memungkinkan Anda menentukan nama alternatif untuk properti menggunakan parameter `as: `:

```php
use Livewire\Attributes\Validate;

#[Validate('required', as: 'tanggal lahir')]
public $dob = '';

```

Kini, jika aturan validasi `required` gagal, pesan kesalahan akan menyatakan "The tanggal lahir field is required." (atau padanannya dalam bahasa Indonesia jika lokalisasi aktif).

### Pesan kustom

Jika menyesuaikan nama properti saja tidak cukup, Anda dapat menyesuaikan seluruh pesan validasi menggunakan parameter `message: `:

```php
use Livewire\Attributes\Validate;

#[Validate('required', message: 'Mohon isi tanggal lahir Anda.')]
public $dob = '';

```

Jika Anda memiliki beberapa aturan dan ingin menyesuaikan pesan untuk masing-masing aturan, direkomendasikan untuk menggunakan atribut `#[Validate]` yang terpisah untuk setiap aturan, seperti ini:

```php
use Livewire\Attributes\Validate;

#[Validate('required', message: 'Mohon masukkan judul.')]
#[Validate('min:5', message: 'Judul Anda terlalu pendek.')]
public $title = '';

```

Jika Anda lebih suka menggunakan sintaks array pada atribut `#[Validate]`, Anda dapat menentukan atribut dan pesan kustom seperti berikut:

```php
use Livewire\Attributes\Validate;

#[Validate([
    'titles' => 'required',
    'titles.*' => 'required|min:5',
], message: [
    'required' => ':attribute tidak boleh kosong.',
    'titles.required' => ':attribute belum ada yang diisi.',
    'min' => ':attribute terlalu pendek.',
], attribute: [
    'titles.*' => 'judul',
])]
public $titles = [];

```

## Mendefinisikan metode `rules()`

Sebagai alternatif dari atribut `#[Validate]`, Anda dapat mendefinisikan metode di dalam komponen Anda yang disebut `rules()` dan mengembalikan daftar kolom beserta aturan validasinya. Ini sangat membantu jika Anda ingin menggunakan sintaks *run-time* yang tidak didukung dalam PHP Attributes, misalnya objek aturan Laravel seperti `Rule::password()`.

Aturan-aturan ini kemudian akan diterapkan saat Anda menjalankan `$this->validate()` di dalam komponen. Anda juga dapat mendefinisikan fungsi `messages()` dan `validationAttributes()`.

Berikut contohnya:

```php
use Livewire\Component;
use App\Models\Post;
use Illuminate\Validation\Rule;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    protected function rules() // [tl! highlight:6]
    {
        return [
            'title' => Rule::exists('posts', 'title'),
            'content' => 'required|min:3',
        ];
    }

    protected function messages() // [tl! highlight:6]
    {
        return [
            'content.required' => ':attribute harus diisi.',
            'content.min' => ':attribute terlalu pendek.',
        ];
    }

    protected function validationAttributes() // [tl! highlight:6]
    {
        return [
            'content' => 'deskripsi',
        ];
    }

    public function save()
    {
        $this->validate();

        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }
}

```

> [!warning] Metode `rules()` tidak memvalidasi saat pembaruan data
> Saat mendefinisikan aturan melalui metode `rules()`, Livewire HANYA akan menggunakan aturan validasi ini untuk memvalidasi properti saat Anda menjalankan `$this->validate()`. Ini berbeda dengan atribut `#[Validate]` standar yang diterapkan setiap kali sebuah kolom diperbarui melalui sesuatu seperti `wire:model`. Untuk menerapkan aturan validasi ini ke properti setiap kali diperbarui, Anda tetap bisa menggunakan `#[Validate]` tanpa parameter tambahan.

> [!warning] Jangan berkonflik dengan mekanisme Livewire
> Saat menggunakan utilitas validasi Livewire, komponen Anda **tidak boleh** memiliki properti atau metode bernama `rules`, `messages`, `validationAttributes`, atau `validationCustomValues`, kecuali Anda sedang menyesuaikan proses validasi tersebut. Jika tidak, nama-nama tersebut akan berkonflik dengan mekanisme internal Livewire.

## Menggunakan objek Laravel Rule

Objek `Rule` Laravel adalah cara yang sangat ampuh untuk menambahkan perilaku validasi tingkat lanjut ke formulir Anda.

Berikut adalah contoh penggunaan objek `Rule` bersama dengan metode `rules()` Livewire untuk mencapai validasi yang lebih canggih:

```php
<?php

namespace App\Livewire;

use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class UpdatePost extends Form
{
    public ?Post $post;

    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => [
                'required',
                Rule::unique('posts')->ignore($this->post), // [tl! highlight]
            ],
            'content' => 'required|min:5',
        ];
    }

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->validate(); // [tl! highlight]

        $this->post->update($this->all());

        $this->reset();
    }
}

```

## Mengontrol validation errors secara manual

Utilitas validasi Livewire seharusnya sudah menangani skenario validasi yang paling umum; namun, ada kalanya Anda mungkin ingin kontrol penuh atas pesan validasi di dalam **component** Anda.

Berikut adalah semua metode yang tersedia untuk memanipulasi validation errors di dalam **component** Livewire Anda:

| Metode | Deskripsi |
| --- | --- |
| `$this->addError([key], [message])` | Menambahkan pesan validasi secara manual ke dalam **error bag** |
| `$this->resetValidation([?key])` | Mereset validation errors untuk **key** yang diberikan, atau mereset semua **errors** jika tidak ada **key** yang disertakan |
| `$this->getErrorBag()` | Mengambil **Laravel error bag** dasar yang digunakan di dalam **component** Livewire |

> [!info] Menggunakan `$this->addError()` dengan Form Objects
> Saat menambahkan **errors** secara manual menggunakan `$this->addError` di dalam sebuah **form object**, **key** akan secara otomatis diberi awalan (*prefix*) nama properti tempat **form** tersebut ditetapkan pada **parent component**. Contohnya, jika di dalam **Component** Anda menetapkan **form** ke properti bernama `$data`, maka **key** akan menjadi `data.key`.

## Mengakses instance validator

Terkadang Anda mungkin ingin mengakses **instance Validator** yang digunakan Livewire secara internal di dalam metode `validate()`. Hal ini memungkinkan dengan menggunakan metode `withValidator`. **Closure** yang Anda berikan menerima **validator** yang telah dikonstruksi sepenuhnya sebagai argumen, memungkinkan Anda memanggil metode apa pun sebelum aturan validasi benar-benar dievaluasi.

Berikut adalah contoh pencegatan (**intercepting**) **validator** internal Livewire untuk memeriksa kondisi secara manual dan menambahkan pesan validasi tambahan:

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3')]
    public $title = '';

    #[Validate('required|min:3')]
    public $content = '';

    public function boot()
    {
        $this->withValidator(function ($validator) {
            $validator->after(function ($validator) {
                if (str($this->title)->startsWith('"')) {
                    $validator->errors()->add('title', 'Judul tidak boleh diawali dengan tanda kutip');
                }
            });
        });
    }

    public function save()
    {
        Post::create($this->all());

        return redirect()->to('/posts');
    }

    // ...
}

```

## Menggunakan custom validators

Jika Anda ingin menggunakan sistem validasi Anda sendiri di Livewire, itu tidak masalah. Livewire akan menangkap setiap pengecualian `ValidationException` yang dilempar di dalam **components** dan menyediakan **errors** ke dalam **view** sama seperti jika Anda menggunakan metode `validate()` milik Livewire sendiri.

Berikut adalah contoh **component** `CreatePost`, tetapi alih-alih menggunakan fitur validasi Livewire, sebuah **validator** yang benar-benar kustom dibuat dan diterapkan pada properti **component**:

```php
use Illuminate\Support\Facades\Validator;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        $validated = Validator::make(
            // Data untuk divalidasi...
            ['title' => $this->title, 'content' => $this->content],

            // Aturan validasi yang diterapkan...
            ['title' => 'required|min:3', 'content' => 'required|min:3'],

            // Pesan validasi kustom...
            ['required' => 'Isian :attribute wajib diisi'],
         )->validate();

        Post::create($validated);

        return redirect()->to('/posts');
    }

    // ...
}

```

## Testing validation

Livewire menyediakan utilitas pengujian (**testing utilities**) yang berguna untuk skenario validasi, seperti metode `assertHasErrors()`.

Berikut adalah kasus uji (*test case*) dasar yang memastikan validation errors muncul jika tidak ada input yang diatur untuk properti `title`:

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_cant_create_post_without_title()
    {
        Livewire::test(CreatePost::class)
            ->set('content', 'Sample content...')
            ->call('save')
            ->assertHasErrors('title');
    }
}

```

Selain menguji keberadaan **errors**, `assertHasErrors` memungkinkan Anda untuk mempersempit asersi ke aturan tertentu dengan meneruskan aturan yang ingin ditegaskan sebagai argumen kedua:

```php
public function test_cant_create_post_with_title_shorter_than_3_characters()
{
    Livewire::test(CreatePost::class)
        ->set('title', 'Sa')
        ->set('content', 'Sample content...')
        ->call('save')
        ->assertHasErrors(['title' => ['min:3']]);
}

```

Anda juga dapat menegaskan keberadaan validation errors untuk beberapa properti secara bersamaan:

```php
public function test_cant_create_post_without_title_and_content()
{
    Livewire::test(CreatePost::class)
        ->call('save')
        ->assertHasErrors(['title', 'content']);
}

```

Untuk informasi lebih lanjut tentang utilitas pengujian lainnya yang disediakan oleh Livewire, silakan pelajari [dokumentasi testing](https://www.google.com/search?q=/docs/4.x/testing).

## Mengakses errors di JavaScript

Livewire menyediakan **magic property** `$errors` untuk akses **validation errors** di sisi klien (*client-side*):

```blade
<form wire:submit="save">
    <input type="email" wire:model="email">

    <div wire:show="$errors.has('email')">
        <span wire:text="$errors.first('email')"></span>
    </div>

    <button type="submit">Save</button>
</form>

```

### Metode yang tersedia

* `$errors.has('field')` - Memeriksa apakah suatu kolom memiliki **errors**
* `$errors.first('field')` - Mendapatkan pesan **error** pertama untuk suatu kolom
* `$errors.get('field')` - Mendapatkan semua pesan **error** untuk suatu kolom
* `$errors.all()` - Mendapatkan semua **errors** untuk semua kolom
* `$errors.clear()` - Menghapus semua **errors**
* `$errors.clear('field')` - Menghapus **errors** untuk kolom tertentu

Saat menggunakan Alpine.js, akses `$errors` melalui `$wire.$errors`.

## Atribut `#[Rule]` yang sudah usang (Deprecated)

Saat Livewire v3 pertama kali diluncurkan, ia menggunakan istilah "Rule" alih-alih "Validate" untuk atribut validasinya (`#[Rule]`).

Karena konflik penamaan dengan **Laravel rule objects**, istilah ini telah diubah menjadi `#[Validate]`. Keduanya masih didukung di Livewire v3, namun sangat direkomendasikan agar Anda mengubah semua penggunaan `#[Rule]` menjadi `#[Validate]` agar tetap mutakhir.

---

## See also

* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Validasi input form dengan umpan balik *real-time*
* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Validasi nilai properti sebelum disimpan
* **[Validate Attribute](https://www.google.com/search?q=/docs/4.x/attribute-validate)** — Menggunakan `#[Validate]` untuk validasi properti
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Validasi data di dalam metode **action**
