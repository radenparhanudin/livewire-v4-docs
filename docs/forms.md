Karena *forms* adalah tulang punggung dari sebagian besar aplikasi web, Livewire menyediakan banyak utilitas bermanfaat untuk membangunnya. Mulai dari menangani elemen *input* sederhana hingga hal-hal kompleks seperti *validation* waktu nyata (*real-time*) atau *file uploading*, Livewire memiliki alat yang sederhana dan terdokumentasi dengan baik untuk mempermudah hidup Anda dan memuaskan pengguna Anda.

Mari kita pelajari lebih dalam.

## Submitting a form

Mari kita mulai dengan melihat *form* yang sangat sederhana dalam komponen `post.create`. *Form* ini akan memiliki dua *text input* sederhana dan sebuah tombol *submit*, serta beberapa kode pada *backend* untuk mengelola *state* dan *submission* dari *form* tersebut:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create(
            $this->only(['title', 'content'])
        );

        session()->flash('status', 'Post successfully updated.');

        return $this->redirect('/posts');
    }
};
?>

<form wire:submit="save">
    <input type="text" wire:model="title">

    <input type="text" wire:model="content">

    <button type="submit">Save</button>
</form>

```

Seperti yang Anda lihat, kita melakukan "**binding**" pada *public property* `$title` dan `$content` di dalam *form* di atas menggunakan `wire:model`. Ini adalah salah satu fitur Livewire yang paling sering digunakan dan sangat kuat.

Selain melakukan *binding* pada `$title` dan `$content`, kita menggunakan `wire:submit` untuk menangkap *event* `submit` ketika tombol "Save" diklik dan memanggil **action** `save()`. *Action* ini akan menyimpan *form input* ke dalam database.

Setelah postingan baru dibuat di database, kita melakukan *redirect* pengguna ke halaman postingan dan menunjukkan pesan "**flash**" bahwa postingan baru telah dibuat.

### Adding validation

Untuk menghindari penyimpanan *user input* yang tidak lengkap atau berbahaya, sebagian besar *forms* memerlukan semacam *input validation*.

Livewire membuat proses *validating* pada *forms* Anda sesederhana menambahkan atribut `#[Validate]` di atas *properties* yang ingin Anda validasi.

Setelah sebuah *property* memiliki atribut `#[Validate]`, aturan *validation* akan diterapkan pada nilai *property* tersebut setiap kali ia diperbarui di sisi server.

Mari tambahkan beberapa aturan *validation* dasar pada *property* `$title` dan `$content` di komponen `post.create` kita:

```php
<?php // resources/views/components/post/⚡create.blade.php

use Livewire\Attributes\Validate; // [tl! highlight]
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Validate('required')] // [tl! highlight]
    public $title = '';

    #[Validate('required')] // [tl! highlight]
    public $content = '';

    public function save()
    {
        $this->validate(); // [tl! highlight]

        Post::create(
            $this->only(['title', 'content'])
        );

        return $this->redirect('/posts');
    }
};

```

Kita juga akan memodifikasi *template* Blade kita untuk menunjukkan *validation errors* pada halaman.

```blade
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror </div>

    <input type="text" wire:model="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror </div>

    <button type="submit">Save</button>
</form>

```

Sekarang, jika pengguna mencoba melakukan *submit* pada *form* tanpa mengisi bidang apa pun, mereka akan melihat pesan *validation* yang memberi tahu bidang mana saja yang `required` sebelum menyimpan postingan.

### Extracting a form object

Jika Anda bekerja dengan *form* yang besar dan lebih suka mengekstraksi semua *properties*, logika *validation*, dll., ke dalam kelas terpisah, Livewire menawarkan **form objects**.

*Form objects* memungkinkan Anda menggunakan kembali logika *form* di berbagai komponen dan memberikan cara yang baik untuk menjaga kelas komponen Anda tetap bersih dengan mengelompokkan semua kode terkait *form* ke dalam kelas terpisah.

Anda dapat membuat kelas *form* secara manual atau menggunakan perintah Artisan yang praktis:

```shell
php artisan livewire:form PostForm

```

Perintah di atas akan membuat file bernama `app/Livewire/Forms/PostForm.php`. Mari kita tulis ulang komponen `post.create` untuk menggunakan kelas `PostForm`:

```php
// app/Livewire/Forms/PostForm.php
namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';
}

```

```php
// resources/views/components/post/create.blade.php
new class extends Component {
    public PostForm $form; // [tl! highlight]

    public function save()
    {
        $this->validate();

        Post::create(
            $this->form->only(['title', 'content'])
        );

        return $this->redirect('/posts');
    }
};

```

### Showing a loading indicator

Secara *default*, Livewire akan secara otomatis menonaktifkan tombol *submit* dan menandai *inputs* sebagai `readonly` saat *form* sedang di-*submit*, mencegah pengguna mengirimkan *form* lagi saat *submission* pertama sedang ditangani.

Namun, sulit bagi pengguna untuk mendeteksi **loading state** ini tanpa bantuan visual ekstra di UI aplikasi Anda.

Berikut adalah contoh penambahan *loading spinner* kecil ke tombol "Save" melalui `wire:loading` sehingga pengguna memahami bahwa *form* sedang di-*submit*:

```blade
<button type="submit">
    Save

    <div wire:loading>
        <svg>...</svg> </div>
</button>

```

### Showing dirty indicators

Dalam skenario *real-time saving*, mungkin membantu untuk menunjukkan kepada pengguna ketika sebuah bidang belum di-*persisted* ke database.

Livewire menyediakan direktif `wire:dirty` yang memungkinkan Anda melakukan *toggle* pada elemen atau memodifikasi *classes* ketika nilai sebuah *input* berbeda (*diverges*) dari komponen di sisi server:

```blade
<input type="text" wire:model.live.blur="title" wire:dirty.class="border-yellow">

```

Pada contoh di atas, ketika pengguna mengetik di kolom *input*, bingkai kuning akan muncul di sekitar kolom tersebut. Ketika pengguna melakukan *tab away* (*blur*), permintaan jaringan dikirim dan bingkai akan hilang; menandakan bahwa *input* telah di-*persisted* dan tidak lagi "**dirty**".

---

## See also

* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Validasi *form inputs* dengan *real-time feedback*.
* **[wire:model](https://www.google.com/search?q=/docs/4.x/wire-model)** — Melakukan *binding form inputs* ke *component properties*.
* **[File Uploads](https://www.google.com/search?q=/docs/4.x/uploads)** — Menangani *file uploads* di dalam *forms*.
* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Memproses *form submissions* dengan *actions*.
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Menampilkan *loading indicators* selama *form submission*.
