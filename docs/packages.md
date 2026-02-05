Untuk menyertakan komponen Livewire dalam sebuah Laravel **package**, Anda perlu mendaftarkannya di dalam **service provider** milik **package** Anda.

## Single-file dan multi-file components

Untuk **single-file components** (SFC) dan **multi-file components** (MFC), gunakan metode `addNamespace` di dalam metode `boot()` pada **service provider** Anda:

```php
use Livewire\Livewire;

public function boot(): void
{
    Livewire::addNamespace(
        namespace: 'mypackage',
        viewPath: __DIR__ . '/../resources/views/livewire',
    );
}

```

Ini akan mendaftarkan semua komponen SFC dan MFC di direktori `resources/views/livewire` milik **package** Anda di bawah **namespace** `mypackage`.

**Penggunaan:**

```blade
<livewire:mypackage::counter />
<livewire:mypackage::users.table />

```

---

## Class-based components

Untuk **class-based components**, Anda perlu memberikan parameter tambahan dan mendaftarkan **views** Anda ke Laravel:

```php
use Livewire\Livewire;

public function boot(): void
{
    Livewire::addNamespace(
        namespace: 'mypackage',
        classNamespace: 'MyVendor\\MyPackage\\Livewire',
        classPath: __DIR__ . '/Livewire',
        classViewPath: __DIR__ . '/../resources/views/livewire',
    );

    $this->loadViewsFrom(__DIR__ . '/../resources/views', 'my-package');
}

```

Metode `render()` pada komponen Anda harus merujuk ke **view** menggunakan sintaks **package namespace** Laravel:

```php
public function render()
{
    return view('my-package::livewire.counter');
}

```

**Penggunaan:**

```blade
<livewire:mypackage::counter />

```

---

## Penamaan File (File naming)

Awalan emoji ⚡ yang digunakan pada nama file komponen Livewire dapat menyebabkan masalah dengan **Composer** saat mempublikasikan **package**. Untuk pengembangan **package**, hindari penggunaan emoji petir tersebut pada nama file komponen Anda—gunakan `counter.blade.php` alih-alih `⚡counter.blade.php`.
