Atribut `#[Reactive]` membuat properti pada **child component** diperbarui secara otomatis ketika **parent** mengubah nilai yang dikirimkan ke komponen tersebut.

## Penggunaan dasar

Terapkan atribut `#[Reactive]` pada properti apa pun yang harus bereaksi terhadap perubahan di **parent**:

```php
<?php // resources/views/components/⚡todo-count.blade.php

use Livewire\Attributes\Reactive;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    #[Reactive] // [tl! highlight]
    public $todos;

    #[Computed]
    public function count()
    {
        return $this->todos->count();
    }
};
?>

<div>
    Count: {{ $this->count }}
</div>

```

Sekarang, ketika **parent component** menambah atau menghapus *todos*, **child component** akan diperbarui secara otomatis untuk mencerminkan jumlah yang baru.

---

## Mengapa props tidak reactive secara default

Secara default, **props** Livewire bersifat **tidak reactive**. Ketika sebuah **parent component** diperbarui, hanya *state* milik **parent** yang dikirim ke server—bukan milik **child**. Hal ini dilakukan untuk meminimalkan transfer data dan meningkatkan performa.

Berikut adalah apa yang terjadi tanpa `#[Reactive]`:

```php
<?php // resources/views/components/⚡todos.blade.php

use Livewire\Component;

new class extends Component {
    public $todos = [];

    public function addTodo($text)
    {
        $this->todos[] = ['text' => $text];
        // Child components dengan props $todos tidak akan diperbarui secara otomatis
    }
};
?>

<div>
    <livewire:todo-count :$todos />

    <button wire:click="addTodo('New task')">Add Todo</button>
</div>

```

Tanpa atribut `#[Reactive]` pada properti `$todos` milik **child**, menambahkan *todo* di **parent** tidak akan memperbarui jumlah (*count*) di **child**.

---

## Cara kerjanya

Ketika Anda menambahkan `#[Reactive]`:

1. **Parent** memperbarui properti `$todos` miliknya.
2. **Parent** mengirimkan nilai `$todos` yang baru ke **child** selama proses respons.
3. **Child component** secara otomatis melakukan *re-render* dengan nilai yang baru tersebut.

Ini menciptakan hubungan "reactive" yang serupa dengan *frontend frameworks* seperti Vue atau React.

---

## Pertimbangan Performa

> [!warning] Gunakan reactive properties secukupnya
> **Reactive properties** mengharuskan data tambahan dikirim antara server dan klien pada setiap pembaruan **parent**. Hanya gunakan `#[Reactive]` jika memang diperlukan untuk kasus penggunaan Anda.

**Kapan harus menggunakan:**

* **Child component** menampilkan data yang berubah-ubah di **parent**.
* **Child** perlu tetap sinkron dengan *state* milik **parent**.
* Anda sedang membangun hubungan **parent-child** yang sangat erat (*tightly coupled*).

**Kapan TIDAK boleh menggunakan:**

* Data awal dikirim sekali dan tidak pernah berubah.
* **Child** mengelola *state* independennya sendiri.
* Performa sangat krusial dan pembaruan otomatis tidak terlalu dibutuhkan.

---

## Contoh: Hasil pencarian live (Live search)

Berikut adalah contoh praktis komponen pencarian dengan hasil yang reactive:

```php
<?php // resources/views/components/⚡search.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $query = '';

    public function posts()
    {
        return Post::where('title', 'like', "%{$this->query}%")->get();
    }
};
?>

<div>
    <input type="text" wire:model.live="query" placeholder="Search posts...">

    <livewire:search-results :posts="$this->posts()" /> </div>

```

```php
<?php // resources/views/components/⚡search-results.blade.php

use Livewire\Attributes\Reactive;
use Livewire\Component;

new class extends Component {
    #[Reactive] // [tl! highlight]
    public $posts;
};
?>

<div>
    @foreach($posts as $post)
        <div wire:key="{{ $post->id }}">{{ $post->title }}</div>
    @endforeach
</div>

```

Saat pengguna mengetik, `$posts` di **parent** berubah dan hasil di **child** otomatis diperbarui.

---

## Alternatif: Events

Untuk komponen yang tidak saling bergantung (*loosely coupled*), pertimbangkan untuk menggunakan **events** alih-alih **reactive props**:

```php
// Parent mengirimkan event
$this->dispatch('todos-updated', todos: $this->todos);

// Child mendengarkan event
#[On('todos-updated')]
public function handleTodosUpdate($todos)
{
    $this->todos = $todos;
}

```

**Events** memberikan fleksibilitas lebih, namun memerlukan komunikasi yang eksplisit antar komponen.

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut mengenai komunikasi **parent-child** dan arsitektur komponen, lihat [dokumentasi Nesting Components](https://www.google.com/search?q=/docs/4.x/nesting%23reactive-props).
