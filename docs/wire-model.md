Livewire mempermudah Anda untuk menghubungkan (*bind*) nilai properti **component** dengan input formulir menggunakan `wire:model`.

Berikut adalah contoh sederhana penggunaan `wire:model` untuk menghubungkan properti `$title` dan `$content` dengan input formulir dalam **component** "Create Post":

```php
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        $post = Post::create([
            'title' => $this->title,
            'content' => $this->content
        ]);

        // ...
    }
}

```

```blade
<form wire:submit="save">
    <label>
        <span>Title</span>

        <input type="text" wire:model="title"> </label>

    <label>
        <span>Content</span>

        <textarea wire:model="content"></textarea> </label>

    <button type="submit">Save</button>
</form>

```

Karena kedua input menggunakan `wire:model`, nilai-nilainya akan disinkronkan dengan properti di server saat tombol "Save" ditekan.

> [!warning] "Mengapa component saya tidak update secara live saat saya mengetik?"
> Jika Anda mencoba ini di browser dan bingung mengapa judulnya tidak diperbarui secara otomatis, itu karena Livewire hanya memperbarui **component** saat sebuah "**action**" dikirimkan—seperti menekan tombol *submit*—bukan saat pengguna mengetik di kolom input. Hal ini mengurangi **network requests** dan meningkatkan performa. Untuk mengaktifkan pembaruan secara langsung (*live*) saat pengguna mengetik, Anda dapat menggunakan `wire:model.live`. [Pelajari lebih lanjut tentang data binding](https://www.google.com/search?q=/docs/4.x/properties%23data-binding).

## Menyesuaikan waktu pembaruan

Secara default, Livewire hanya akan mengirimkan **network request** saat sebuah **action** dilakukan (seperti `wire:click` atau `wire:submit`), BUKAN saat input `wire:model` diperbarui.

Hal ini secara drastis meningkatkan performa Livewire dengan mengurangi **network requests** dan memberikan pengalaman yang lebih mulus bagi pengguna Anda.

Namun, ada kalanya Anda ingin memperbarui server lebih sering untuk hal-hal seperti validasi waktu nyata (*real-time validation*).

### Pembaruan Live

Untuk mengirimkan pembaruan properti ke server saat pengguna mengetik di kolom input, Anda dapat menambahkan **modifier** `.live` pada `wire:model`:

```html
<input type="text" wire:model.live="title">

```

#### Menyesuaikan debounce

Secara default, saat menggunakan `wire:model.live`, Livewire menambahkan **debounce** sebesar 150 milidetik pada pembaruan server. Ini berarti jika pengguna mengetik terus-menerus, Livewire akan menunggu hingga pengguna berhenti mengetik selama 150 milidetik sebelum mengirimkan **request**.

Anda dapat menyesuaikan waktu ini dengan menambahkan `.debounce.Xms` setelah `.live`. Berikut adalah contoh mengubah **debounce** menjadi 250 milidetik:

```html
<input type="text" wire:model.live.debounce.250ms="title">

```

### Memperbarui pada event "blur"

**Modifier** `.blur` menunda sinkronisasi hingga pengguna mengeklik di luar area input (*blur*):

```html
<input type="text" wire:model.blur="title">

```

Untuk juga mengirimkan **network request** saat *blur*, tambahkan `.live`:

```html
<input type="text" wire:model.blur.live="title">

```

### Memperbarui pada event "change"

**Modifier** `.change` terpicu pada **event** perubahan (*change event*), yang berguna untuk elemen *select*:

```html
<select wire:model.change="state">...</select>

<select wire:model.change.live="state">...</select>

```

### Memperbarui pada tombol "enter"

**Modifier** `.enter` melakukan sinkronisasi saat pengguna menekan tombol Enter:

```html
<input type="text" wire:model.enter="search">

<input type="text" wire:model.enter.live="search">

```

## Input fields

Livewire mendukung sebagian besar elemen input asli browser secara langsung. Artinya, Anda seharusnya bisa memasangkan `wire:model` ke elemen input apa pun dan menghubungkan properti ke elemen tersebut dengan mudah.

Berikut adalah daftar lengkap berbagai tipe input yang tersedia dan cara menggunakannya dalam konteks Livewire.

### Text inputs

Yang paling utama, input teks adalah dasar dari sebagian besar formulir. Berikut cara menghubungkan properti bernama "title" ke input teks:

```blade
<input type="text" wire:model="title">

```

### Textarea inputs

Elemen textarea juga sama sederhananya. Cukup tambahkan `wire:model` ke textarea dan nilainya akan terhubung:

```blade
<textarea type="text" wire:model="content"></textarea>

```

Jika nilai "content" diinisialisasi dengan sebuah string, Livewire akan mengisi textarea dengan nilai tersebut - tidak perlu melakukan hal seperti berikut:

```blade
<textarea type="text" wire:model="content">{{ $content }}</textarea>

```

### Checkboxes

*Checkboxes* dapat digunakan untuk nilai tunggal, seperti saat mengubah properti boolean. Atau, *checkboxes* dapat digunakan untuk memilih satu atau beberapa nilai dalam sekelompok nilai yang terkait. Kita akan membahas kedua skenario tersebut:

#### Single checkbox

Di akhir formulir pendaftaran, Anda mungkin memiliki *checkbox* yang memungkinkan pengguna untuk berlangganan pembaruan email. Anda mungkin menamai properti ini `$receiveUpdates`. Anda dapat dengan mudah menghubungkan nilai ini ke *checkbox* menggunakan `wire:model`:

```blade
<input type="checkbox" wire:model="receiveUpdates">

```

Sekarang ketika nilai `$receiveUpdates` adalah `false`, *checkbox* tidak akan dicentang. Tentu saja, ketika nilainya `true`, *checkbox* akan dicentang.

#### Multiple checkboxes

Sekarang, katakanlah selain mengizinkan pengguna untuk memutuskan menerima pembaruan, Anda memiliki properti array di class Anda yang disebut `$updateTypes`, yang memungkinkan pengguna memilih dari berbagai jenis pembaruan:

```php
public $updateTypes = [];

```

Dengan menghubungkan beberapa *checkbox* ke properti `$updateTypes`, pengguna dapat memilih beberapa jenis pembaruan dan nilai tersebut akan ditambahkan ke properti array `$updateTypes`:

```blade
<input type="checkbox" value="email" wire:model="updateTypes">
<input type="checkbox" value="sms" wire:model="updateTypes">
<input type="checkbox" value="notification" wire:model="updateTypes">

```

Misalnya, jika pengguna mencentang dua kotak pertama tetapi tidak yang ketiga, nilai dari `$updateTypes` akan menjadi: `["email", "sms"]`

### Radio buttons

Untuk beralih di antara dua nilai yang berbeda untuk satu properti, Anda dapat menggunakan *radio buttons*:

```blade
<input type="radio" value="yes" wire:model="receiveUpdates">
<input type="radio" value="no" wire:model="receiveUpdates">

```

### Select dropdowns

Livewire mempermudah pekerjaan dengan dropdown `<select>`. Saat menambahkan `wire:model` ke dropdown, nilai yang saat ini dipilih akan dihubungkan ke nama properti yang diberikan dan sebaliknya.

Selain itu, tidak perlu menambahkan `selected` secara manual ke opsi yang akan dipilih - Livewire menangani hal itu untuk Anda secara otomatis.

Di bawah ini adalah contoh dropdown *select* yang diisi dengan daftar statis negara bagian:

```blade
<select wire:model="state">
    <option value="AL">Alabama</option>
    <option value="AK">Alaska</option>
    <option value="AZ">Arizona</option>
    ...
</select>

```

Ketika negara bagian tertentu dipilih, misalnya "Alaska", properti `$state` pada **component** akan diatur menjadi `AK`. Jika Anda lebih suka nilainya diatur menjadi "Alaska" alih-alih "AK", Anda dapat menghapus atribut `value=""` dari elemen `<option>` sepenuhnya.

Sering kali, Anda mungkin membangun opsi dropdown Anda secara dinamis menggunakan Blade:

```blade
<select wire:model="state">
    @foreach (\App\Models\State::all() as $state)
        <option value="{{ $state->id }}">{{ $state->label }}</option>
    @endforeach
</select>

```

Jika Anda tidak memiliki opsi khusus yang dipilih secara default, Anda mungkin ingin menampilkan opsi *placeholder* yang diredam secara default, seperti "Select a state":

```blade
<select wire:model="state">
    <option disabled value="">Select a state...</option>

    @foreach (\App\Models\State::all() as $state)
        <option value="{{ $state->id }}">{{ $state->label }}</option>
    @endforeach
</select>

```

Seperti yang Anda lihat, tidak ada atribut "placeholder" untuk menu *select* seperti pada input teks. Sebagai gantinya, Anda harus menambahkan elemen opsi dengan atribut `disabled` sebagai opsi pertama dalam daftar.

### Dependent select dropdowns

Terkadang Anda mungkin ingin satu menu *select* bergantung pada menu *select* lainnya. Contohnya, daftar kota yang berubah berdasarkan negara bagian yang dipilih.

Secara umum, ini bekerja seperti yang Anda harapkan, namun ada satu hal penting yang perlu diperhatikan (*gotcha*): Anda harus menambahkan `wire:key` ke menu *select* yang berubah agar Livewire dapat menyegarkan nilainya dengan benar saat opsi di dalamnya berubah.

Berikut adalah contoh dua buah *select*, satu untuk negara bagian (*states*), satu untuk kota (*cities*). Ketika *select* negara bagian berubah, opsi dalam *select* kota akan berubah dengan benar:

```blade
<select wire:model.live="selectedState">
    @foreach (State::all() as $state)
        <option value="{{ $state->id }}">{{ $state->label }}</option>
    @endforeach
</select>

<select wire:model.live="selectedCity" wire:key="{{ $selectedState }}"> @foreach (City::whereStateId($selectedState->id)->get() as $city)
        <option value="{{ $city->id }}">{{ $city->label }}</option>
    @endforeach
</select>

```

Sekali lagi, satu-satunya hal yang tidak standar di sini adalah `wire:key` yang ditambahkan ke *select* kedua. Ini memastikan bahwa ketika negara bagian berubah, nilai "selectedCity" akan diatur ulang (*reset*) dengan benar.

### Multi-select dropdowns

Jika Anda menggunakan menu *select* "multiple", Livewire bekerja sebagaimana mestinya. Dalam contoh ini, negara bagian akan ditambahkan ke properti array `$states` saat dipilih dan dihapus jika pilihan dibatalkan:

```blade
<select wire:model="states" multiple>
    <option value="AL">Alabama</option>
    <option value="AK">Alaska</option>
    <option value="AZ">Arizona</option>
    ...
</select>

```

---

## Event propagation

Secara default, `wire:model` hanya mendengarkan **event** *input/change* yang berasal langsung dari elemen itu sendiri, bukan **event** yang merambat naik (*bubble up*) dari elemen anak (*child elements*). Hal ini mencegah perilaku tak terduga saat menggunakan `wire:model` pada elemen kontainer seperti *modal* atau *accordion* yang berisi input formulir lainnya.

Misalnya, jika Anda memiliki *modal* dengan `wire:model="showModal"` dan sebuah kolom input di dalamnya, menghapus isi input tersebut tidak akan secara tidak sengaja menutup *modal* karena adanya perambatan **event** *change*.

### Mendengarkan event dari elemen anak

Dalam kasus langka di mana Anda ingin `wire:model` juga merespons **event** yang merambat naik dari elemen anak, Anda dapat menggunakan **modifier** `.deep`:

```blade
<div wire:model.deep="value">
    <input type="text"> </div>

```

> [!warning] Gunakan `.deep` dengan bijak
> Sebagian besar kasus penggunaan tidak memerlukan pemantauan **event** elemen anak. Hanya gunakan `.deep` saat Anda secara spesifik perlu menangkap **event** dari elemen keturunan (*descendant*).

---

## Pelajari lebih dalam

Untuk dokumentasi yang lebih lengkap mengenai penggunaan `wire:model` dalam konteks formulir HTML, kunjungi [halaman dokumentasi Livewire forms](https://www.google.com/search?q=/docs/4.x/forms).

---

## See also

* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Panduan lengkap membangun formulir dengan Livewire
* **[Properties](https://www.google.com/search?q=/docs/4.x/properties)** — Pahami data binding dan manajemen properti
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Validasi properti yang terikat secara *real-time*
* **[File Uploads](https://www.google.com/search?q=/docs/4.x/uploads)** — Hubungkan input file dengan `wire:model`

---

## Referensi

```blade
wire:model="propertyName"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.live` | Mengirimkan pembaruan ke server secara langsung |
| `.blur` | Hanya memperbarui saat elemen kehilangan fokus (*blur*) |
| `.change` | Hanya memperbarui saat terjadi perubahan (*change*) |
| `.enter` | Hanya memperbarui saat tombol Enter ditekan |
| `.lazy` | Memperbarui saat terjadi perubahan dan mengirimkan *network request* (kompatibel dengan v3) |
| `.debounce.Xms` | Menunda pembaruan (digunakan bersama `.live`) |
| `.throttle.Xms` | Membatasi frekuensi pembaruan (digunakan bersama `.live`) |
| `.number` | Mengubah nilai menjadi `int` di server |
| `.boolean` | Mengubah nilai menjadi `bool` di server |
| `.fill` | Menggunakan nilai awal dari atribut HTML `value` |
| `.deep` | Juga mendengarkan **event** dari elemen anak |
| `.preserve-scroll` | Mempertahankan posisi gulir (*scroll*) selama pembaruan |
