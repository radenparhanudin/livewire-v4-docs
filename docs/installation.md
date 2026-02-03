Livewire adalah sebuah Laravel **package**, jadi Anda perlu memiliki aplikasi Laravel yang sudah berjalan sebelum Anda dapat menginstal dan menggunakan Livewire. Jika Anda memerlukan bantuan dalam menyiapkan aplikasi Laravel baru, silakan lihat [official Laravel documentation](https://laravel.com/docs/installation).

## Prerequisites

Sebelum menginstal Livewire, pastikan Anda memiliki:

* Laravel versi 10 atau lebih baru
* PHP versi 8.1 atau lebih baru

## Install Livewire

Untuk menginstal Livewire, buka terminal Anda dan navigasikan ke direktori aplikasi Laravel Anda, kemudian jalankan perintah berikut:

```shell
composer require livewire/livewire

```

Itu saja! Livewire menggunakan Laravel **package auto-discovery**, sehingga tidak diperlukan **setup** tambahan.

**Ready untuk membangun komponen pertama Anda?** Head over ke [Quickstart guide](https://www.google.com/search?q=/docs/4.x/quickstart) untuk membuat **component** Livewire pertama Anda dalam hitungan menit.

## Create a layout file

Saat menggunakan Livewire **components** sebagai **full pages**, Anda akan membutuhkan sebuah **layout file**. Anda dapat men-generate satu menggunakan **command** Livewire:

```shell
php artisan livewire:layout

```

Ini akan membuat sebuah **layout file** pada `resources/views/layouts/app.blade.php` dengan isi sebagai berikut:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        {{ $slot }}

        @livewireScripts
    </body>
</html>

```

**Directives** `@livewireStyles` dan `@livewireScripts` menyertakan **assets** JavaScript dan CSS yang diperlukan agar Livewire dapat berfungsi. Livewire men-**bundle** Alpine.js bersama JavaScript-nya, sehingga keduanya di-**load** bersamaan.

> [!info] Asset injection bersifat otomatis
> Bahkan tanpa **directives** ini, Livewire akan secara otomatis meng-**inject assets**-nya ke dalam halaman yang berisi Livewire **components**. Namun, menyertakan **directives** ini memberi Anda kontrol eksplisit atas di mana **assets** ditempatkan, yang dapat membantu untuk **performance optimization** atau kompatibilitas dengan **package** lain.

## Publishing the configuration file

Livewire adalah "**zero-config**", artinya Anda dapat menggunakannya dengan mengikuti konvensi tanpa **configuration** tambahan apa pun. Namun, jika diperlukan, Anda dapat mem-**publish** dan melakukan kustomisasi pada Livewire **configuration file**:

```shell
php artisan livewire:config

```

Ini akan membuat sebuah file `livewire.php` baru di direktori `config` aplikasi Laravel Anda di mana Anda dapat menyesuaikan berbagai **Livewire settings**.

---

# Advanced configuration

Bagian berikut mencakup skenario tingkat lanjut yang tidak akan dibutuhkan oleh sebagian besar aplikasi. Hanya konfigurasikan ini jika Anda memiliki **requirement** khusus.

## Manually bundling Livewire and Alpine

**Kapan Anda membutuhkan ini:** Jika Anda ingin menggunakan Alpine.js **plugins** atau membutuhkan kontrol yang halus kapan Alpine dan Livewire di-**initialize**.

Secara **default**, Livewire secara otomatis me-**load** Alpine.js yang di-**bundle** dengan JavaScript-nya. Namun, jika Anda perlu meregistrasi Alpine **plugins** atau menyesuaikan urutan **initialization**, Anda dapat melakukan **bundle** Livewire dan Alpine secara manual menggunakan JavaScript **build tool** Anda.

Pertama, tambahkan **directive** `@livewireScriptConfig` ke **layout file** Anda:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        {{ $slot }}

        @livewireScriptConfig
    </body>
</html>

```

**Directive** `@livewireScriptConfig` meng-**inject configuration** dan **runtime globals** yang dibutuhkan Livewire, tetapi tanpa JavaScript Livewire dan Alpine yang sebenarnya (karena Anda melakukan **bundling** sendiri). Ganti `@livewireScripts` dengan `@livewireScriptConfig` saat melakukan **manual bundling**.

Selanjutnya, **import** dan jalankan Livewire dan Alpine di file `resources/js/app.js` Anda:

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';
import Clipboard from '@ryangjchandler/alpine-clipboard'

Alpine.plugin(Clipboard)

Livewire.start()

```

> [!tip] Rebuild assets setelah Livewire updates
> Saat melakukan **manual bundling**, ingatlah untuk me-**rebuild assets** JavaScript Anda (`npm run build`) setiap kali Anda memperbarui Livewire melalui Composer.

## Customizing Livewire's update endpoint

**Kapan Anda membutuhkan ini:** Jika aplikasi Anda menggunakan **route prefixes** untuk lokalisasi (seperti `/en/`, `/fr/`) atau **multi-tenancy** (seperti `/tenant-1/`, `/tenant-2/`), Anda mungkin perlu menyesuaikan Livewire **update endpoint** agar sesuai dengan struktur **routing** Anda.

Secara **default**, Livewire mengirimkan **component updates** ke sebuah **endpoint** berbasis **hash** seperti `/livewire-{hash}/update`, di mana `{hash}` diturunkan dari `APP_KEY` aplikasi Anda. Untuk menyesuaikan ini, registrasikan **route** Anda sendiri di sebuah **service provider** (biasanya `App\Providers\AppServiceProvider`):

```php
use Livewire\Livewire;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Livewire::setUpdateRoute(function ($handle) {
            return Route::post('/custom/livewire/update', $handle);
        });
    }
}

```

Anda juga dapat menambahkan **middleware** ke **update route** tersebut:

```php
Livewire::setUpdateRoute(function ($handle) {
    return Route::post('/custom/livewire/update', $handle)
        ->middleware(['web', 'auth']);
});

```

## Customizing the JavaScript asset URL

**Kapan Anda membutuhkan ini:** Jika aplikasi Anda menggunakan **route prefixes** untuk lokalisasi atau **multi-tenancy**, Anda mungkin perlu menyesuaikan dari mana Livewire menyajikan JavaScript-nya agar sesuai dengan struktur **routing** Anda.

Secara **default**, Livewire menyajikan JavaScript-nya dari sebuah **endpoint** berbasis **hash** seperti `/livewire-{hash}/livewire.js`, di mana `{hash}` diturunkan dari `APP_KEY` aplikasi Anda. **Path** unik per instalasi ini membuatnya lebih sulit untuk menargetkan aplikasi Livewire dengan **automated scanners**.

Untuk menyesuaikan ini, registrasikan **route** Anda sendiri di sebuah **service provider**:

```php
use Livewire\Livewire;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Livewire::setScriptRoute(function ($handle) {
            return Route::get('/custom/livewire/livewire.js', $handle);
        });
    }
}

```

> [!note] Setting a custom route menggunakan static path
> Saat Anda melakukan kustomisasi pada **script route**, ia akan menggunakan **path** tepat yang Anda tentukan alih-alih **default** berbasis **hash**.

## Publishing Livewire's assets ke public directory

**Kapan Anda membutuhkan ini:** Jika Anda ingin menyajikan JavaScript Livewire melalui **web server** Anda secara langsung (misalnya, untuk **CDN distribution** atau strategi **caching** tertentu) alih-alih melalui Laravel **routing**.

Anda dapat mem-**publish assets** JavaScript Livewire ke direktori `public` Anda:

```bash
php artisan livewire:publish --assets

```

Untuk memastikan **assets** tetap **up-to-date** saat Anda memperbarui Livewire, tambahkan ini ke `composer.json` Anda:

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=livewire:assets --ansi --force"
        ]
    }
}

```

> [!warning] Most applications don't need this
> Mempublikasikan **assets** jarang sekali diperlukan. Hanya lakukan ini jika Anda memiliki **architectural requirement** khusus yang mencegah Laravel menyajikan **assets** secara dinamis.

## Disabling automatic asset injection

**Kapan Anda membutuhkan ini:** Jika Anda ingin kontrol penuh atas kapan dan bagaimana Livewire **assets** di-**load**, Anda dapat menonaktifkan **automatic injection**.

Perbarui opsi **configuration** `inject_assets` di file `config/livewire.php` Anda:

```php
'inject_assets' => false,

```

Saat dinonaktifkan, Anda harus menyertakan `@livewireStyles` dan `@livewireScripts** secara manual di **layouts** Anda, atau Livewire tidak akan berfungsi.

Alternatif lainnya, Anda dapat memaksakan **asset injection** pada halaman tertentu:

```php
\Livewire\Livewire::forceAssetInjection();

```

Panggil ini di dalam **route** atau **controller** di mana Anda ingin memastikan **assets** di-**inject**.

---

# Troubleshooting

## Livewire JavaScript tidak loading (404 error)

**Symptom:** File JavaScript Livewire mengembalikan **error 404**, atau fitur Livewire tidak berfungsi.

Livewire menyajikan JavaScript-nya dari sebuah **endpoint** berbasis **hash** seperti `/livewire-{hash}/livewire.js`, di mana `{hash}` diturunkan dari `APP_KEY` aplikasi Anda. **Path** unik ini bervariasi per instalasi.

**Common causes:**

**Nginx configuration memblokir route:**
Jika Anda menggunakan Nginx dengan kustom **configuration**, ia mungkin memblokir **dynamic Livewire routes** milik Laravel. Anda dapat melakukan salah satu dari:

* Konfigurasikan Nginx untuk meneruskan **requests** yang cocok dengan `/livewire-*/` ke Laravel (misal, `location ~ ^/livewire-[a-f0-9]+/ { try_files $uri $uri/ /index.php?$query_string; }`)
* [Manually bundle Livewire](https://www.google.com/search?q=%23manually-bundling-livewire-and-alpine) untuk menghindari penyajian melalui Laravel.
* [Publish Livewire's assets](https://www.google.com/search?q=%23publishing-livewires-assets-to-public-directory) untuk menyajikannya secara langsung dari **web server** Anda.

**Route caching:**
Jika Anda telah menjalankan `php artisan route:cache`, Laravel mungkin tidak mengenali **Livewire routes**. Bersihkan **cache**:

```shell
php artisan route:clear

```

**Missing @livewireScripts:**
Jika Anda telah menonaktifkan **automatic asset injection**, pastikan `@livewireScripts` ada di **layout file** Anda sebelum `</body>`.

## Alpine.js tidak tersedia pada halaman tanpa Livewire components

**Symptom:** Anda ingin menggunakan Alpine.js pada halaman yang tidak memiliki Livewire **components** apa pun.

**Solution:** Karena Alpine di-**bundle** dengan Livewire, Anda perlu menyertakan `@livewireScripts` bahkan pada halaman tanpa Livewire **components**:

```blade
<!DOCTYPE html>
<html>
    <head>
        @livewireStyles
    </head>
    <body>
        <div x-data="{ open: false }">
            <button @click="open = !open">Toggle</button>
        </div>

        @livewireScripts
    </body>
</html>

```

Alternatif lainnya, [manually bundle Livewire and Alpine](https://www.google.com/search?q=%23manually-bundling-livewire-and-alpine) dan **import** Alpine di JavaScript Anda.

## Components tidak updating atau errors di browser console

**Check the berikut ini:**

* Pastikan `@livewireStyles` ada di dalam `<head>` dari **layout** Anda
* Pastikan `@livewireScripts` ada sebelum `</body>` di **layout** Anda
* Periksa **developer console** browser Anda untuk JavaScript **errors**
* Verifikasi Anda menjalankan versi PHP yang didukung (8.1+) dan versi Laravel (10+)
* Bersihkan **application cache** Anda: `php artisan cache:clear`

Jika masalah berlanjut, periksa [troubleshooting documentation](https://www.google.com/search?q=/docs/4.x/troubleshooting) untuk langkah-langkah **debugging** yang lebih rinci.
