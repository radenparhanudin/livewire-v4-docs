Livewire memungkinkan Anda untuk menyarangkan (**nest**) tambahan **Livewire components** di dalam sebuah **parent component**. Fitur ini sangat kuat, karena memungkinkan Anda untuk menggunakan kembali dan mengenkapsulasi perilaku di dalam **Livewire components** yang dibagikan ke seluruh aplikasi Anda.

> [!warning] Anda mungkin tidak membutuhkan sebuah Livewire component
> Sebelum Anda mengekstrak sebagian dari **template** Anda ke dalam sebuah **nested Livewire component**, tanyakan pada diri sendiri: Apakah konten di dalam **component** ini harus "live"? Jika tidak, kami merekomendasikan Anda untuk membuat sebuah [Blade component](https://laravel.com/docs/blade#components) biasa sebagai gantinya. Hanya buat sebuah **Livewire component** jika **component** tersebut mendapat manfaat dari sifat dinamis Livewire atau jika ada manfaat performa langsung.

> [!tip] Pertimbangkan islands untuk pembaruan yang terisolasi
> Jika Anda ingin mengisolasi perenderan ulang (**re-rendering**) ke wilayah tertentu dari **component** Anda tanpa beban biaya pembuatan **child components** terpisah, pertimbangkan untuk menggunakan [islands](https://www.google.com/search?q=/docs/4.x/islands) sebagai gantinya. **Islands** memungkinkan Anda membuat wilayah yang diperbarui secara independen dalam satu **component** tanpa mengelola **props**, **events**, atau komunikasi **child component**.

Konsultasikan [pemeriksaan teknis mendalam kami tentang Livewire component nesting](https://www.google.com/search?q=/docs/4.x/understanding-nesting) untuk informasi lebih lanjut tentang performa, implikasi penggunaan, dan batasan dari **nested Livewire components**.

## Nesting a component

Untuk menyarangkan sebuah **Livewire component** di dalam sebuah **parent component**, cukup masukkan ke dalam **Blade view** dari **parent component**. Di bawah ini adalah contoh dari **parent component** `dashboard` yang berisi sebuah **nested component** `todos`:

```php
<?php // resources/views/components/⚡dashboard.blade.php

use Livewire\Component;

new class extends Component {
    //
};
?>

<div>
    <h1>Dashboard</h1>

    <livewire:todos /> </div>

```

Pada perenderan awal halaman ini, **component** `dashboard` akan menemukan `<livewire:todos />` dan merendernya di tempat. Pada **network request** berikutnya ke `dashboard`, **nested component** `todos` akan melewati perenderan karena sekarang ia adalah **independent component**-nya sendiri di halaman tersebut. Untuk informasi lebih lanjut tentang konsep teknis di balik **nesting** dan **rendering**, konsultasikan dokumentasi kami tentang mengapa [nested components are independent](https://www.google.com/search?q=/docs/4.x/understanding-nesting%23every-component-is-an-island).

Untuk informasi lebih lanjut tentang sintaks untuk merender **components**, konsultasikan dokumentasi kami tentang [Rendering Components](https://www.google.com/search?q=/docs/4.x/components%23rendering-components).

## Passing props to children

Meneruskan data dari sebuah **parent component** ke sebuah **child component** sangatlah mudah. Faktanya, ini sangat mirip dengan meneruskan **props** ke sebuah [Blade component](https://laravel.com/docs/blade#components) tipikal.

Sebagai contoh, mari kita periksa sebuah **component** `todos` yang meneruskan sebuah koleksi `$todos` ke sebuah **child component** bernama `todo-count`:

```php
<?php // resources/views/components/⚡todos.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    #[Computed]
    public function todos()
    {
        return Auth::user()->todos,
    }
};
?>

<div>
    <livewire:todo-count :todos="$this->todos" />

    </div>

```

Seperti yang Anda lihat, kita meneruskan `$this->todos` ke dalam `todo-count` dengan sintaks: `:todos="$this->todos"`.

Sekarang setelah `$todos` diteruskan ke **child component**, Anda dapat menerima data tersebut melalui metode `mount()` dari **child component**:

```php
<?php // resources/views/components/⚡todo-count.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    public $todos;

    public function mount($todos)
    {
        $this->todos = $todos;
    }

    #[Computed]
    public function count()
    {
        return $this->todos->count(),
    }
};
?>

<div>
    Count: {{ $this->count }}
</div>

```

> [!tip] Abaikan `mount()` sebagai alternatif yang lebih singkat
> Jika metode `mount()` pada contoh di atas terasa seperti kode **boilerplate** yang berlebihan bagi Anda, metode tersebut dapat diabaikan selama nama properti dan nama parameter cocok:
> ```php
> public $todos; // [tl! highlight]
> 
> ```
> 
> 

### Passing static props

Pada contoh sebelumnya, kita meneruskan **props** ke **child component** menggunakan sintaks **dynamic prop** Livewire, yang mendukung ekspresi PHP seperti berikut:

```blade
<livewire:todo-count :todos="$todos" />

```

Namun, terkadang Anda mungkin ingin meneruskan nilai statis sederhana seperti sebuah **string** ke sebuah **component**. Dalam kasus ini, Anda dapat mengabaikan titik dua dari awal pernyataan:

```blade
<livewire:todo-count :todos="$todos" label="Todo Count:" />

```

Nilai **Boolean** dapat diberikan ke **components** hanya dengan menentukan kuncinya. Misalnya, untuk meneruskan variabel `$inline` dengan nilai `true` ke sebuah **component**, kita cukup menempatkan `inline` pada tag **component**:

```blade
<livewire:todo-count :todos="$todos" inline />

```

### Shortened attribute syntax

Saat meneruskan variabel PHP ke dalam sebuah **component**, nama variabel dan nama **prop** seringkali sama. Untuk menghindari penulisan nama dua kali, Livewire memungkinkan Anda untuk cukup mengawali variabel dengan titik dua:

```blade
<livewire:todo-count :todos="$todos" /> <livewire:todo-count :$todos /> ```

## Rendering children in a loop

Saat merender sebuah **child component** di dalam sebuah loop, Anda harus menyertakan nilai `key` yang unik untuk setiap iterasi.

**Component keys** adalah cara Livewire melacak setiap **component** pada perenderan berikutnya, terutama jika sebuah **component** telah dirender atau jika beberapa **components** telah diatur ulang pada halaman.

Anda dapat menentukan **key** dari **component** dengan menentukan sebuah **prop** `:key` pada **child component**:

```blade
<div>
    <h1>Todos</h1>

    @foreach ($todos as $todo)
        <livewire:todo-item :$todo :wire:key="$todo->id" />
    @endforeach
</div>

```

Seperti yang Anda lihat, setiap **child component** akan memiliki **key** unik yang diatur ke ID dari setiap `$todo`. Ini memastikan **key** akan unik dan dilacak jika **todos** diurutkan ulang.

> [!warning] Keys tidak bersifat opsional
> Jika Anda pernah menggunakan **frontend frameworks** seperti Vue atau Alpine, Anda sudah familiar dengan menambahkan **key** ke elemen yang bersarang dalam loop. Namun, dalam framework tersebut, sebuah **key** bukanlah *mandatory*, artinya item akan tetap dirender, tetapi urutan ulang mungkin tidak dilacak dengan benar. Namun, Livewire lebih sangat bergantung pada **keys** dan akan berperilaku tidak benar tanpanya.

## Reactive props

Pengembang yang baru mengenal Livewire mengharapkan bahwa **props** bersifat "reactive" secara **default**. Dengan kata lain, mereka mengharapkan ketika **parent** mengubah nilai **prop** yang diteruskan ke **child component**, maka **child component** tersebut akan diperbarui secara otomatis. Namun, secara **default**, **props** Livewire tidak bersifat **reactive**.

Saat menggunakan Livewire, [setiap component adalah independen](https://www.google.com/search?q=/docs/4.x/understanding-nesting%23every-component-is-an-island). Ini berarti ketika sebuah pembaruan dipicu pada **parent** dan sebuah **network request** dikirimkan, hanya **state** dari **parent component** yang dikirim ke server untuk dirender ulang - bukan milik **child component**. Niat di balik perilaku ini adalah untuk hanya mengirimkan jumlah data minimal bolak-balik antara server dan klien, membuat pembaruan seefisien mungkin.

Tetapi, jika Anda ingin atau butuh sebuah **prop** menjadi **reactive**, Anda dapat dengan mudah mengaktifkan perilaku ini menggunakan parameter atribut `#[Reactive]`.

Sebagai contoh, di bawah ini adalah **template** dari sebuah **parent component** `todos`. Di dalamnya, ia merender sebuah **component** `todo-count` dan meneruskan daftar **todos** saat ini:

```blade
<div>
    <h1>Todos:</h1>

    <livewire:todo-count :$todos />

    </div>

```

Sekarang mari kita tambahkan `#[Reactive]` ke properti `$todos` di dalam **component** `todo-count`. Setelah kita melakukannya, setiap **todos** yang ditambah atau dihapus di dalam **parent component** akan secara otomatis memicu pembaruan di dalam **component** `todo-count`:

```php
<?php // resources/views/components/⚡todo-count.blade.php

use Livewire\Attributes\Reactive;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    #[Reactive] // [tl! highlight]
    public $todos;

    #[Computed]
    public function count()
    {
        return $this->todos->count(),
    }
};
?>

<div>
    Count: {{ $this->count }}
</div>

```

**Reactive properties** adalah fitur yang sangat kuat, membuat Livewire lebih mirip dengan **frontend component libraries** seperti Vue dan React. Namun, penting untuk memahami implikasi performa dari fitur ini dan hanya menambahkan `#[Reactive]` jika memang masuk akal untuk skenario khusus Anda.

> [!tip] Islands dapat meniadakan kebutuhan akan reactive props
> Jika Anda mendapati diri Anda membuat **child components** terutama untuk mengisolasi pembaruan dan menggunakan `#[Reactive]` untuk menjaganya tetap sinkron, pertimbangkan untuk menggunakan [islands](https://www.google.com/search?q=/docs/4.x/islands) sebagai gantinya. **Islands** menyediakan perenderan ulang yang terisolasi dalam satu **component** tanpa perlu **reactive props** atau komunikasi **child component**.

## Binding to child data menggunakan `wire:model`

Pola kuat lainnya untuk berbagi **state** antara **parent** dan **child components** adalah menggunakan `wire:model` secara langsung pada sebuah **child component** melalui fitur `Modelable` milik Livewire.

Perilaku ini sangat umum dibutuhkan saat mengekstrak sebuah elemen **input** ke dalam sebuah **Livewire component** khusus sambil tetap mengakses **state**-nya di **parent component**.

Di bawah ini adalah contoh dari sebuah **parent component** `todos` yang berisi sebuah properti `$todo` yang melacak **todo** saat ini yang akan ditambahkan oleh pengguna:

```php
<?php // resources/views/components/⚡todos.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    public $todo = '';

    public function add()
    {
        Todo::create([
            'content' => $this->pull('todo'),
        ]);
    }

    #[Computed]
    public function todos()
    {
        return Auth::user()->todos,
    }
};

```

Seperti yang Anda lihat di dalam **template** `todos`, `wire:model` digunakan untuk mengikat (**bind**) properti `$todo` secara langsung ke sebuah **nested component** `todo-input`:

```blade
<div>
    <h1>Todos</h1>

    <livewire:todo-input wire:model="todo" /> <button wire:click="add">Add Todo</button>

    <div>
        @foreach ($this->todos as $todo)
            <livewire:todo-item :$todo :wire:key="$todo->id" />
        @endforeach
    </div>
</div>

```

Livewire menyediakan sebuah atribut `#[Modelable]` yang dapat Anda tambahkan ke properti **child component** mana pun untuk menjadikannya *modelable* dari sebuah **parent component**.

Di bawah ini adalah **component** `todo-input` dengan atribut `#[Modelable]` ditambahkan di atas properti `$value` untuk memberi sinyal kepada Livewire bahwa jika `wire:model` dideklarasikan pada **component** oleh sebuah **parent**, ia harus mengikat ke properti ini:

```php
<?php // resources/views/components/⚡todo-input.blade.php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable] // [tl! highlight]
    public $value = '';
};
?>

<div>
    <input type="text" wire:model="value" >
</div>

```

Sekarang **parent component** `todos` dapat memperlakukan `todo-input` seperti elemen **input** lainnya dan mengikat langsung ke nilainya menggunakan `wire:model`.

> [!warning]
> Saat ini Livewire hanya mendukung satu atribut `#[Modelable]`, sehingga hanya atribut pertama yang akan diikat.

## Slots

**Slots** memungkinkan Anda meneruskan konten Blade dari sebuah **parent component** ke dalam sebuah **child component**. Ini berguna ketika sebuah **child component** perlu merender kontennya sendiri sambil tetap mengizinkan **parent** untuk menyuntikkan konten khusus di tempat-tempat tertentu.

Di bawah ini adalah contoh dari sebuah **parent component** yang merender daftar komentar. Setiap komentar dirender oleh sebuah **child component** `Comment`, tetapi **parent** meneruskan sebuah tombol "Remove" melalui sebuah **slot**:

```php
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[Computed]
    public function comments()
    {
        return $this->post->comments;
    }

    public function removeComment($id)
    {
        $this->post->comments()->find($id)->delete();
    }
};
?>

<div>
    @foreach ($this->comments as $comment)
        <livewire:comment :$comment :wire:key="$comment->id">
            <button wire:click="removeComment({{ $comment->id }})">
                Remove
            </button>
        </livewire:comment>
    @endforeach
</div>

```

Sekarang setelah konten tersebut diteruskan ke **child component** `Comment`, Anda dapat merendernya menggunakan variabel `$slot`:

```php
<?php

use Livewire\Component;
use App\Models\Comment;

new class extends Component {
    public Comment $comment;
};
?>

<div>
    <p>{{ $comment->author }}</p>
    <p>{{ $comment->body }}</p>

    {{ $slot }}
</div>

```

Saat **component** `Comment` merender `$slot`, Livewire akan menyuntikkan konten yang diteruskan dari **parent**.

Penting untuk dipahami bahwa **slots** dievaluasi dalam konteks **parent component**. Ini berarti setiap properti atau metode yang direferensikan di dalam **slot** adalah milik **parent**, bukan **child**. Pada contoh di atas, metode `removeComment()` dipanggil pada **parent component**, bukan pada **child** `Comment`.

### Named slots

Selain default slot, Anda juga dapat meneruskan beberapa **named slots** ke dalam sebuah **child component**. Ini berguna ketika Anda ingin menyediakan konten untuk beberapa area dari sebuah **child component**.

Di bawah ini adalah contoh meneruskan sebuah default slot dan sebuah **named slot** `actions` ke **component** `Comment`:

```blade
<div>
    @foreach ($this->comments as $comment)
        <livewire:comment :$comment :wire:key="$comment->id">
            <livewire:slot name="actions">
                <button wire:click="removeComment({{ $comment->id }})">
                    Remove
                </button>
            </livewire:slot>

            <span>Posted on {{ $comment->created_at }}</span>
        </livewire:comment>
    @endforeach
</div>

```

Anda dapat mengakses **named slots** di dalam **child component** dengan meneruskan nama slot ke variabel `$slots`:

```blade
<div>
    <p>{{ $comment->author }}</p>
    <p>{{ $comment->body }}</p>

    <div class="actions">
        {{ $slots['actions'] }}
    </div>

    <div class="metadata">
        {{ $slot }}
    </div>
</div>

```

### Checking if a slot was provided

Anda dapat memeriksa apakah sebuah slot disediakan oleh **parent** menggunakan metode `has()` pada variabel `$slots`. Ini membantu ketika Anda ingin merender konten secara kondisional berdasarkan apakah sebuah slot ada atau tidak:

```blade
<div>
    <p>{{ $comment->author }}</p>
    <p>{{ $comment->body }}</p>

    @if ($slots->has('actions'))
        <div class="actions">
            {{ $slots['actions'] }}
        </div>
    @endif

    {{ $slot }}
</div>

```

## Forwarding HTML attributes

Seperti Blade components, **Livewire components** mendukung penerusan (**forwarding**) atribut HTML dari sebuah **parent** ke sebuah **child** menggunakan variabel `$attributes`.

Di bawah ini adalah contoh dari sebuah **parent component** yang meneruskan atribut `class` ke sebuah **child component**:

```blade
<livewire:comment :$comment class="border-b" />

```

Anda dapat menerapkan atribut-atribut ini di dalam **child component** menggunakan variabel `$attributes`:

```blade
<div {{ $attributes->class('bg-white rounded-md') }}>
    <p>{{ $comment->author }}</p>
    <p>{{ $comment->body }}</p>
</div>

```

Atribut yang cocok dengan nama **public property** secara otomatis diteruskan sebagai **props** dan dikecualikan dari `$attributes`. Atribut apa pun yang tersisa seperti `class`, `id`, atau `data-*` tersedia melalui `$attributes`.

## Islands vs nested components

Saat membangun aplikasi Livewire, Anda akan sering menghadapi pilihan: Haruskah Anda membuat sebuah **nested child component** atau menggunakan sebuah **island**? Kedua pendekatan ini memungkinkan Anda untuk mengisolasi pembaruan pada wilayah tertentu, tetapi keduanya melayani tujuan yang berbeda.

### Kapan menggunakan islands

**Islands** sangat ideal ketika Anda menginginkan isolasi performa tanpa kompleksitas arsitektural. Gunakan **islands** ketika:

**Anda membutuhkan optimasi performa tanpa beban overhead**

Jika tujuan utama Anda adalah mencegah komputasi yang mahal berjalan secara tidak perlu, **islands** adalah solusi yang lebih sederhana:

```blade
{{-- Island: Simple performance isolation --}}
@island
    <div>
        Revenue: {{ $this->expensiveRevenue }}
        <button wire:click="$refresh">Refresh</button>
    </div>
@endisland

```

Ini mencapai manfaat performa yang sama dengan sebuah **child component**, tetapi tanpa membuat file **component** terpisah, mengelola **props**, atau menyiapkan komunikasi **event**.

**Anda ingin menunda (defer) atau lazy load konten**

**Islands** unggul dalam menunda operasi yang mahal hingga setelah pemuatan halaman awal:

```blade
@island(lazy: true)
    <div>{{ $this->slowApiCall }}</div>
@endisland

```

**Anda memiliki beberapa wilayah UI yang independen**

Ketika Anda memiliki beberapa wilayah yang diperbarui secara independen tetapi tidak membutuhkan logika terpisah:

```blade
@island(name: 'stats')
    <div>Stats: {{ $this->stats }}</div>
@endisland

@island(name: 'chart')
    <div>Chart: {{ $this->chartData }}</div>
@endisland

```

**Wilayah terisolasi tersebut tidak membutuhkan lifecycle-nya sendiri**

**Islands** berbagi **lifecycle**, **state**, dan **methods** milik **parent component**. Ini sempurna ketika wilayah tersebut secara konseptual adalah bagian dari **component** yang sama.

### Kapan menggunakan nested components

**Nested components** lebih baik ketika Anda membutuhkan enkapsulasi sejati dan penggunaan kembali (**reusability**). Gunakan **nested components** ketika:

**Anda membutuhkan fungsionalitas yang reusable dan mandiri (self-contained)**

Jika **component** tersebut akan digunakan di berbagai tempat dengan logika dan **state**-nya sendiri:

```blade
{{-- todo-item ini dapat digunakan kembali di seluruh aplikasi --}}
<livewire:todo-item :$todo :wire:key="$todo->id" />

```

**Anda membutuhkan lifecycle hooks yang terpisah**

Ketika **child** membutuhkan `mount()`, `updated()`, atau metode **lifecycle** lainnya sendiri:

```php
public function mount($todo)
{
    $this->authorize('view', $todo);
}

public function updated($property)
{
    // Logika pembaruan khusus child
}

```

**Anda membutuhkan state dan logika yang terenkapsulasi**

Ketika **child** memiliki manajemen **state** yang kompleks yang harus diisolasi:

```php
// Child component dengan state terenkapsulasinya sendiri
public $editMode = false;
public $draft = '';

public function startEdit() { /* ... */ }
public function saveEdit() { /* ... */ }
public function cancelEdit() { /* ... */ }

```

**Anda membutuhkan component tersebut untuk benar-benar independen**

**Nested components** benar-benar independen, mempertahankan **state** mereka sendiri di seluruh pembaruan **parent**. Ini berharga ketika Anda tidak ingin perenderan ulang **parent** memengaruhi **child**.

**Anda sedang membangun sebuah component library**

Saat membuat **components** yang dapat digunakan kembali untuk tim atau organisasi Anda, **nested components** menyediakan batasan enkapsulasi yang tepat.

### Panduan keputusan cepat

Masih belum yakin? Tanyakan pada diri sendiri:

* **"Apakah ini perlu bisa digunakan kembali?"** → **Nested component**
* **"Apakah ini membutuhkan metode lifecycle-nya sendiri?"** → **Nested component**
* **"Apakah saya hanya mencoba mengoptimalkan performa?"** → **Island**
* **"Apakah saya ingin menunda pemuatan konten yang mahal?"** → **Island** (dengan `lazy` atau `defer`)
* **"Apakah ini hanya akan digunakan di satu tempat saja?"** → Mungkin sebuah **island**
* **"Apakah ini membutuhkan state yang kompleks dan terisolasi?"** → **Nested component**

Ingat: Anda selalu dapat memulai dengan sebuah **island** untuk kesederhanaan dan melakukan **refactor** ke sebuah **nested component** kemudian jika Anda membutuhkan enkapsulasi tambahan.

## Listening for events dari children

Teknik komunikasi **parent-child component** kuat lainnya adalah sistem **event** Livewire, yang memungkinkan Anda untuk men-**dispatch** sebuah **event** di server atau klien yang dapat dicegat oleh **components** lain.

Dokumentasi lengkap kami tentang [sistem event Livewire](https://www.google.com/search?q=/docs/4.x/events) menyediakan informasi lebih rinci tentang **events**, tetapi di bawah ini kita akan membahas contoh sederhana menggunakan sebuah **event** untuk memicu pembaruan di dalam sebuah **parent component**.

Pertimbangkan sebuah **component** `todos` dengan fungsionalitas untuk menampilkan dan menghapus **todos**:

```php
<?php // resources/views/components/⚡todos.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    public function remove($todoId)
    {
        $todo = Todo::find($todoId);

        $this->authorize('delete', $todo);

        $todo->delete();
    }

    #[Computed]
    public function todos()
    {
        return Auth::user()->todos,
    }
};
?>

<div>
    @foreach ($this->todos as $todo)
        <livewire:todo-item :$todo :wire:key="$todo->id" />
    @endforeach
</div>

```

Untuk memanggil `remove()` dari dalam **child** `todo-item` **components**, Anda dapat menambahkan sebuah **event listener** ke `todos` melalui atribut `#[On]`:

```php
<?php // resources/views/components/⚡todos.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Attributes\On;
use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    #[On('remove-todo')] // [tl! highlight]
    public function remove($todoId)
    {
        $todo = Todo::find($todoId);

        $this->authorize('delete', $todo);

        $todo->delete();
    }

    #[Computed]
    public function todos()
    {
        return Auth::user()->todos,
    }
};
?>

<div>
    @foreach ($this->todos as $todo)
        <livewire:todo-item :$todo :wire:key="$todo->id" />
    @endforeach
</div>

```

Setelah atribut tersebut ditambahkan ke **action**, Anda dapat men-**dispatch** **event** `remove-todo` dari **child component** `todo-item`:

```php
<?php // resources/views/components/⚡todo-item.blade.php

use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    public Todo $todo;

    public function remove()
    {
        $this->dispatch('remove-todo', todoId: $this->todo->id); // [tl! highlight]
    }
};
?>

<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="remove">Remove</button>
</div>

```

Sekarang ketika tombol "Remove" diklik di dalam sebuah `todo-item`, **parent component** `todos` akan mencegat **event** yang di-**dispatch** tersebut dan melakukan penghapusan **todo**.

Setelah **todo** dihapus di **parent**, daftar akan dirender ulang dan **child** yang men-**dispatch** **event** `remove-todo` akan dihapus dari halaman.

### Meningkatkan performa dengan dispatching client-side

Meskipun contoh di atas berfungsi, ia membutuhkan dua **network requests** untuk melakukan satu tindakan:

1. **Network request** pertama dari **component** `todo-item` memicu **action** `remove`, yang men-**dispatch** **event** `remove-todo`.
2. **Network request** kedua terjadi setelah **event** `remove-todo` di-**dispatch** di sisi klien dan dicegat oleh `todos` untuk memanggil **action** `remove` miliknya.

Anda dapat menghindari **request** pertama sepenuhnya dengan men-**dispatch** **event** `remove-todo` secara langsung di sisi klien (**client-side**). Di bawah ini adalah **component** `todo-item` yang telah diperbarui yang tidak memicu **network request** saat men-**dispatch** **event** `remove-todo`:

```php
<?php // resources/views/components/⚡todo-item.blade.php

use Livewire\Component;
use App\Models\Todo;

new class extends Component {
    public Todo $todo;
};
?>

<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="$dispatch('remove-todo', { todoId: {{ $todo->id }} })">Remove</button>
</div>

```

Sebagai aturan praktis, selalu utamakan men-**dispatch** secara **client-side** jika memungkinkan.

> [!tip] Islands meniadakan beban komunikasi event
> Jika Anda membuat **child components** terutama untuk memicu pembaruan **parent** melalui **events**, pertimbangkan untuk menggunakan [islands](https://www.google.com/search?q=/docs/4.x/islands) sebagai gantinya. **Islands** dapat memanggil metode **component** secara langsung tanpa perantara **events**, karena mereka berbagi konteks **component** yang sama.

Komunikasi melalui **event** menambahkan lapisan ketidaklarangsungan (**indirection**). Seorang **parent** dapat mendengarkan sebuah **event** yang tidak pernah di-**dispatch** dari seorang **child**, dan seorang **child** dapat men-**dispatch** sebuah **event** yang tidak pernah dicegat oleh seorang **parent**.

Ketidaklarangsungan ini terkadang diinginkan; namun, dalam kasus lain Anda mungkin lebih suka mengakses **parent component** secara langsung dari **child component**.

Livewire memungkinkan Anda untuk mencapai hal ini dengan menyediakan variabel **magic** `$parent` ke dalam **Blade template** Anda yang dapat digunakan untuk mengakses **actions** dan **properties** secara langsung dari **child**. Berikut adalah template `TodoItem` di atas yang ditulis ulang untuk memanggil **action** `remove()` secara langsung pada **parent** melalui variabel **magic** `$parent`:

```blade
<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="$parent.remove({{ $todo->id }})">Remove</button>
</div>

```

**Events** dan komunikasi **parent** langsung adalah beberapa cara untuk berkomunikasi bolak-balik antara **parent** dan **child components**. Memahami **tradeoffs** mereka memungkinkan Anda membuat keputusan yang lebih tepat tentang pola mana yang akan digunakan dalam skenario tertentu.

## Dynamic child components

Terkadang, Anda mungkin tidak tahu **child component** mana yang harus dirender pada sebuah halaman hingga saat aplikasi berjalan (**run-time**). Oleh karena itu, Livewire memungkinkan Anda memilih sebuah **child component** pada saat **run-time** melalui `<livewire:dynamic-component ...>`, yang menerima sebuah **prop** `:is`:

```blade
<livewire:dynamic-component :is="$current" />

```

**Dynamic child components** berguna dalam berbagai skenario yang berbeda, namun di bawah ini adalah contoh merender langkah-langkah yang berbeda dalam sebuah **multi-step form** menggunakan **dynamic component**:

```php
<?php // resources/views/components/⚡steps.blade.php

use Livewire\Component;

new class extends Component {
    public $current = 'step-one';

    protected $steps = [
        'step-one',
        'step-two',
        'step-three',
    ];

    public function next()
    {
        $currentIndex = array_search($this->current, $this->steps);

        $this->current = $this->steps[$currentIndex + 1];
    }
};
?>

<div>
    <livewire:dynamic-component :is="$current" :wire:key="$current" />

    <button wire:click="next">Next</button>
</div>

```

Sekarang, jika **prop** `$current` milik **component** `steps` diatur ke "step-one", Livewire akan merender sebuah **component** bernama "step-one" seperti ini:

```php
<?php // resources/views/components/⚡step-one.blade.php

use Livewire\Component;

new class extends Component {
    //
};
?>

<div>
    Step One Content
</div>

```

Jika Anda lebih suka, Anda dapat menggunakan sintaks alternatif:

```blade
<livewire:is :component="$current" :wire:key="$current" />

```

> [!warning]
> Jangan lupa untuk menetapkan **key** yang unik pada setiap **child component**. Meskipun Livewire secara otomatis menghasilkan sebuah **key** untuk `<livewire:dynamic-child />` dan `<livewire:is />`, **key** yang sama tersebut akan berlaku untuk *semua* **child components** Anda, yang berarti perenderan berikutnya akan dilewati.
> Lihat [memaksa sebuah child component untuk merender ulang](https://www.google.com/search?q=%23memaksa-sebuah-child-component-untuk-merender-ulang) untuk pemahaman yang lebih dalam tentang bagaimana **keys** memengaruhi perenderan **component**.

## Recursive components

Meskipun jarang dibutuhkan oleh sebagian besar aplikasi, **Livewire components** dapat disarangkan secara rekursif (**recursively**), artinya sebuah **parent component** merender dirinya sendiri sebagai **child**-nya.

Bayangkan sebuah survei yang berisi sebuah **component** `survey-question` yang dapat memiliki sub-pertanyaan yang terlampir pada dirinya sendiri:

```php
<?php // resources/views/components/⚡survey-question.blade.php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Question;

new class extends Component {
    public Question $question;

    #[Computed]
    public function subQuestions()
    {
        return $this->question->subQuestions,
    }
};
?>

<div>
    Question: {{ $question->content }}

    @foreach ($this->subQuestions as $subQuestion)
        <livewire:survey-question :question="$subQuestion" :wire:key="$subQuestion->id" />
    @endforeach
</div>

```

> [!warning]
> Tentu saja, aturan standar rekursi berlaku untuk **recursive components**. Yang paling penting, Anda harus memiliki logika dalam **template** Anda untuk memastikan **template** tersebut tidak melakukan rekursi tanpa henti. Pada contoh di atas, jika sebuah `$subQuestion` berisi pertanyaan asli sebagai `$subQuestion`-nya sendiri, sebuah **infinite loop** akan terjadi.

## Memaksa sebuah child component untuk merender ulang

Di balik layar, Livewire menghasilkan sebuah **key** untuk setiap **nested Livewire component** di dalam **template**-nya.

Sebagai contoh, pertimbangkan **nested component** `todo-count` berikut:

```blade
<div>
    <livewire:todo-count :$todos />
</div>

```

Livewire secara internal melampirkan sebuah **key** string acak ke **component** tersebut seperti ini:

```blade
<div>
    <livewire:todo-count :$todos wire:key="lska" />
</div>

```

Ketika **parent component** sedang merender dan menemui sebuah **child component** seperti di atas, ia menyimpan **key** tersebut dalam daftar **children** yang terlampir pada **parent**:

```php
'children' => ['lska'],

```

Livewire menggunakan daftar ini sebagai referensi pada perenderan berikutnya untuk mendeteksi apakah sebuah **child component** sudah dirender pada **request** sebelumnya. Jika sudah pernah dirender, **component** tersebut akan dilewati. Ingat, [nested components are independent](https://www.google.com/search?q=/docs/4.x/understanding-nesting%23every-component-is-an-island). Namun, jika **child key** tidak ada dalam daftar, yang berarti belum pernah dirender, Livewire akan membuat instans baru dari **component** tersebut dan merendernya di tempat.

Nuansa-nuansa ini semuanya adalah perilaku di balik layar yang tidak perlu diketahui oleh sebagian besar pengguna; namun, konsep pengaturan **key** pada sebuah **child** adalah alat yang ampuh untuk mengontrol perenderan **child**.

Menggunakan pengetahuan ini, jika Anda ingin memaksa sebuah **component** untuk merender ulang, Anda cukup mengubah **key**-nya.

Di bawah ini adalah contoh di mana kita mungkin ingin menghancurkan dan menginisialisasi ulang **component** `todo-count` jika `$todos` yang diteruskan ke **component** tersebut berubah:

```blade
<div>
    <livewire:todo-count :todos="$todos" :wire:key="$todos->pluck('id')->join('-')" />
</div>

```

Seperti yang Anda lihat di atas, kita menghasilkan sebuah string `:key` dinamis berdasarkan konten dari `$todos`. Dengan cara ini, **component** `todo-count` akan merender dan ada seperti biasa sampai `$todos` itu sendiri berubah. Pada titik itu, **component** akan diinisialisasi ulang sepenuhnya dari awal, dan **component** yang lama akan dibuang.

---

## See also

* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — Berkomunikasi antar **nested components**
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Pelajari tentang merender dan mengorganisir **components**
* **[Islands](https://www.google.com/search?q=/docs/4.x/islands)** — Alternatif untuk **nesting** untuk pembaruan yang terisolasi
* **[Understanding Nesting](https://www.google.com/search?q=/docs/4.x/understanding-nesting)** — Penjelasan mendalam tentang performa dan perilaku **nesting**
* **[Reactive Attribute](https://www.google.com/search?q=/docs/4.x/attribute-reactive)** — Membuat **props** bersifat **reactive** di dalam **nested components**
