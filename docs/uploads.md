Livewire menawarkan dukungan yang sangat kuat untuk menangani unggahan file di dalam **components** Anda.

Pertama, tambahkan **trait** `WithFileUploads` ke dalam **component** Anda. Setelah **trait** ini ditambahkan, Anda dapat menggunakan `wire:model` pada input file seolah-olah itu adalah tipe input lainnya, dan Livewire akan menangani sisanya.

Berikut adalah contoh **component** sederhana yang menangani pengunggahan foto:

```php
<?php // resources/views/components/⚡upload-photo.blade.php

use Livewire\Attributes\Validate;
use Livewire\WithFileUploads;
use Livewire\Component;

new class extends Component {
    use WithFileUploads;

    #[Validate('image|max:1024')] // Maksimal 1MB
    public $photo;

    public function save()
    {
        $this->photo->store(path: 'photos');
    }
};

```

```blade
<form wire:submit="save">
    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>

```

> [!warning] Metode "upload" sudah dipesan (reserved)
> Perhatikan bahwa contoh di atas menggunakan metode "save", bukan metode "upload". Ini adalah "jebakan" umum. Istilah "upload" sudah dipesan oleh Livewire secara internal. Anda tidak dapat menggunakannya sebagai nama metode atau nama properti di dalam **component** Anda.

Dari sudut pandang pengembang, menangani input file tidak ada bedanya dengan menangani tipe input lainnya: Cukup tambahkan `wire:model` ke tag `<input>` dan semuanya akan diurus untuk Anda.

Namun, di balik layar, terdapat proses yang lebih kompleks agar unggah file dapat bekerja di Livewire. Berikut adalah gambaran apa yang terjadi saat pengguna memilih file untuk diunggah:

1. Saat file baru dipilih, JavaScript Livewire membuat permintaan awal ke **component** di server untuk mendapatkan URL unggah sementara yang "ditandatangani" (**signed URL**).
2. Setelah URL diterima, JavaScript melakukan pengunggahan (*upload*) yang sebenarnya ke **signed URL** tersebut, menyimpan file di direktori sementara yang ditentukan oleh Livewire, dan mengembalikan ID hash unik dari file sementara tersebut.
3. Setelah file diunggah dan ID hash unik dibuat, JavaScript Livewire membuat permintaan terakhir ke **component** di server, memerintahkannya untuk mengisi (*set*) properti publik yang diinginkan dengan file sementara yang baru tersebut.
4. Sekarang, properti publik (dalam kasus ini, `$photo`) sudah berisi file unggahan sementara dan siap untuk disimpan atau divalidasi kapan saja.

---

## Storing uploaded files

Contoh sebelumnya menunjukkan skenario penyimpanan yang paling dasar: memindahkan file unggahan sementara ke direktori "photos" pada **disk** sistem file **default** aplikasi.

Namun, Anda mungkin ingin menyesuaikan nama file yang disimpan atau menentukan **disk** penyimpanan tertentu (seperti S3).

> [!tip] Nama file asli
> Anda dapat mengakses nama file asli dari unggahan sementara dengan memanggil metode `->getClientOriginalName()`.

Livewire mematuhi API yang sama dengan yang digunakan Laravel untuk menyimpan file yang diunggah, jadi silakan merujuk pada [dokumentasi file upload Laravel](https://laravel.com/docs/filesystem#file-uploads). Berikut adalah beberapa skenario penyimpanan yang umum:

```php
public function save()
{
    // Simpan file di direktori "photos" pada disk default
    $this->photo->store(path: 'photos');

    // Simpan file di direktori "photos" pada disk "s3"
    $this->photo->store(path: 'photos', options: 's3');

    // Simpan file di direktori "photos" dengan nama "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar');

    // Simpan file di direktori "photos" pada disk "s3" dengan nama "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar', options: 's3');

    // Simpan file secara publik di disk "s3"
    $this->photo->storePublicly(path: 'photos', options: 's3');

    // Simpan secara publik dengan nama tertentu di disk "s3"
    $this->photo->storePubliclyAs(path: 'photos', name: 'avatar', options: 's3');
}

```

---

## Handling multiple files

Livewire secara otomatis menangani pengunggahan banyak file dengan mendeteksi atribut `multiple` pada tag `<input>`.

Contoh di bawah ini adalah **component** dengan properti array bernama `$photos`. Dengan menambahkan `multiple` pada input file, Livewire akan secara otomatis memasukkan file-file baru ke dalam array ini:

```php
new class extends Component {
    use WithFileUploads;

    #[Validate(['photos.*' => 'image|max:1024'])]
    public $photos = [];

    public function save()
    {
        foreach ($this->photos as $photo) {
            $photo->store(path: 'photos');
        }
    }
};

```

```blade
<form wire:submit="save">
    <input type="file" wire:model="photos" multiple>

    @error('photos.*') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>

```

---

## File validation

Seperti yang telah dibahas, memvalidasi unggahan file dengan Livewire sama dengan menangani validasi file dari **controller** Laravel standar.

> [!warning] Pastikan S3 dikonfigurasi dengan benar
> Banyak aturan validasi file memerlukan akses langsung ke file tersebut. Saat menggunakan fitur **Direct Uploads** ke S3, aturan validasi ini akan gagal jika objek file di S3 tidak dapat diakses secara publik.

Untuk informasi lebih lanjut tentang validasi file, lihat [dokumentasi validasi Laravel](https://laravel.com/docs/validation#available-validation-rules).

---

## Temporary preview URLs

Setelah pengguna memilih file, biasanya Anda perlu menampilkan pratinjau (*preview*) file tersebut sebelum mereka mengirim formulir.

Livewire mempermudah hal ini dengan menggunakan metode `->temporaryUrl()` pada file yang diunggah.

> [!info] Temporary URL terbatas untuk gambar
> Untuk alasan keamanan, pratinjau URL sementara hanya didukung pada file dengan tipe MIME gambar.

Berikut contoh pengunggahan file dengan pratinjau gambar:

```blade
<form wire:submit="save">
    @if ($photo) 
        <img src="{{ $photo->temporaryUrl() }}">
    @endif

    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>

```

Karena Livewire menyimpan file sementara di direktori non-publik, ia menyediakan **signed URL** sementara yang "berpura-pura" menjadi gambar agar halaman Anda dapat menampilkan pratinjau kepada pengguna tanpa mengekspos direktori sistem.

> [!tip] S3 temporary signed URLs
> Jika Anda mengonfigurasi Livewire untuk menggunakan S3 sebagai penyimpanan sementara, memanggil `->temporaryUrl()` akan menghasilkan **signed URL** langsung ke S3 sehingga beban server Laravel Anda berkurang.

---

## Testing file uploads

Anda dapat menggunakan **testing helpers** Laravel yang sudah ada untuk menguji pengunggahan file.

Berikut adalah contoh lengkap pengujian **component** `UploadPhoto` dengan Livewire:

```php
public function test_can_upload_photo()
{
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.png');

    Livewire::test(UploadPhoto::class)
        ->set('photo', $file)
        ->call('upload', 'uploaded-avatar.png');

    Storage::disk('avatars')->assertExists('uploaded-avatar.png');
}

```

Berikut adalah bagian fungsional dari **component** tersebut agar pengujian di atas berhasil:

```php
new class extends Component {
    use WithFileUploads;

    public $photo;

    public function upload($name)
    {
        $this->photo->storeAs('/', $name, disk: 'avatars');
    }
};

```

Untuk informasi lebih lanjut mengenai pengujian unggahan file, silakan merujuk pada [dokumentasi pengujian unggahan file Laravel](https://laravel.com/docs/http-tests#testing-file-uploads).

## Uploading directly to Amazon S3

Seperti yang telah dibahas sebelumnya, Livewire menyimpan semua unggahan file di direktori sementara sampai pengembang menyimpannya secara permanen.

Secara *default*, Livewire menggunakan konfigurasi **disk** sistem file bawaan (biasanya `local`) dan menyimpan file-file tersebut di dalam direktori `livewire-tmp/`.

Akibatnya, unggahan file akan selalu menggunakan server aplikasi Anda, meskipun Anda memilih untuk menyimpan file tersebut ke dalam **S3 bucket** nantinya.

Jika Anda ingin melewati server aplikasi dan langsung menyimpan unggahan sementara Livewire ke dalam **S3 bucket**, atur **environment variable** `LIVEWIRE_TEMPORARY_FILE_UPLOAD_DISK` di file `.env` Anda menjadi `s3` (atau **disk** kustom lainnya yang menggunakan **driver** `s3`):

```env
LIVEWIRE_TEMPORARY_FILE_UPLOAD_DISK=s3

```

Sekarang, saat pengguna mengunggah file, file tersebut tidak akan pernah benar-benar disimpan di server Anda. Sebaliknya, file akan diunggah langsung ke **S3 bucket** Anda di dalam sub-direktori `livewire-tmp/`.

> [!tip]
> Sebagai alternatif, Anda dapat mempublikasikan file konfigurasi Livewire dengan `php artisan livewire:config` untuk kontrol penuh atas konfigurasi `temporary_file_upload`.

### Mengonfigurasi pembersihan file otomatis

Direktori unggahan sementara Livewire akan terisi file dengan cepat; oleh karena itu, sangat penting untuk mengonfigurasi S3 agar membersihkan file yang lebih tua dari 24 jam.

Untuk mengonfigurasi perilaku ini, jalankan perintah Artisan berikut dari lingkungan yang menggunakan **S3 bucket** untuk unggahan file:

```shell
php artisan livewire:configure-s3-upload-cleanup

```

Sekarang, setiap file sementara yang lebih tua dari 24 jam akan dibersihkan oleh S3 secara otomatis.

> [!info]
> Jika Anda tidak menggunakan S3 untuk penyimpanan file, Livewire akan menangani pembersihan file secara otomatis dan tidak perlu menjalankan perintah di atas.

## Loading indicators

Meskipun di balik layar `wire:model` untuk unggahan file bekerja secara berbeda dari tipe input `wire:model` lainnya, antarmuka untuk menampilkan **loading indicators** tetap sama.

Anda dapat menampilkan indikator pemuatan yang dikhususkan untuk unggahan file menggunakan `wire:loading`:

```blade
<input type="file" wire:model="photo">

<div wire:loading wire:target="photo">Uploading...</div>

```

Atau lebih sederhana lagi menggunakan atribut otomatis `data-loading` milik Livewire:

```blade
<div>
    <input type="file" wire:model="photo">

    <div class="not-data-loading:hidden">Uploading...</div>
</div>

```

Kini, saat file sedang diunggah, pesan "Uploading..." akan ditampilkan dan kemudian disembunyikan saat unggahan selesai.

[Pelajari lebih lanjut tentang loading states →](https://www.google.com/search?q=/docs/4.x/loading-states)

## Progress indicators

Setiap operasi unggah file Livewire mengirimkan **JavaScript events** pada elemen `<input>` yang bersangkutan, memungkinkan JavaScript kustom untuk menangkap **events** tersebut:

| Event | Deskripsi |
| --- | --- |
| `livewire-upload-start` | Dikirim saat unggahan dimulai |
| `livewire-upload-finish` | Dikirim jika unggahan berhasil diselesaikan |
| `livewire-upload-cancel` | Dikirim jika unggahan dibatalkan sebelum waktunya |
| `livewire-upload-error` | Dikirim jika unggahan gagal |
| `livewire-upload-progress` | Event yang berisi persentase progres unggahan seiring berjalannya proses |

Berikut adalah contoh membungkus unggahan file Livewire di dalam **Alpine component** untuk menampilkan **upload progress bar**:

```blade
<form wire:submit="save">
    <div
        x-data="{ uploading: false, progress: 0 }"
        x-on:livewire-upload-start="uploading = true"
        x-on:livewire-upload-finish="uploading = false"
        x-on:livewire-upload-cancel="uploading = false"
        x-on:livewire-upload-error="uploading = false"
        x-on:livewire-upload-progress="progress = $event.detail.progress"
    >
        <input type="file" wire:model="photo">

        <div x-show="uploading">
            <progress max="100" x-bind:value="progress"></progress>
        </div>
    </div>

    </form>

```

## Cancelling an upload

Jika pengunggahan memakan waktu lama, pengguna mungkin ingin membatalkannya. Anda dapat menyediakan fungsionalitas ini dengan fungsi `$cancelUpload()` Livewire di JavaScript.

Berikut adalah contoh membuat tombol "Cancel Upload" dalam **Livewire component** menggunakan `wire:click` untuk menangani **event** klik:

```blade
<form wire:submit="save">
    <input type="file" wire:model="photo">

    <button type="button" wire:click="$cancelUpload('photo')">Cancel Upload</button>

    </form>

```

Ketika "Cancel upload" ditekan, permintaan unggah file akan dihentikan dan input file akan dikosongkan. Pengguna kini dapat mencoba unggahan lain dengan file yang berbeda.

Alternatifnya, Anda dapat memanggil `cancelUpload(...)` dari Alpine seperti berikut:

```blade
<button type="button" x-on:click="$wire.cancelUpload('photo')">Cancel Upload</button>

```

## JavaScript upload API

Mengintegrasikan pustaka unggah file pihak ketiga sering kali memerlukan kontrol lebih besar daripada elemen `<input type="file" wire:model="...">` sederhana.

Untuk skenario ini, Livewire menyediakan fungsi JavaScript khusus.

Fungsi-fungsi ini berada pada objek **JavaScript component**, yang dapat diakses menggunakan objek `$wire` yang praktis dari dalam **template component** Livewire Anda:

```blade
<script>
    let file = $wire.el.querySelector('input[type="file"]').files[0]

    // Unggah satu file...
    $wire.upload('photo', file, (uploadedFilename) => {
        // Success callback...
    }, () => {
        // Error callback...
    }, (event) => {
        // Progress callback...
        // event.detail.progress berisi angka antara 1 dan 100
    }, () => {
        // Cancelled callback...
    })

    // Unggah banyak file...
    $wire.uploadMultiple('photos', [file], successCallback, errorCallback, progressCallback, cancelledCallback)

    // Hapus satu file dari beberapa file yang diunggah...
    $wire.removeUpload('photos', uploadedFilename, successCallback)

    // Batalkan unggahan...
    $wire.cancelUpload('photos')
</script>

```

## Configuration

Karena Livewire menyimpan semua unggahan file secara sementara sebelum pengembang dapat memvalidasi atau menyimpannya, Livewire menetapkan beberapa perilaku penanganan **default**.

### Global validation

Secara *default*, Livewire akan memvalidasi semua unggahan file sementara dengan aturan berikut: `file|max:12288` (Harus berupa file dan berukuran kurang dari 12MB).

Jika Anda ingin menyesuaikan aturan ini, Anda dapat melakukannya di dalam file `config/livewire.php` aplikasi Anda:

```php
'temporary_file_upload' => [
    // ...
    'rules' => 'file|mimes:png,jpg,pdf|max:102400', // (Maks 100MB, dan hanya menerima PNG, JPEG, dan PDF)
],

```

### Global middleware

**Endpoint** unggahan file sementara diberikan **throttling middleware** secara *default*. Anda dapat menyesuaikan **middleware** apa yang digunakan **endpoint** ini melalui opsi konfigurasi berikut:

```php
'temporary_file_upload' => [
    // ...
    'middleware' => 'throttle:5,1', // Hanya izinkan 5 unggahan per pengguna per menit
],

```

### Temporary upload directory

File sementara diunggah ke direktori `livewire-tmp/` pada **disk** yang ditentukan. Anda dapat menyesuaikan direktori ini melalui opsi konfigurasi berikut:

```php
'temporary_file_upload' => [
    // ...
    'directory' => 'tmp',
],

```

---

## See also

* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Menangani unggahan file dalam formulir
* **[Validation](https://www.google.com/search?q=/docs/4.x/validation)** — Memvalidasi file yang diunggah
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Menampilkan indikator progres unggahan
* **[wire:model](https://www.google.com/search?q=/docs/4.x/wire-model)** — Menghubungkan input file ke properti
