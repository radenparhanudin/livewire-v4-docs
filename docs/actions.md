Livewire **actions** adalah **methods** pada **component** Anda yang dapat dipicu oleh interaksi *frontend* seperti mengklik tombol atau mengirimkan formulir. Mereka memberikan pengalaman pengembangan berupa kemampuan untuk memanggil **PHP method** secara langsung dari browser, sehingga Anda dapat fokus pada logika aplikasi tanpa harus menulis kode *boilerplate* yang menghubungkan *frontend* dan *backend* aplikasi Anda.

Mari kita pelajari contoh dasar pemanggilan **action** `save`:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }
};
?>

<form wire:submit="save"> 
    <input type="text" wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>
</form>

```

Pada contoh di atas, saat pengguna mengirimkan formulir dengan mengklik "Save", `wire:submit` akan mencegat *event* `submit` dan memanggil **action** `save()` di server.

Intinya, **actions** adalah cara untuk memetakan interaksi pengguna ke fungsionalitas sisi server dengan mudah tanpa repot mengirimkan dan menangani permintaan AJAX secara manual.

## Passing parameters

Livewire memungkinkan Anda mengirimkan parameter dari Blade **template** ke **actions** di dalam **component**, memberikan Anda kesempatan untuk menyediakan data atau **state** tambahan dari *frontend* saat **action** dipanggil.

Sebagai contoh, bayangkan Anda memiliki **component** `ShowPosts` yang memungkinkan pengguna menghapus postingan. Anda dapat mengirimkan ID postingan sebagai parameter ke **action** `delete()` di Livewire **component** Anda. Kemudian, **action** tersebut dapat mengambil postingan yang relevan dan menghapusnya dari database:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function delete($id)
    {
        $post = Post::findOrFail($id);

        $this->authorize('delete', $post);

        $post->delete();
    }
};

```

```blade
<div>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button> 
        </div>
    @endforeach
</div>

```

Untuk postingan dengan ID 2, tombol "Delete" pada Blade **template** di atas akan di-*render* di browser sebagai:

```blade
<button wire:click="delete(2)">Delete</button>

```

Saat tombol ini diklik, **method** `delete()` akan dipanggil dan `$id` akan dikirimkan dengan nilai "2".

> [!warning] Jangan mempercayai parameter action
> Parameter **action** harus diperlakukan sama seperti *input* permintaan HTTP, yang berarti nilai parameter **action** tidak boleh dipercayai begitu saja. Anda harus selalu melakukan otorisasi kepemilikan suatu entitas sebelum memperbaruinya di database.
> Untuk informasi lebih lanjut, pelajari dokumentasi kami mengenai [masalah keamanan dan praktik terbaik](https://www.google.com/search?q=/docs/4.x/actions%23security-concerns).

Sebagai kenyamanan tambahan, Anda dapat secara otomatis me-*resolve* **Eloquent models** melalui ID model terkait yang diberikan ke **action** sebagai parameter. Ini sangat mirip dengan [route model binding](https://www.google.com/search?q=/docs/4.x/components%23using-route-model-binding). Untuk memulainya, berikan **type-hint** pada parameter **action** dengan **model class**, dan model yang sesuai akan secara otomatis diambil dari database dan dikirimkan ke **action**, alih-alih hanya berupa ID:

```php
public function delete(Post $post) 
{
    $this->authorize('delete', $post);

    $post->delete();
}

```

## Dependency injection

Anda dapat memanfaatkan sistem [dependency injection Laravel](https://laravel.com/docs/controllers#dependency-injection-and-controllers) dengan memberikan **type-hint** pada parameter di *signature* **action** Anda. Livewire dan Laravel akan secara otomatis me-*resolve* dependensi **action** tersebut dari **container**:

```php
public function delete(PostRepository $posts, $postId) 
{
    $posts->deletePost($postId);
}

```

## Event listeners

Livewire mendukung berbagai macam **event listeners**, memungkinkan Anda untuk merespons berbagai jenis interaksi pengguna:

| Listener | Deskripsi |
| --- | --- |
| `wire:click` | Dipicu saat sebuah elemen diklik |
| `wire:submit` | Dipicu saat sebuah formulir dikirimkan |
| `wire:keydown` | Dipicu saat sebuah tombol keyboard ditekan |
| `wire:keyup` | Dipicu saat sebuah tombol keyboard dilepaskan |
| `wire:mouseenter` | Dipicu saat mouse memasuki area elemen |
| `wire:*` | Teks apa pun setelah `wire:` akan digunakan sebagai nama *event* listener |

Karena nama *event* setelah `wire:` bisa berupa apa saja, Livewire mendukung *event* browser apa pun yang mungkin perlu Anda dengarkan. Misalnya, untuk mendengarkan `transitionend`, Anda dapat menggunakan `wire:transitionend`.

### Listening for specific keys

Anda dapat menggunakan salah satu alias praktis Livewire untuk menyaring *event listener* penekanan tombol ke tombol tertentu atau kombinasi tombol.

Contohnya, untuk melakukan pencarian saat pengguna menekan `Enter` setelah mengetik di kotak pencarian, Anda dapat menggunakan `wire:keydown.enter`:

```blade
<input wire:model="query" wire:keydown.enter="searchPosts">

```

Anda dapat merantai lebih banyak alias tombol untuk mendengarkan kombinasi tombol. Misalnya, jika Anda ingin mendengarkan tombol `Enter` hanya saat tombol `Shift` juga ditekan, Anda dapat menulis sebagai berikut:

```blade
<input wire:keydown.shift.enter="...">

```

Berikut adalah daftar semua **key modifiers** yang tersedia: `.shift`, `.enter`, `.space`, `.ctrl`, `.cmd`, `.meta`, `.alt`, `.up`, `.down`, `.left`, `.right`, `.escape`, `.tab`, `.caps-lock`, `.equal`, `.period`, `.slash`.

### Event handler modifiers

Livewire juga menyertakan pengubah (*modifiers*) yang membantu untuk membuat tugas penanganan *event* umum menjadi sangat mudah.

Misalnya, jika Anda perlu memanggil `event.preventDefault()` dari dalam *event listener*, Anda dapat menambahkan akhiran `.prevent` pada nama *event*:

```blade
<input wire:keydown.prevent="...">

```

Daftar lengkap pengubah *event listener* yang tersedia meliputi: `.prevent`, `.stop`, `.window`, `.outside`, `.document`, `.once`, `.debounce`, `.throttle`, `.self`, `.camel`, `.dot`, `.passive`, `.capture`.

Karena `wire:` menggunakan direktif `x-on` milik [Alpine](https://alpinejs.dev) di balik layar, pengubah ini disediakan untuk Anda oleh Alpine.

### Menangani event pihak ketiga

Livewire juga mendukung pendengaran (*listening*) terhadap *custom events* yang dipicu oleh pustaka (*library*) pihak ketiga.

Sebagai contoh, bayangkan Anda menggunakan editor teks kaya [Trix](https://trix-editor.org/) dalam proyek Anda dan ingin mendengarkan *event* `trix-change` untuk menangkap konten editor. Anda dapat mencapainya menggunakan direktif `wire:trix-change`:

```blade
<form wire:submit="save">
    <trix-editor
        wire:trix-change="setPostContent($event.target.value)"
    ></trix-editor>

    </form>

```

Dalam contoh ini, *action* `setPostContent` dipanggil setiap kali *event* `trix-change` dipicu, memperbarui properti `content` di komponen Livewire dengan nilai saat ini dari editor Trix.

> [!info] Anda dapat mengakses objek event menggunakan `$event`
> Di dalam *event handler* Livewire, Anda dapat mengakses objek *event* melalui `$event`. Ini berguna untuk mereferensikan informasi pada *event* tersebut. Misalnya, Anda dapat mengakses elemen yang memicu *event* melalui `$event.target`.

> [!warning]
> Kode demo Trix di atas belum lengkap dan hanya berguna sebagai demonstrasi *event listener*. Jika digunakan apa adanya, permintaan jaringan akan dikirim pada setiap ketukan tombol. Implementasi yang lebih berperforma adalah:
> ```blade
> <trix-editor
>    x-on:trix-change="$wire.content = $event.target.value"
> ></trix-editor>
> 
> ```
> 
> 

### Mendengarkan custom events yang dikirimkan (dispatched)

Jika aplikasi Anda mengirimkan (*dispatch*) *custom events* dari Alpine, Anda juga dapat mendengarkannya menggunakan Livewire:

```blade
<div wire:custom-event="...">

    <button x-on:click="$dispatch('custom-event')">...</button>

</div>

```

Ketika tombol diklik pada contoh di atas, *event* `custom-event` dikirimkan dan naik (*bubbles up*) ke akar komponen Livewire di mana `wire:custom-event` menangkapnya dan memanggil *action* yang ditentukan.

Jika Anda ingin mendengarkan *event* yang dikirimkan di tempat lain di aplikasi Anda, Anda harus menunggu hingga *event* tersebut naik ke objek `window` dan mendengarkannya di sana. Untungnya, Livewire memudahkan hal ini dengan mengizinkan Anda menambahkan pengubah `.window` sederhana ke *event listener* apa pun:

```blade
<div wire:custom-event.window="...">
    </div>

<button x-on:click="$dispatch('custom-event')">...</button>

```

### Menonaktifkan input saat formulir sedang dikirim

Perhatikan contoh `CreatePost` yang kita bahas sebelumnya:

```blade
<form wire:submit="save">
    <input wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>
</form>

```

Ketika pengguna mengklik "Save", permintaan jaringan dikirim ke server untuk memanggil *action* `save()` pada komponen Livewire.

Namun, bayangkan jika pengguna mengisi formulir ini dengan koneksi internet yang lambat. Pengguna mengklik "Save" dan awalnya tidak terjadi apa-apa karena permintaan jaringan memakan waktu lebih lama dari biasanya. Mereka mungkin bertanya-tanya apakah pengiriman gagal dan mencoba mengklik tombol "Save" lagi saat permintaan pertama masih ditangani.

Dalam kasus ini, akan ada dua permintaan untuk *action* yang sama yang diproses secara bersamaan.

Untuk mencegah skenario ini, Livewire secara otomatis menonaktifkan tombol *submit* dan semua input formulir di dalam elemen `<form>` saat *action* `wire:submit` sedang diproses. Ini memastikan bahwa formulir tidak terkirim dua kali secara tidak sengaja.

Untuk lebih mengurangi kebingungan bagi pengguna dengan koneksi lambat, sering kali sangat membantu untuk menunjukkan indikator pemuatan (*loading indicator*) seperti perubahan warna latar belakang yang halus atau animasi SVG.

Livewire menyediakan direktif `wire:loading` yang memudahkan penampilan dan penyembunyian indikator pemuatan di mana saja pada halaman. Berikut adalah contoh singkat penggunaan `wire:loading` untuk menampilkan pesan pemuatan di bawah tombol "Save":

```blade
<form wire:submit="save">
    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>

    <span wire:loading>Saving...</span> 
</form>

```

Alternatifnya, Anda dapat mengatur gaya status pemuatan secara langsung menggunakan Tailwind dan atribut otomatis `data-loading` milik Livewire:

```blade
<form wire:submit="save">
    <textarea wire:model="content"></textarea>

    <button type="submit" class="data-loading:opacity-50">Save</button>

    <span class="not-data-loading:hidden">Saving...</span>
</form>

```

Untuk sebagian besar kasus, menggunakan selektor `data-loading` lebih sederhana dan lebih fleksibel daripada `wire:loading`. [Pelajari lebih lanjut tentang loading states →](https://www.google.com/search?q=/docs/4.x/loading-states)

## Refreshing a component (Menyegarkan komponen)

Terkadang Anda mungkin ingin memicu "penyegaran" (*refresh*) sederhana pada komponen Anda. Misalnya, jika Anda memiliki komponen yang memeriksa status sesuatu di database, Anda mungkin ingin menampilkan tombol bagi pengguna untuk menyegarkan hasil yang ditampilkan.

Anda dapat melakukan ini menggunakan *action* `$refresh` sederhana dari Livewire di mana pun Anda biasanya mereferensikan metode komponen Anda sendiri:

```blade
<button type="button" wire:click="$refresh">...</button>

```

Ketika *action* `$refresh` dipicu, Livewire akan melakukan perjalanan pulang-pergi ke server (*server-roundtrip*) dan merender ulang komponen Anda tanpa memanggil metode apa pun.

Penting untuk dicatat bahwa pembaruan data yang tertunda di komponen Anda (misalnya *binding* `wire:model`) akan diterapkan di server saat komponen disegarkan.

Anda juga dapat memicu penyegaran komponen menggunakan AlpineJS di dalam komponen Livewire Anda:

```blade
<button type="button" x-on:click="$wire.$refresh()">...</button>

```

Pelajari lebih lanjut dengan membaca [dokumentasi penggunaan Alpine di dalam Livewire](https://www.google.com/search?q=/docs/4.x/alpine).

## Konfirmasi sebuah action

Saat mengizinkan pengguna untuk melakukan tindakan berbahaya—seperti menghapus postingan dari database—Anda mungkin ingin menunjukkan kepada mereka peringatan konfirmasi untuk memverifikasi bahwa mereka benar-benar ingin melakukan tindakan tersebut.

Livewire memudahkan hal ini dengan menyediakan direktif sederhana yang disebut `wire:confirm`:

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm="Apakah Anda yakin ingin menghapus postingan ini?"
>
    Delete post
</button>

```

Ketika `wire:confirm` ditambahkan ke elemen yang berisi *action* Livewire, saat pengguna mencoba memicu tindakan tersebut, mereka akan disuguhkan dialog konfirmasi yang berisi pesan yang diberikan. Mereka dapat menekan "OK" untuk mengonfirmasi tindakan, atau menekan "Cancel" atau menekan tombol escape.

Untuk informasi lebih lanjut, kunjungi [halaman dokumentasi `wire:confirm](https://www.google.com/search?q=/docs/4.x/wire-confirm)`.

## Memanggil actions dari Alpine

Livewire terintegrasi secara mulus dengan [Alpine](https://alpinejs.dev/). Faktanya, di balik layar, setiap komponen Livewire juga merupakan komponen Alpine. Ini berarti Anda dapat memanfaatkan sepenuhnya Alpine di dalam komponen Anda untuk menambahkan interaktivitas sisi klien yang didukung oleh JavaScript.

Untuk membuat pasangan ini lebih kuat, Livewire mengekspos objek ajaib `$wire` ke Alpine yang dapat diperlakukan sebagai representasi JavaScript dari komponen PHP Anda. Selain [mengakses dan mengubah properti publik via `$wire](https://www.google.com/search?q=/docs/4.x/properties%23accessing-properties-from-javascript)`, Anda dapat memanggil *actions*. Ketika sebuah *action* dipanggil pada objek `$wire`, metode PHP yang sesuai akan dipanggil pada komponen Livewire *backend* Anda:

```blade
<button x-on:click="$wire.save()">Save Post</button>

```

Atau, untuk mengilustrasikan contoh yang lebih kompleks, Anda mungkin menggunakan utilitas [`x-intersect`](https://www.google.com/search?q=%5Bhttps://alpinejs.dev/plugins/intersect%5D(https://alpinejs.dev/plugins/intersect)) dari Alpine untuk memicu *action* Livewire `incrementViewCount()` ketika elemen tertentu terlihat di halaman:

```blade
<div x-intersect="$wire.incrementViewCount()">...</div>

```

### Mengirimkan parameter

Parameter apa pun yang Anda kirimkan ke metode `$wire` juga akan diteruskan ke metode kelas PHP. Sebagai contoh, perhatikan *action* Livewire berikut:

```php
public function addTodo($todo)
{
    $this->todos[] = $todo;
}

```

Di dalam Blade *template* komponen Anda, Anda dapat memanggil *action* ini melalui Alpine, dengan memberikan parameter yang harus diberikan ke *action* tersebut:

```blade
<div x-data="{ todo: '' }">
    <input type="text" x-model="todo">

    <button x-on:click="$wire.addTodo(todo)">Add Todo</button>
</div>

```

Jika pengguna mengetikkan "Buang sampah" ke dalam input teks dan menekan tombol "Add Todo", metode `addTodo()` akan dipicu dengan nilai parameter `$todo` adalah "Buang sampah".

### Menerima nilai balik (return values)

Untuk kekuatan yang lebih besar, *actions* `$wire` yang dipanggil mengembalikan *promise* selama permintaan jaringan sedang diproses. Ketika respons server diterima, *promise* tersebut diselesaikan (*resolves*) dengan nilai yang dikembalikan oleh *action backend*.

Sebagai contoh, perhatikan komponen Livewire yang memiliki *action* berikut:

```php
use App\Models\Post;

public function getPostCount()
{
    return Post::count();
}

```

Menggunakan `$wire`, *action* tersebut dapat dipanggil dan nilai baliknya diselesaikan:

```blade
<span x-init="$el.innerHTML = await $wire.getPostCount()"></span>

```

Dalam contoh ini, jika metode `getPostCount()` mengembalikan "10", tag `<span>` juga akan berisi "10".

> [!tip] Gunakan #[Json] untuk action yang dikonsumsi JavaScript
> Untuk *actions* yang terutama dikonsumsi oleh JavaScript, pertimbangkan untuk menggunakan [atribut `#[Json]](https://www.google.com/search?q=/docs/4.x/attribute-json)`. Atribut ini mengembalikan data melalui resolusi/penolakan *promise*, menangani kesalahan validasi secara otomatis dengan penolakan *promise*, dan melewati proses render ulang untuk performa yang lebih baik.

Pengetahuan Alpine tidak wajib saat menggunakan Livewire; namun, Alpine adalah alat yang sangat kuat dan mempelajarinya akan meningkatkan pengalaman serta produktivitas Anda menggunakan Livewire.

## JavaScript actions

Livewire memungkinkan Anda untuk mendefinisikan **JavaScript actions** yang berjalan sepenuhnya di sisi klien tanpa melakukan permintaan (*request*) ke server. Ini sangat berguna dalam dua skenario:

1. Ketika Anda ingin melakukan pembaruan UI sederhana yang tidak memerlukan komunikasi server.
2. Ketika Anda ingin memperbarui UI secara **optimistic** dengan JavaScript sebelum melakukan permintaan ke server.

Untuk mendefinisikan JavaScript action, Anda dapat menggunakan fungsi `$js()` di dalam tag `<script>` pada komponen Anda.

Berikut adalah contoh fitur *bookmark* postingan yang menggunakan JavaScript action untuk memperbarui UI secara optimis. Ikon *bookmark* akan langsung berubah menjadi terisi, kemudian sistem mengirimkan permintaan untuk menyimpan data tersebut ke database:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;
    public $bookmarked = false;

    public function mount()
    {
        $this->bookmarked = $this->post->bookmarkedBy(auth()->user());
    }

    public function bookmarkPost()
    {
        $this->post->bookmark(auth()->user());
        $this->bookmarked = $this->post->bookmarkedBy(auth()->user());
    }
};

```

```blade
<div>
    <button wire:click="$js.bookmark" class="flex items-center gap-1">
        {{-- Ikon bookmark garis luar... --}}
        <svg wire:show="!bookmarked" wire:cloak ...>...</svg>

        {{-- Ikon bookmark terisi... --}}
        <svg wire:show="bookmarked" wire:cloak ...>...</svg>
    </button>
</div>

<script>
    this.$js.bookmark = () => {
        // Update UI secara instan di klien
        $wire.bookmarked = !$wire.bookmarked

        // Panggil method PHP untuk simpan ke DB
        $wire.bookmarkPost()
    }
</script>

```

Saat pengguna mengklik tombol, urutan berikut terjadi:

1. JavaScript action "bookmark" dipicu.
2. Ikon *bookmark* langsung berubah dengan mengganti nilai `$wire.bookmarked` di sisi klien.
3. Method `bookmarkPost()` dipanggil untuk menyimpan perubahan ke database.

Ini memberikan umpan balik visual yang instan sambil memastikan status *bookmark* tersimpan dengan benar di server.

> [!warning] Komponen berbasis kelas memerlukan pembungkus @@script
> Contoh di atas menggunakan tag `<script>` biasa, yang berfungsi untuk komponen *single-file* dan *multi-file*. Jika Anda menggunakan komponen berbasis kelas (*class-based components*), Anda harus membungkus tag script dengan direktif `@@script`:
> ```blade
> @@script
> <script>
>      this.$js.bookmark = () => { /* ... */ }
> </script>
> @@endscript
> 
> ```
> 
> 

### Memanggil dari Alpine

Anda dapat memanggil JavaScript actions langsung dari Alpine menggunakan objek `$wire`:

```blade
<button x-on:click="$wire.$js.bookmark()">Bookmark</button>

```

### Memanggil dari PHP

JavaScript actions juga dapat dipanggil menggunakan method `js()` dari sisi PHP:

```php
public function save()
{
    // ... logic simpan ...
    $this->js('onPostSaved'); 
}

```

## Magic actions

Livewire menyediakan set "**magic actions**" yang memungkinkan Anda melakukan tugas-tugas umum tanpa harus mendefinisikan method khusus di kelas PHP.

### `$parent`

Variabel ajaib `$parent` memungkinkan Anda mengakses properti atau memanggil action milik komponen induk dari komponen anak:

```blade
<button wire:click="$parent.removePost({{ $post->id }})">Remove</button>

```

### `$set`

Action `$set` memungkinkan Anda memperbarui properti di komponen Livewire secara langsung dari Blade:

```blade
<button wire:click="$set('query', '')">Reset Search</button>

```

### `$refresh`

Action `$refresh` akan memicu render ulang (*re-render*) komponen:

```blade
<button wire:click="$refresh">Refresh</button>

```

### `$toggle`

Action `$toggle` digunakan untuk membolak-balik nilai (*toggle*) properti boolean:

```blade
<button wire:click="$toggle('sortAsc')">
    Sort {{ $sortAsc ? 'Descending' : 'Ascending' }}
</button>

```

### `$dispatch`

Action `$dispatch` memungkinkan Anda mengirimkan (*dispatch*) *event* Livewire langsung di browser:

```blade
<button type="submit" wire:click="$dispatch('post-deleted')">Delete Post</button>

```

### `$event`

Variabel ajaib `$event` dapat digunakan di dalam *event listener* seperti `wire:click`. Action ini memberi Anda akses ke objek *event* JavaScript asli yang dipicu, memungkinkan Anda untuk mereferensikan elemen pemicu dan informasi relevan lainnya:

```blade
<input type="text" wire:keydown.enter="search($event.target.value)">

```

Ketika tombol Enter ditekan saat pengguna mengetik pada input di atas, konten dari input tersebut akan dikirimkan sebagai parameter ke action `search()`.

### Menggunakan magic actions dari Alpine

Anda juga dapat memanggil *magic actions* dari Alpine menggunakan objek `$wire`. Misalnya, Anda dapat menggunakan objek `$wire` untuk memicu *magic action* `$refresh`:

```blade
<button x-on:click="$wire.$refresh()">Refresh</button>

```

## Melewatkan Render Ulang (Skipping re-renders)

Terkadang terdapat *action* di komponen Anda yang tidak memiliki efek samping (*side effects*) yang akan mengubah tampilan Blade saat *action* tersebut dipanggil. Jika demikian, Anda dapat melewatkan bagian `render` dari siklus hidup Livewire dengan menambahkan atribut `#[Renderless]` di atas method *action* tersebut.

Untuk mendemonstrasikannya, pada komponen `ShowPost` di bawah ini, "jumlah tampilan" dicatat ketika pengguna telah menggulir ke bagian bawah postingan:

```php
<?php // resources/views/components/post/⚡show.blade.php

use Livewire\Attributes\Renderless;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    #[Renderless] // [tl! highlight]
    public function incrementViewCount()
    {
        $this->post->incrementViewCount();
    }
};

```

```blade
<div>
    <h1>{{ $post->title }}</h1>
    <p>{{ $post->content }}</p>

    <div wire:intersect="incrementViewCount"></div>
</div>

```

Contoh di atas menggunakan `wire:intersect` untuk memanggil *action* ketika elemen memasuki area pandang (*viewport*). Karena `#[Renderless]` ditambahkan, log tampilan dicatat, tetapi *template* tidak melakukan render ulang dan tidak ada bagian halaman yang terpengaruh.

Jika Anda memilih untuk tidak menggunakan atribut method atau perlu melewatkan render secara kondisional, Anda dapat memanggil method `skipRender()` di dalam *action* komponen Anda:

```php
public function incrementViewCount()
{
    $this->post->incrementViewCount();

    $this->skipRender(); // [tl! highlight]
}

```

Anda juga dapat melewatkan render langsung dari elemen menggunakan pengubah `.renderless`:

```blade
<button type="button" wire:click.renderless="incrementViewCount">

```

## Eksekusi Paralel dengan Async

Secara default, Livewire menjalankan *actions* secara berurutan (*sequential*) dalam komponen yang sama untuk memastikan pembaruan *state* yang dapat diprediksi. Jika satu *action* sedang berjalan, *action* berikutnya akan masuk antrean. Namun, terkadang Anda ingin *action* berjalan segera tanpa menunggu—secara paralel.

Atribut `#[Async]` dan pengubah `wire:click.async` memberitahu Livewire untuk mengeksekusi *action* secara paralel, melewati antrean permintaan normal.

### Menggunakan pengubah async

Anda dapat membuat *action* apa pun menjadi asinkron dengan menambahkan pengubah `.async` pada *event listener* Anda:

```blade
<button wire:click.async="logActivity">Track Event</button>

```

### Menggunakan atribut Async

Atau, Anda dapat menandai sebuah method sebagai asinkron menggunakan atribut `#[Async]`. Ini membuat *action* tersebut selalu asinkron dari mana pun ia dipanggil:

```php
#[Async]
public function logActivity()
{
    Activity::log('post-viewed', $this->post);
}

```

### Kapan menggunakan async actions

*Async actions* berguna untuk operasi "tembak dan lupakan" (*fire-and-forget*) di mana hasilnya tidak memengaruhi apa yang ditampilkan di halaman. Kasus penggunaan umum meliputi:

* **Analitik dan Logging:** Melacak perilaku pengguna atau tampilan halaman.
* **Operasi Latar Belakang:** Memicu *job*, mengirim notifikasi, atau memperbarui layanan eksternal.
* **Hasil Khusus JavaScript:** Mengambil data via `await $wire.getData()` yang akan dikonsumsi murni oleh JavaScript.

### Kapan TIDAK menggunakan async actions

> [!warning] Async actions dan mutasi state tidak boleh dicampur
> **Jangan pernah menggunakan async actions jika mereka mengubah state komponen yang tercermin di UI Anda.** Karena berjalan secara paralel, Anda bisa berakhir dengan kondisi balapan (*race conditions*) yang tidak terduga di mana *state* komponen Anda menjadi tidak sinkron antar permintaan yang simultan.

**Aturan praktisnya:** Hanya gunakan *async* untuk *actions* yang melakukan efek samping murni—operasi yang tidak mengubah properti apa pun yang memengaruhi tampilan komponen Anda.

## Mempertahankan Posisi Scroll (Preserving scroll position)

Saat memperbarui konten, browser mungkin melompat ke posisi gulir (*scroll*) yang berbeda. Pengubah `.preserve-scroll` mempertahankan posisi gulir saat ini selama pembaruan:

```blade
<button wire:click.preserve-scroll="loadMore">Load More</button>

```

Ini sangat berguna untuk *infinite scroll*, filter, dan pembaruan konten dinamis agar halaman tidak "melompat".

## Masalah Keamanan (Security concerns)

Ingatlah bahwa **setiap method publik** dalam komponen Livewire Anda dapat dipanggil dari sisi klien, bahkan tanpa adanya handler `wire:click` yang terkait. Pengguna yang jahat dapat memicu *action* tersebut melalui DevTools browser.

Selalu terapkan otorisasi di dalam method Anda. Jika Anda menganggap method komponen sebagai proksi untuk method *controller*, dan parameternya sebagai proksi untuk *input* permintaan, Anda dapat menerapkan pengetahuan keamanan aplikasi standar Anda pada kode Livewire.

### Selalu otorisasi parameter action

Sama seperti input permintaan (*request*) pada controller, sangat penting untuk mengotorisasi parameter action karena parameter tersebut merupakan input pengguna yang bersifat arbitrer.

Di bawah ini adalah komponen `ShowPosts` di mana pengguna dapat melihat semua postingan mereka dalam satu halaman. Mereka dapat menghapus postingan apa pun yang mereka inginkan menggunakan salah satu tombol "Delete" pada postingan tersebut.

Berikut adalah versi komponen yang rentan (*vulnerable*):

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function delete($id)
    {
        $post = Post::find($id);

        $post->delete();
    }
};

```

```blade
<div>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>

```

Ingatlah bahwa pengguna jahat dapat memanggil `delete()` secara langsung dari konsol JavaScript, mengirimkan parameter apa pun yang mereka inginkan ke action tersebut. Ini berarti pengguna yang sedang melihat salah satu postingan mereka dapat menghapus postingan milik pengguna lain dengan mengirimkan ID postingan yang bukan miliknya ke `delete()`.

Untuk melindungi dari hal ini, kita perlu mengotorisasi bahwa pengguna tersebut memiliki postingan yang akan dihapus:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function delete($id)
    {
        $post = Post::find($id);

        $this->authorize('delete', $post); // [tl! highlight]

        $post->delete();
    }
};

```

### Selalu otorisasi di sisi server

Seperti controller standar Laravel, action Livewire dapat dipanggil oleh pengguna mana pun, bahkan jika tidak ada fasilitas (*affordance*) untuk memanggil action tersebut di UI.

Perhatikan komponen `BrowsePosts` berikut di mana pengguna mana pun dapat melihat semua postingan dalam aplikasi, tetapi hanya administrator yang dapat menghapus postingan:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function deletePost($id)
    {
        $post = Post::find($id);

        $post->delete();
    }
};

```

```blade
<div>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            @if (Auth::user()->isAdmin())
                <button wire:click="deletePost({{ $post->id }})">Delete</button>
            @endif
        </div>
    @endforeach
</div>

```

Seperti yang Anda lihat, hanya administrator yang dapat melihat tombol "Delete"; namun, pengguna mana pun dapat memanggil `deletePost()` pada komponen dari DevTools browser.

Untuk menambal kerentanan ini, kita perlu mengotorisasi action di server seperti berikut:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) { // [tl! highlight:2]
            abort(403);
        }

        $post = Post::find($id);

        $post->delete();
    }
};

```

Dengan perubahan ini, hanya administrator yang dapat menghapus postingan dari komponen ini.

### Jaga agar metode berbahaya tetap protected atau private

Setiap metode `public` di dalam komponen Livewire Anda dapat dipanggil dari sisi klien (*client-side*). Bahkan metode yang tidak Anda referensikan di dalam handler `wire:click`. Untuk mencegah pengguna memanggil metode yang tidak dimaksudkan untuk dipanggil dari sisi klien, Anda harus menandainya sebagai `protected` atau `private`. Dengan melakukan itu, Anda membatasi visibilitas metode sensitif tersebut hanya untuk kelas komponen itu sendiri dan subkelasnya, memastikan metode tersebut tidak dapat dipanggil dari sisi klien.

Pertimbangkan contoh `BrowsePosts` yang kita bahas sebelumnya, di mana pengguna dapat melihat semua postingan dalam aplikasi Anda, tetapi hanya administrator yang dapat menghapus postingan. Pada bagian [Selalu otorisasi di sisi server](https://www.google.com/search?q=/docs/4.x/actions%23always-authorize-server-side), kita telah membuat action tersebut aman dengan menambahkan otorisasi sisi server. Sekarang bayangkan kita melakukan refaktor pada proses penghapusan postingan ke dalam metode khusus seperti yang mungkin Anda lakukan untuk menyederhanakan kode Anda:

```php
// Peringatan: Cuplikan kode ini menunjukkan apa yang TIDAK boleh dilakukan...
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        // Memanggil metode pembantu di bawah ini
        $this->delete($id); // [tl! highlight]
    }

    // Metode ini bersifat publik secara default jika tidak ditentukan aksesnya
    public function delete($postId)  // [tl! highlight:5]
    {
        $post = Post::find($postId);

        $post->delete();
    }
};

```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="deletePost({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>

```

Seperti yang Anda lihat, kita merefaktorkan logika penghapusan postingan ke dalam metode khusus bernama `delete()`. Meskipun metode ini tidak direferensikan di mana pun dalam template kita, jika seorang pengguna mengetahui keberadaannya, mereka akan dapat memanggilnya dari DevTools browser karena metode tersebut bersifat `public`.

Untuk mengatasi hal ini, kita dapat menandai metode tersebut sebagai `protected` atau `private`. Setelah metode ditandai sebagai `protected` atau `private`, sebuah kesalahan (*error*) akan muncul jika pengguna mencoba memanggilnya:

```php
<?php // resources/views/components/post/⚡index.blade.php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        $this->delete($id);
    }

    // Menandai metode sebagai protected mencegah akses dari browser
    protected function delete($postId) // [tl! highlight]
    {
        $post = Post::find($postId);

        $post->delete();
    }
};

```

## Lihat juga

* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — Berkomunikasi antar komponen menggunakan events
* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Menangani pengiriman formulir dengan actions
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Menampilkan umpan balik saat actions sedang diproses
* **[wire:click](https://www.google.com/search?q=/docs/4.x/wire-click)** — Memicu actions dari klik tombol
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Memvalidasi data sebelum memproses actions
