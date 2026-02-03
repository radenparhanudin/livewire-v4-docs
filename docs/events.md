Livewire menawarkan sistem **event** yang kokoh yang dapat Anda gunakan untuk berkomunikasi antara **components** yang berbeda pada halaman. Karena ia menggunakan **browser events** di balik layar, Anda juga dapat menggunakan sistem **event** Livewire untuk berkomunikasi dengan **Alpine components** atau bahkan JavaScript murni (*plain vanilla JavaScript*).

Untuk memicu sebuah **event**, Anda dapat menggunakan metode `dispatch()` dari mana saja di dalam **component** Anda dan mendengarkan (**listen**) **event** tersebut dari **component** lain mana pun pada halaman.

## Dispatching events

Untuk melakukan **dispatch** sebuah **event** dari sebuah **Livewire component**, Anda dapat memanggil metode `dispatch()`, dengan meneruskan nama **event** dan data tambahan apa pun yang ingin Anda kirimkan bersama dengan **event** tersebut.

Di bawah ini adalah contoh melakukan **dispatch** **event** `post-created` dari sebuah **component** `post.create`:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public function save()
    {
        // ...

        $this->dispatch('post-created'); // [tl! highlight]
    }
};

```

Dalam contoh ini, ketika metode `dispatch()` dipanggil, **event** `post-created` akan di-**dispatched**, dan setiap **component** lain pada halaman yang sedang mendengarkan (**listening**) **event** ini akan diberitahu.

Anda dapat meneruskan data tambahan dengan **event** tersebut dengan meneruskan data sebagai parameter kedua ke metode `dispatch()`:

```php
$this->dispatch('post-created', title: $post->title);

```

---

## Listening for events

Untuk mendengarkan (**listen**) sebuah **event** dalam sebuah **Livewire component**, tambahkan atribut `#[On]` di atas metode yang Anda inginkan untuk dipanggil ketika sebuah **event** yang diberikan di-**dispatched**:

> [!warning] Pastikan Anda mengimpor attribute classes
> Pastikan Anda mengimpor **attribute classes** apa pun. Sebagai contoh, atribut `#[On()]` di bawah ini membutuhkan impor berikut: `use Livewire\Attributes\On;`.

```php
<?php // resources/views/components/⚡dashboard.blade.php

use Livewire\Component;
use Livewire\Attributes\On; // [tl! highlight]

new class extends Component {
    #[On('post-created')] // [tl! highlight]
    public function updatePostList($title)
    {
        // ...
    }
};

```

Sekarang, ketika **event** `post-created` di-**dispatched** dari `post.create`, sebuah **network request** akan dipicu dan **action** `updatePostList()` akan dipanggil.

Seperti yang Anda lihat, data tambahan yang dikirim dengan **event** akan diberikan ke **action** sebagai argumen pertamanya.

### Listening for dynamic event names

Sesekali, Anda mungkin ingin menghasilkan nama **event listener** secara dinamis pada saat **run-time** menggunakan data dari **component** Anda.

Sebagai contoh, jika Anda ingin membatasi (**scope**) sebuah **event listener** ke sebuah **Eloquent model** yang spesifik, Anda dapat menambahkan ID milik **model** tersebut ke nama **event** saat melakukan **dispatch** seperti berikut:

```php
<?php // resources/views/components/post/⚡edit.blade.php

use Livewire\Component;

new class extends Component {
    public function update()
    {
        // ...

        $this->dispatch("post-updated.{$post->id}"); // [tl! highlight]
    }
};

```

Dan kemudian mendengarkan (**listen**) untuk **model** spesifik tersebut:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\On; // [tl! highlight]
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[On('post-updated.{post.id}')] // [tl! highlight]
    public function refreshPost()
    {
        // ...
    }
};

```

Jika **model** `$post` di atas memiliki ID `3`, metode `refreshPost()` hanya akan dipicu oleh sebuah **event** bernama: `post-updated.3`.

### Listening for events from specific child components

Livewire memungkinkan Anda untuk mendengarkan (**listen**) **events** secara langsung pada masing-masing **child components** di dalam **Blade template** Anda seperti berikut:

```blade
<div>
    <livewire:edit-post @saved="$refresh">

    </div>

```

Dalam skenario di atas, jika **child component** `edit-post` melakukan **dispatch** sebuah **event** `saved`, maka `$refresh` milik **parent** akan dipanggil dan **parent** akan di-**refreshed**.

Alih-alih meneruskan `$refresh`, Anda dapat meneruskan metode apa pun yang biasanya Anda lakukan pada sesuatu seperti `wire:click`. Berikut adalah contoh memanggil sebuah metode `close()` yang mungkin melakukan sesuatu seperti menutup sebuah **modal dialog**:

```blade
<livewire:edit-post @saved="close">

```

Jika **child** melakukan **dispatched** parameter bersama dengan **request** tersebut, sebagai contoh `$this->dispatch('saved', postId: 1)`, Anda dapat meneruskan nilai-nilai tersebut ke metode **parent** menggunakan sintaks berikut:

```blade
<livewire:edit-post @saved="close($event.detail.postId)">

```

---

## Using JavaScript to interact with events

Sistem **event** Livewire menjadi jauh lebih kuat ketika Anda berinteraksi dengannya dari JavaScript di dalam aplikasi Anda. Ini membuka kemampuan bagi JavaScript lain apa pun di aplikasi Anda untuk berkomunikasi dengan **Livewire components** pada halaman.

### Listening for events inside component scripts

Anda dapat dengan mudah mendengarkan (**listen**) **event** `post-created` di dalam **template component** Anda dari sebuah tag `<script>` seperti berikut:

```html
<script>
    this.$on('post-created', () => {
        //
    });
</script>

```

Cuplikan di atas akan mendengarkan (**listen**) untuk `post-created` dari **component** tempat ia didaftarkan. Jika **component** tersebut sudah tidak ada lagi di halaman, **event listener** tidak akan lagi dipicu.

### Dispatching events from component scripts

Selain itu, Anda dapat melakukan **dispatch** **events** dari dalam tag `<script>` milik **component** seperti berikut:

```html
<script>
    this.$dispatch('post-created');
</script>

```

Ketika skrip di atas dijalankan, **event** `post-created` akan di-**dispatched** ke **component** tempat ia didefinisikan.

Untuk melakukan **dispatch** **event** hanya ke **component** tempat skrip tersebut berada dan tidak ke **components** lain pada halaman (mencegah **event** agar tidak "**bubbling**" ke atas), Anda dapat menggunakan `dispatchSelf()`:

```js
this.$dispatchSelf('post-created');

```

Anda dapat meneruskan parameter tambahan apa pun ke **event** dengan meneruskan sebuah objek sebagai argumen kedua ke `dispatch()`:

```html
<script>
    this.$dispatch('post-created', { refreshPosts: true });
</script>

```

Anda sekarang dapat mengakses parameter **event** tersebut dari kelas Livewire Anda dan juga **JavaScript event listeners** lainnya.

Berikut adalah contoh menerima parameter `refreshPosts` di dalam sebuah kelas Livewire:

```php
use Livewire\Attributes\On;

// ...

#[On('post-created')]
public function handleNewPost($refreshPosts = false)
{
    //
}

```

Anda juga dapat mengakses parameter `refreshPosts` dari sebuah **JavaScript event listener** dari properti `detail` milik **event**:

```html
<script>
    this.$on('post-created', (event) => {
        let refreshPosts = event.detail.refreshPosts

        // ...
    });
</script>

```

---

## Listening for Livewire events from global JavaScript

Sebagai alternatif, Anda dapat mendengarkan (**listen**) **Livewire events** secara global menggunakan `Livewire.on` dari skrip apa pun di dalam aplikasi Anda:

```html
<script>
    document.addEventListener('livewire:init', () => {
       Livewire.on('post-created', (event) => {
           //
       });
    });
</script>

```

Cuplikan di atas akan mendengarkan (**listen**) untuk **event** `post-created` yang di-**dispatched** dari **component** mana pun pada halaman.

Jika Anda ingin menghapus **event listener** ini karena alasan apa pun, Anda dapat melakukannya menggunakan fungsi `cleanup` yang dikembalikan:

```html
<script>
    document.addEventListener('livewire:init', () => {
        let cleanup = Livewire.on('post-created', (event) => {
            //
        });

        // Memanggil "cleanup()" akan menghapus pendaftaran event listener di atas...
        cleanup();
    });
</script>

```

---

## Events in Alpine

Karena **Livewire events** adalah **plain browser events** biasa di balik layar, Anda dapat menggunakan **Alpine** untuk mendengarkan (**listen**) mereka atau bahkan melakukan **dispatch**.

### Listening for Livewire events in Alpine

Sebagai contoh, kita mungkin dengan mudah mendengarkan (**listen**) untuk **event** `post-created` menggunakan **Alpine**:

```blade
<div x-on:post-created="..."></div>

```

Cuplikan di atas akan mendengarkan (**listen**) untuk **event** `post-created` dari **Livewire components** mana pun yang merupakan **children** dari elemen HTML tempat direktif `x-on` ditetapkan.

Untuk mendengarkan (**listen**) **event** tersebut dari **Livewire component** mana pun pada halaman, Anda dapat menambahkan `.window` pada **listener**:

```blade
<div x-on:post-created.window="..."></div>

```

Jika Anda ingin mengakses data tambahan yang dikirimkan bersama dengan **event**, Anda dapat melakukannya menggunakan `$event.detail`:

```blade
<div x-on:post-created="notify('New post: ' + $event.detail.title)"></div>

```

### Dispatching Livewire events from Alpine

**Event** apa pun yang di-**dispatched** dari **Alpine** mampu dicegat (**intercepted**) oleh sebuah **Livewire component**.

Sebagai contoh, kita mungkin dengan mudah melakukan **dispatch** **event** `post-created` dari **Alpine**:

```blade
<button x-on:click="$dispatch('post-created')">...</button>

```

Seperti metode `dispatch()` milik Livewire, Anda dapat meneruskan data tambahan bersama dengan **event** dengan meneruskan data tersebut sebagai parameter kedua ke metode tersebut:

```blade
<button x-on:click="$dispatch('post-created', { title: 'Post Title' })">...</button>

```

---

## Dispatching directly to another component

Jika Anda ingin menggunakan **events** untuk berkomunikasi secara langsung antara dua **components** pada halaman, Anda dapat menggunakan **modifier** `dispatch()->to()`.

Di bawah ini adalah contoh dari **component** `post.create` yang melakukan **dispatch** **event** `post-created` secara langsung ke **component** `dashboard`, melewati **components** lain mana pun yang sedang mendengarkan (**listening**) **event** spesifik tersebut:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public function save()
    {
        // ...

        $this->dispatch('post-created')->to(component: Dashboard::class);
    }
};

```

## Dispatching a component event to itself

Menggunakan **modifier** `dispatch()->self()`, Anda dapat membatasi sebuah **event** agar hanya dicegat (**intercepted**) oleh **component** tempat **event** tersebut dipicu:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;

new class extends Component {
    public function save()
    {
        // ...

        $this->dispatch('post-created')->to(self: true);
    }
};

```

---

## Dispatching events from Blade templates

Anda dapat melakukan **dispatch** **events** secara langsung dari **Blade templates** Anda menggunakan fungsi JavaScript `$dispatch`. Ini berguna ketika Anda ingin memicu sebuah **event** dari sebuah interaksi pengguna, seperti klik tombol:

```blade
<button wire:click="$dispatch('show-post-modal', { id: {{ $post->id }} })">
    EditPost
</button>

```

Dalam contoh ini, ketika tombol diklik, **event** `show-post-modal` akan di-**dispatched** dengan data yang ditentukan.

Jika Anda ingin melakukan **dispatch** sebuah **event** secara langsung ke **component** lain Anda dapat menggunakan fungsi JavaScript `$dispatchTo()`:

```blade
<button wire:click="$dispatchTo('posts', 'show-post-modal', { id: {{ $post->id }} })">
    EditPost
</button>

```

---

## Testing dispatched events

Untuk menguji **events** yang di-**dispatched** oleh **component** Anda, gunakan metode `assertDispatched()` di dalam **Livewire test** Anda. Metode ini memeriksa bahwa sebuah **event** spesifik telah di-**dispatched** selama siklus hidup **component**:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Livewire\CreatePost;
use Livewire\Livewire;

class CreatePostTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_dispatches_post_created_event()
    {
        Livewire::test(CreatePost::class)
            ->call('save')
            ->assertDispatched('post-created');
    }
}

```

### Testing Event Listeners

Untuk menguji **event listeners**, Anda dapat melakukan **dispatch** **events** dari lingkungan pengujian dan menegaskan (**assert**) bahwa **actions** yang diharapkan dilakukan sebagai respons terhadap **event** tersebut:

```php
public function test_it_updates_post_count_when_a_post_is_created()
{
    Livewire::test(Dashboard::class)
        ->assertSee('Posts created: 0')
        ->dispatch('post-created')
        ->assertSee('Posts created: 1');
}

```

---

## Real-time events using Laravel Echo

Livewire berpasangan dengan baik dengan [Laravel Echo](https://laravel.com/docs/broadcasting#client-side-installation) untuk menyediakan fungsionalitas **real-time** pada halaman web Anda menggunakan **WebSockets**.

> [!warning] Menginstal Laravel Echo adalah sebuah prasyarat
> Fitur ini mengasumsikan Anda telah menginstal **Laravel Echo** dan objek `window.Echo` tersedia secara global di dalam aplikasi Anda.

### Mendengarkan Echo events

Bayangkan Anda memiliki sebuah **event** di dalam aplikasi Laravel Anda bernama `OrderShipped`:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public Order $order;

    public function broadcastOn()
    {
        return new Channel('orders');
    }
}

```

Anda mungkin melakukan **dispatch** **event** ini dari bagian lain aplikasi Anda seperti berikut:

```php
use App\Events\OrderShipped;

OrderShipped::dispatch();

```

Jika Anda mendengarkan (**listen**) **event** ini di JavaScript hanya menggunakan **Laravel Echo**, tampilannya akan seperti ini:

```js
Echo.channel('orders')
    .listen('OrderShipped', e => {
        console.log(e.order)
    })

```

Dengan asumsi Anda telah menginstal dan mengonfigurasi **Laravel Echo**, Anda dapat mendengarkan (**listen**) **event** ini dari dalam sebuah **Livewire component**.

Di bawah ini adalah contoh dari **component** `order-tracker` yang sedang mendengarkan (**listening**) **event** `OrderShipped` untuk menunjukkan indikasi visual pesanan baru kepada pengguna:

```php
<?php // resources/views/components/⚡order-tracker.blade.php

use Livewire\Attributes\On; // [tl! highlight]
use Livewire\Component;

new class extends Component {
    public $showNewOrderNotification = false;

    #[On('echo:orders,OrderShipped')]
    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }

    // ...
};

```

Jika Anda memiliki **Echo channels** dengan variabel yang tertanam di dalamnya (seperti Order ID), Anda dapat mendefinisikan **listeners** melalui metode `getListeners()` alih-alih atribut `#[On]`:

```php
<?php // resources/views/components/⚡order-tracker.blade.php

use Livewire\Attributes\On; // [tl! highlight]
use Livewire\Component;
use App\Models\Order;

new class extends Component {
    public Order $order;

    public $showOrderShippedNotification = false;

    public function getListeners()
    {
        return [
            "echo:orders.{$this->order->id},OrderShipped" => 'notifyShipped',
        ];
    }

    public function notifyShipped()
    {
        $this->showOrderShippedNotification = true;
    }

    // ...
};

```

Atau, jika Anda lebih suka, Anda dapat menggunakan sintaks nama **event** dinamis:

```php
#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder()
{
    $this->showNewOrderNotification = true;
}

```

Jika Anda perlu mengakses **event payload**, Anda dapat melakukannya melalui parameter `$event` yang diteruskan:

```php
#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder($event)
{
    $order = Order::find($event['orderId']);

    //
}

```

---

### Customizing broadcast event names dengan `broadcastAs()`

Secara **default**, Laravel melakukan **broadcast** **events** menggunakan nama **event class**. Namun, Anda dapat menyesuaikan nama **broadcast event** dengan mengimplementasikan metode `broadcastAs()` di dalam **event class** Anda.

Sebagai contoh, jika Anda memiliki **event** `ScoreSubmitted` tetapi ingin mem-**broadcast**-nya sebagai `score.submitted`:

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class ScoreSubmitted implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function broadcastOn()
    {
        return new Channel('scores');
    }

    public function broadcastAs(): string
    {
        return 'score.submitted';
    }
}

```

Saat mendengarkan (**listening**) **event** ini di dalam **Livewire component**, Anda harus menggunakan nama **broadcast** kustom yang dikembalikan oleh `broadcastAs()` alih-alih nama **class**. **Penting:** Saat menggunakan nama **broadcast** kustom, Anda harus mengawalinya dengan titik (`.`) untuk membedakannya dari nama **namespaced event class**. Ini adalah sebuah konvensi **Laravel Echo**:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\On;
use Livewire\Component;

class ScoreBoard extends Component
{
    public $scores = [];

    #[On('echo:scores,.score.submitted')]
    public function handleScoreSubmitted($event)
    {
        $this->scores[] = $event['score'];
    }
}

```

Dalam contoh di atas, **Livewire component** mendengarkan `.score.submitted` (nama **broadcast** kustom yang diawali dengan titik) daripada `ScoreSubmitted` (nama **class**). Awalan titik memberi tahu **Laravel Echo** untuk tidak menambahkan **namespace** aplikasi (`App\Events`) ke nama **event**.

Anda juga dapat menggunakan nama **broadcast** kustom dengan nama **dynamic channel**:

```php
#[On('echo:scores.{game.id},.score.submitted')]
public function handleScoreSubmitted($event)
{
    $this->scores[] = $event['score'];
}

```

---

### Private & presence channels

Anda juga dapat mendengarkan (**listen**) **events** yang di-**broadcast** ke **private** dan **presence channels**:

> [!info]
> Sebelum melanjutkan, pastikan Anda telah menentukan [Authentication Callbacks](https://laravel.com/docs/master/broadcasting#defining-authorization-callbacks) untuk **broadcast channels** Anda.

```php
<?php // resources/views/components/⚡order-tracker.blade.php

use Livewire\Component;

new class extends Component {
    public $showNewOrderNotification = false;

    public function getListeners()
    {
        return [
            // Public Channel
            "echo:orders,OrderShipped" => 'notifyNewOrder',

            // Private Channel
            "echo-private:orders,OrderShipped" => 'notifyNewOrder',

            // Presence Channel
            "echo-presence:orders,OrderShipped" => 'notifyNewOrder',
            "echo-presence:orders,here" => 'notifyNewOrder',
            "echo-presence:orders,joining" => 'notifyNewOrder',
            "echo-presence:orders,leaving" => 'notifyNewOrder',
        ];
    }

    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }
};

```

---

## See also

* **[Nesting](https://www.google.com/search?q=/docs/4.x/nesting)** — Berkomunikasi antara **parent** dan **child components**
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Memicu **events** dari **component actions**
* **[Alpine](https://www.google.com/search?q=/docs/4.x/alpine)** — Melakukan **dispatch** dan mendengarkan (**listen**) **events** dengan **Alpine**
* **[On Attribute](https://www.google.com/search?q=/docs/4.x/attribute-on)** — Mendengarkan (**listen**) **events** menggunakan atribut `#[On]`
