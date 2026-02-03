**Livewire components** sangat sederhana untuk diuji. Karena di balik layar mereka hanyalah class Laravel, mereka dapat diuji menggunakan alat pengujian Laravel yang sudah ada. Namun, Livewire menyediakan banyak utilitas tambahan untuk membuat pengujian **components** Anda menjadi sangat mudah.

Dokumentasi ini akan memandu Anda melalui pengujian **Livewire components** menggunakan **Pest** sebagai **testing framework** yang direkomendasikan, meskipun Anda juga dapat menggunakan **PHPUnit** jika Anda lebih suka.

## Installing Pest

[Pest](https://pestphp.com/) adalah **testing framework** PHP yang menyenangkan dengan fokus pada kesederhanaan. Ini adalah cara yang direkomendasikan untuk menguji **Livewire components** di Livewire 4.

Untuk menginstal **Pest** di aplikasi Laravel Anda, pertama-tama hapus **PHPUnit** (jika terinstal) dan butuhkan (**require**) **Pest**:

```shell
composer remove phpunit/phpunit
composer require pestphp/pest --dev --with-all-dependencies

```

Selanjutnya, inisialisasi **Pest** di proyek Anda:

```shell
./vendor/bin/pest --init

```

Ini akan membuat file konfigurasi `tests/Pest.php` di proyek Anda.

Untuk instruksi instalasi yang lebih rinci, lihat [dokumentasi instalasi Pest](https://pestphp.com/docs/installation).

## Configuring Pest untuk view-based components

Jika Anda menulis tes bersama dengan **view-based components** Anda (**single-file** atau **multi-file**), Anda perlu mengonfigurasi **Pest** untuk mengenali file-file tes tersebut.

Pertama, perbarui file `tests/Pest.php` Anda untuk menyertakan direktori `resources/views`:

```php
pest()->extend(Tests\TestCase::class)
    // ...
    ->in('Feature', '../resources/views');

```

Ini memberitahu **Pest** untuk menggunakan base class `TestCase` Anda untuk tes yang ditemukan baik di direktori `tests/Feature` maupun di mana pun di dalam `resources/views`.

Selanjutnya, perbarui file `phpunit.xml` Anda untuk menyertakan sebuah **test suite** untuk tes **component**:

```xml
<testsuite name="Components">
    <directory suffix=".test.php">resources/views</directory>
</testsuite>

```

Sekarang **Pest** akan mengenali dan menjalankan tes yang terletak di sebelah **components** Anda saat Anda menjalankan `./vendor/bin/pest`.

## Creating your first test

Anda dapat menghasilkan sebuah file tes bersama dengan sebuah **component** dengan menambahkan flag `--test` pada perintah `make:livewire`:

```shell
php artisan make:livewire post.create --test

```

Untuk **multi-file components**, ini akan membuat sebuah file tes di `resources/views/components/post/create.test.php`:

```php
<?php

use Livewire\Livewire;

it('renders successfully', function () {
    Livewire::test('post.create')
        ->assertStatus(200);
});

```

Untuk **class-based components**, ini akan membuat file tes **PHPUnit** di `tests/Feature/Livewire/Post/CreateTest.php`. Anda dapat mengubahnya ke sintaks **Pest** atau tetap menggunakan **PHPUnit**—keduanya berfungsi dengan baik dengan Livewire.

### Testing a page contains a component

Tes Livewire paling sederhana yang dapat Anda tulis adalah menegaskan (**asserting**) bahwa sebuah **endpoint** tertentu menyertakan dan berhasil merender sebuah **Livewire component**.

```php
it('component exists on the page', function () {
    $this->get('/posts/create')
        ->assertSeeLivewire('post.create');
});

```

> [!tip] Smoke tests memberikan nilai yang besar
> Tes seperti ini disebut "**smoke tests**"—mereka memastikan tidak ada masalah fatal dalam aplikasi Anda. Meskipun sederhana, tes ini memberikan nilai yang sangat besar karena membutuhkan sangat sedikit pemeliharaan dan memberi Anda tingkat kepercayaan dasar bahwa halaman Anda berhasil dirender.

## Browser testing

Pest v4 menyertakan dukungan **browser testing** pihak pertama yang didukung oleh **Playwright**. Ini memungkinkan Anda untuk menguji **Livewire components** Anda di browser yang nyata, berinteraksi dengan mereka persis seperti yang dilakukan pengguna.

### Installing browser testing

Pertama, instal plugin browser **Pest**:

```shell
composer require pestphp/pest-plugin-browser --dev

```

Selanjutnya, instal **Playwright** via npm:

```shell
npm install playwright@latest
npx playwright install

```

Untuk dokumentasi pengujian browser yang lengkap, lihat [panduan browser testing Pest](https://pestphp.com/docs/browser-testing).

### Writing browser tests

Alih-alih menggunakan `Livewire::test()`, Anda dapat menggunakan `Livewire::visit()` untuk menguji **component** Anda di browser yang nyata:

```php
it('can create a new post', function () {
    Livewire::visit('post.create')
        ->type('[wire\:model="title"]', 'My first post')
        ->type('[wire\:model="content"]', 'This is the content')
        ->press('Save')
        ->assertSee('Post created successfully');
});

```

**Browser tests** lebih lambat daripada **unit tests** tetapi memberikan kepercayaan **end-to-end** bahwa **components** Anda berfungsi sesuai harapan di lingkungan browser yang nyata.

Untuk daftar lengkap **assertions** pengujian browser yang tersedia, lihat [Pest browser testing assertions](https://pestphp.com/docs/browser-testing#content-available-assertions).

> [!info] Kapan menggunakan browser tests
> Gunakan **browser tests** untuk alur pengguna yang kritis dan interaksi yang kompleks. Untuk sebagian besar pengujian **component**, pendekatan standar `Livewire::test()` lebih cepat dan sudah cukup.

## Testing views

Livewire menyediakan `assertSee()` untuk memverifikasi bahwa teks muncul dalam output **component** Anda yang dirender:

```php
use App\Models\Post;

it('displays posts', function () {
    Post::factory()->create(['title' => 'My first post']);
    Post::factory()->create(['title' => 'My second post']);

    Livewire::test('show-posts')
        ->assertSee('My first post')
        ->assertSee('My second post');
});

```

### Asserting view data

Terkadang sangat membantu untuk menguji data yang diteruskan ke dalam **view** daripada output yang dirender:

```php
use App\Models\Post;

it('passes all posts to the view', function () {
    Post::factory()->count(3)->create();

    Livewire::test('show-posts')
        ->assertViewHas('posts', function ($posts) {
            return count($posts) === 3;
        });
});

```

Untuk **assertions** sederhana, Anda dapat meneruskan nilai yang diharapkan secara langsung:

```php
Livewire::test('show-posts')
    ->assertViewHas('postCount', 3);

```

## Testing with authentication

Sebagian besar aplikasi mengharuskan pengguna untuk log in. Daripada melakukan autentikasi secara manual di awal setiap tes, gunakan metode `actingAs()`:

```php
use App\Models\User;
use App\Models\Post;

it('user only sees their own posts', function () {
    $user = User::factory()
        ->has(Post::factory()->count(3))
        ->create();

    $stranger = User::factory()
        ->has(Post::factory()->count(2))
        ->create();

    Livewire::actingAs($user)
        ->test('show-posts')
        ->assertViewHas('posts', function ($posts) {
            return count($posts) === 3;
        });
});

```

## Testing properties

Livewire menyediakan utilitas untuk menetapkan (**setting**) dan menegaskan (**asserting**) **component properties**.

Gunakan `set()` untuk memperbarui **properties** dan `assertSet()` untuk memverifikasi nilainya:

```php
it('can set the title property', function () {
    Livewire::test('post.create')
        ->set('title', 'My amazing post')
        ->assertSet('title', 'My amazing post');
});

```

### Initializing properties

**Components** sering kali menerima data dari **parent components** atau parameter **route**. Teruskan data ini sebagai parameter kedua ke `Livewire::test()`:

```php
use App\Models\Post;

it('title field is populated when editing', function () {
    $post = Post::factory()->create([
        'title' => 'Existing post title',
    ]);

    Livewire::test('post.edit', ['post' => $post])
        ->assertSet('title', 'Existing post title');
});

```

### Setting URL parameters

Jika **component** Anda menggunakan [fitur URL Livewire](https://www.google.com/search?q=/docs/4.x/url) untuk melacak **state** dalam **query strings**, gunakan `withQueryParams()` untuk mensimulasikan parameter URL:

```php
use App\Models\Post;

it('can search posts via url query string', function () {
    Post::factory()->create(['title' => 'Laravel testing']);
    Post::factory()->create(['title' => 'Vue components']);

    Livewire::withQueryParams(['search' => 'Laravel'])
        ->test('search-posts')
        ->assertSee('Laravel testing')
        ->assertDontSee('Vue components');
});

```

### Setting cookies

Gunakan `withCookie()` atau `withCookies()` untuk menetapkan **cookies** untuk tes Anda:

```php
it('loads discount token from cookie', function () {
    Livewire::withCookies(['discountToken' => 'SUMMER2024'])
        ->test('cart')
        ->assertSet('discountToken', 'SUMMER2024');
});

```

## Calling actions

Gunakan metode `call()` untuk memicu **component actions** dalam tes Anda:

```php
use App\Models\Post;

it('can create a post', function () {
    expect(Post::count())->toBe(0);

    Livewire::test('post.create')
        ->set('title', 'My new post')
        ->set('content', 'Post content here')
        ->call('save');

    expect(Post::count())->toBe(1);
});

```

> [!tip] Pest expectations
> Contoh-contoh di atas menggunakan sintaks `expect()` milik **Pest** untuk **assertions**. Untuk daftar lengkap **expectations** yang tersedia, lihat [dokumentasi expectations Pest](https://pestphp.com/docs/expectations).

Anda dapat meneruskan parameter ke **actions**:

```php
Livewire::test('post.show')
    ->call('deletePost', $postId);

```

### Testing validation

Tegaskan bahwa **validation errors** telah dilemparkan menggunakan `assertHasErrors()`:

```php
it('title field is required', function () {
    Livewire::test('post.create')
        ->set('title', '')
        ->call('save')
        ->assertHasErrors('title');
});

```

Uji aturan validasi tertentu:

```php
it('title must be at least 3 characters', function () {
    Livewire::test('post.create')
        ->set('title', 'ab')
        ->call('save')
        ->assertHasErrors(['title' => ['min:3']]);
});

```

### Testing authorization

Pastikan pemeriksaan otorisasi berfungsi dengan benar menggunakan `assertUnauthorized()` dan `assertForbidden()`:

```php
use App\Models\User;
use App\Models\Post;

it('cannot update another users post', function () {
    $user = User::factory()->create();
    $stranger = User::factory()->create();
    $post = Post::factory()->for($stranger)->create();

    Livewire::actingAs($user)
        ->test('post.edit', ['post' => $post])
        ->set('title', 'Hacked!')
        ->call('save')
        ->assertForbidden();
});

```

### Testing redirects

Tegaskan bahwa sebuah **action** melakukan sebuah **redirect**:

```php
it('redirects to posts index after creating', function () {
    Livewire::test('post.create')
        ->set('title', 'New post')
        ->set('content', 'Content here')
        ->call('save')
        ->assertRedirect('/posts');
});

```

Anda juga dapat menegaskan **redirects** ke **named routes** atau **page components**:

```php
->assertRedirect(route('posts.index'));
->assertRedirectToRoute('posts.index');

```

### Testing events

Tegaskan bahwa **events** di-**dispatch** dari **component** Anda:

```php
it('dispatches event when post is created', function () {
    Livewire::test('post.create')
        ->set('title', 'New post')
        ->call('save')
        ->assertDispatched('post-created');
});

```

Uji komunikasi **event** antar **components**:

```php
it('updates post count when event is dispatched', function () {
    $badge = Livewire::test('post-count-badge')
        ->assertSee('0');

    Livewire::test('post.create')
        ->set('title', 'New post')
        ->call('save')
        ->assertDispatched('post-created');

    $badge->dispatch('post-created')
        ->assertSee('1');
});

```

Tegaskan **events** di-**dispatch** dengan parameter tertentu:

```php
it('dispatches notification when deleting post', function () {
    Livewire::test('post.show')
        ->call('delete', postId: 3)
        ->assertDispatched('notify', message: 'Post deleted');
});

```

Untuk **assertions** yang kompleks, gunakan sebuah **closure**:

```php
it('dispatches event with correct data', function () {
    Livewire::test('post.show')
        ->call('delete', postId: 3)
        ->assertDispatched('notify', function ($event, $params) {
            return ($params['message'] ?? '') === 'Post deleted';
        });
});

```

## Using PHPUnit

Meskipun **Pest** direkomendasikan, Anda benar-benar dapat menggunakan **PHPUnit** untuk menguji **Livewire components**. Semua utilitas pengujian yang sama berfungsi dengan sintaks **PHPUnit**.

Berikut adalah contoh **PHPUnit** untuk perbandingan:

```php
<?php

namespace Tests\Feature\Livewire;

use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_can_create_post()
    {
        $this->assertEquals(0, Post::count());

        Livewire::test('post.create')
            ->set('title', 'My new post')
            ->set('content', 'Post content')
            ->call('save');

        $this->assertEquals(1, Post::count());
    }

    public function test_title_is_required()
    {
        Livewire::test('post.create')
            ->set('title', '')
            ->call('save')
            ->assertHasErrors('title');
    }
}

```

Semua fitur yang didokumentasikan di halaman ini berfungsi secara identik dengan **PHPUnit**—cukup gunakan sintaks **assertion PHPUnit** alih-alih milik **Pest**.

> [!tip] Consider trying Pest
> Jika Anda tertarik untuk menjelajahi sintaks dan fitur **Pest** yang lebih elegan, kunjungi [pestphp.com](https://pestphp.com/) untuk mempelajari lebih lanjut.

## All available testing methods

Di bawah ini adalah referensi komprehensif dari setiap **testing method** Livewire yang tersedia untuk Anda:

### Setup methods

| Method | Description |
| --- | --- |
| `Livewire::test('post.create')` | Tes **component** `post.create` |
| `Livewire::test(UpdatePost::class, ['post' => $post])` | Tes **component** `UpdatePost` dengan **parameters** yang diteruskan ke `mount()` |
| `Livewire::actingAs($user)` | Set **authenticated user** untuk tes tersebut |
| `Livewire::withQueryParams(['search' => '...'])` | Set **URL query parameters** (contoh: `?search=...`) |
| `Livewire::withCookie('name', 'value')` | Set sebuah **cookie** untuk tes tersebut |
| `Livewire::withCookies(['color' => 'blue', 'name' => 'Taylor'])` | Set beberapa **cookies** sekaligus |
| `Livewire::withHeaders(['X-Header' => 'value'])` | Set **custom headers** |
| `Livewire::withoutLazyLoading()` | Nonaktifkan **lazy loading** untuk semua **components** dalam tes ini |

### Interacting with components

| Method | Description |
| --- | --- |
| `set('title', '...')` | Set **property** `title` ke nilai yang diberikan |
| `set(['title' => '...', 'content' => '...'])` | Set beberapa **properties** menggunakan sebuah **array** |
| `toggle('sortAsc')` | Ganti (**toggle**) sebuah **boolean property** antara `true` dan `false` |
| `call('save')` | Panggil **action**/**method** `save` |
| `call('remove', $postId)` | Panggil sebuah **method** dengan **parameters** |
| `refresh()` | Picu **re-render** pada **component** |
| `dispatch('post-created')` | **Dispatch** sebuah **event** dari **component** |
| `dispatch('post-created', postId: $post->id)` | **Dispatch** sebuah **event** dengan **parameters** |

### Assertions

| Method | Description |
| --- | --- |
| `assertSet('title', '...')` | **Assert** bahwa sebuah **property** sama dengan nilai yang diberikan |
| `assertNotSet('title', '...')` | **Assert** bahwa sebuah **property** tidak sama dengan nilai yang diberikan |
| `assertCount('posts', 3)` | **Assert** bahwa sebuah **property** berisi 3 item |
| `assertSee('...')` | **Assert** bahwa **rendered HTML** berisi teks yang diberikan |
| `assertDontSee('...')` | **Assert** bahwa **rendered HTML** tidak berisi teks yang diberikan |
| `assertSeeHtml('<div>...</div>')` | **Assert** bahwa **raw HTML** ada dalam **rendered output** |
| `assertDontSeeHtml('<div>...</div>')` | **Assert** bahwa **raw HTML** tidak ada dalam **rendered output** |
| `assertSeeInOrder(['first', 'second'])` | **Assert** bahwa **strings** muncul berurutan dalam **rendered output** |
| `assertDispatched('post-created')` | **Assert** bahwa sebuah **event** telah di-**dispatched** |
| `assertNotDispatched('post-created')` | **Assert** bahwa sebuah **event** tidak di-**dispatched** |
| `assertHasErrors('title')` | **Assert** bahwa **validation** gagal untuk sebuah **property** |
| `assertHasErrors(['title' => ['required', 'min:6']])` | **Assert** bahwa **validation rules** tertentu gagal |
| `assertHasNoErrors('title')` | **Assert** bahwa tidak ada **validation errors** untuk sebuah **property** |
| `assertRedirect()` | **Assert** bahwa sebuah **redirect** telah dipicu |
| `assertRedirect('/posts')` | **Assert** sebuah **redirect** ke URL tertentu |
| `assertRedirectToRoute('posts.index')` | **Assert** sebuah **redirect** ke **named route** |
| `assertNoRedirect()` | **Assert** bahwa tidak ada **redirect** yang dipicu |
| `assertViewHas('posts')` | **Assert** bahwa data telah diteruskan ke **view** |
| `assertViewHas('postCount', 3)` | **Assert** bahwa **view data** memiliki nilai tertentu |
| `assertViewHas('posts', function ($posts) { ... })` | **Assert** bahwa **view data** lolos **custom validation** |
| `assertViewIs('livewire.show-posts')` | **Assert** bahwa **view** tertentu telah di-**render** |
| `assertFileDownloaded()` | **Assert** bahwa sebuah **file download** telah dipicu |
| `assertFileDownloaded($filename)` | **Assert** bahwa **file** tertentu telah di-**download** |
| `assertUnauthorized()` | **Assert** bahwa sebuah **authorization exception** dilemparkan (401) |
| `assertForbidden()` | **Assert** bahwa akses dilarang (**forbidden**) (403) |
| `assertStatus(500)` | **Assert** bahwa **status code** tertentu dikembalikan |

---

## See also

* **[Actions](https://www.google.com/search?q=/docs/4.x/actions)** — Tes **component actions** dan interaksinya
* **[Forms](https://www.google.com/search?q=/docs/4.x/forms)** — Tes **form submissions** dan **validation**
* **[Events](https://www.google.com/search?q=/docs/4.x/events)** — Tes **event dispatching** dan **listening**
* **[Components](https://www.google.com/search?q=/docs/4.x/components)** — Pelajari tentang struktur **component** yang dapat diuji
