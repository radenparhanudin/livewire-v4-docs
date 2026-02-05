Atribut `#[On]` memungkinkan sebuah **component** untuk mendengarkan *events* dan mengeksekusi sebuah metode ketika *events* tersebut dikirimkan (*dispatched*).

## Penggunaan dasar

Terapkan atribut `#[On]` pada metode apa pun yang harus dipanggil saat sebuah *event* dikirimkan:

```php
<?php // resources/views/components/⚡dashboard.blade.php

use Livewire\Attributes\On;
use Livewire\Component;

new class extends Component {
    #[On('post-created')] // [tl! highlight]
    public function updatePostList($title)
    {
        session()->flash('status', "New post created: {$title}");
    }
};

```

Ketika **component** lain mengirimkan *event* `post-created`, metode `updatePostList()` akan dipanggil secara otomatis.

---

## Mengirimkan events (Dispatching events)

Untuk mengirimkan *event* yang memicu pendengar (*listeners*), gunakan metode `dispatch()`:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public function save()
    {
        $post = Post::create(['title' => $this->title]);

        $this->dispatch('post-created', title: $post->title); // [tl! highlight]

        return redirect('/posts');
    }
};

```

*Event* `post-created` akan memicu metode apa pun yang didekorasi dengan `#[On('post-created')]`.

---

## Mengirim data ke listeners

*Events* dapat mengirimkan data sebagai parameter bernama (*named parameters*):

```php
// Mengirim dengan beberapa parameter
$this->dispatch('post-updated', id: $post->id, title: $post->title);

```

```php
// Mendengarkan dan menerima parameter
#[On('post-updated')]
public function handlePostUpdate($id, $title)
{
    // Gunakan $id dan $title...
}

```

---

## Nama event dinamis

Anda dapat menggunakan properti **component** dalam nama *event* untuk pendengaran yang bersifat spesifik (*scoped*):

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\On;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[On('post-updated.{post.id}')] // [tl! highlight]
    public function refreshPost()
    {
        $this->post->refresh();
    }
};

```

Jika `$post->id` bernilai `3`, ini hanya akan mendengarkan *event* `post-updated.3`, dan mengabaikan pembaruan pada postingan lainnya.

---

## Beberapa event listeners

Satu metode dapat mendengarkan beberapa *event* sekaligus:

```php
#[On('post-created')]
#[On('post-updated')]
#[On('post-deleted')]
public function refreshStats()
{
    // Perbarui statistik ketika ada postingan yang berubah
}

```

---

## Mendengarkan event browser

Anda juga dapat mendengarkan *event* browser yang dikirimkan dari JavaScript:

```php
#[On('user-logged-in')]
public function handleUserLogin()
{
    // Tangani login...
}

```

```javascript
// Dari JavaScript
window.dispatchEvent(new CustomEvent('user-logged-in'));

```

---

## Alternatif: Mendengarkan di dalam template

Alih-alih menggunakan atribut, Anda dapat mendengarkan *events* langsung pada **child components** di dalam template Blade Anda:

```blade
<livewire:post.edit @saved="$refresh" />

```

Ini akan mendengarkan *event* `saved` dari **child component** `post.edit` dan me-*refresh* **parent** saat *event* tersebut dikirimkan.

Anda juga dapat memanggil metode tertentu:

```blade
<livewire:post.edit @saved="handleSave($event.id)" />

```

---

## Kapan harus menggunakan

Gunakan `#[On]` ketika:

* Satu **component** perlu bereaksi terhadap tindakan di **component** lain.
* Mengimplementasikan notifikasi atau pembaruan waktu nyata (*real-time*).
* Membangun komponen yang tidak saling bergantung (*loosely coupled*) namun berkomunikasi via *events*.
* Mendengarkan *event* browser atau Laravel Echo.
* Memperbarui data ketika terjadi perubahan eksternal.

---

## Contoh: Notifikasi real-time

Berikut adalah contoh praktis dari ikon lonceng notifikasi yang mendengarkan notifikasi baru:

```php
<?php // resources/views/components/⚡notification-bell.blade.php

use Livewire\Attributes\On;
use Livewire\Component;

new class extends Component {
    public $unreadCount = 0;

    public function mount()
    {
        $this->unreadCount = auth()->user()->unreadNotifications()->count();
    }

    #[On('notification-sent')] // [tl! highlight]
    public function incrementCount()
    {
        $this->unreadCount++;
    }

    #[On('notifications-read')] // [tl! highlight]
    public function resetCount()
    {
        $this->unreadCount = 0;
    }
};
?>

<button class="relative">
    <svg></svg>
    @if($unreadCount > 0)
        <span class="absolute -top-1 -right-1 bg-red-500 text-white rounded-full px-2 py-1 text-xs">
            {{ $unreadCount }}
        </span>
    @endif
</button>

```

---

## Referensi

```php
#[On(
    string $event,
)]

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$event` | `string` | *required* | Nama *event* yang ingin didengarkan |
