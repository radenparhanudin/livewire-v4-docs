Atribut `#[Transition]` mengonfigurasi perilaku **view transition** untuk metode **action**, memungkinkan Anda untuk mengatur tipe transisi atau melewatkan transisi sepenuhnya.

## Penggunaan dasar

Terapkan atribut `#[Transition]` pada metode **action** yang harus memicu animasi transisi tertentu:

```php
<?php

use Livewire\Attributes\Transition;
use Livewire\Component;

class Wizard extends Component
{
    public $step = 1;

    #[Transition(type: 'forward')] // [tl! highlight]
    public function next()
    {
        $this->step++;
    }

    #[Transition(type: 'backward')] // [tl! highlight]
    public function previous()
    {
        $this->step--;
    }
}

```

```blade
<div>
    <div wire:transition="content">
        Step {{ $step }}
    </div>

    <button wire:click="previous">Back</button>
    <button wire:click="next">Next</button>
</div>

```

Tipe transisi dapat ditargetkan di CSS menggunakan pemilih (*selector*) `:active-view-transition-type()`:

```css
html:active-view-transition-type(forward) {
    &::view-transition-old(content) {
        animation: 300ms ease-out slide-out-left;
    }
    &::view-transition-new(content) {
        animation: 300ms ease-in slide-in-right;
    }
}

html:active-view-transition-type(backward) {
    &::view-transition-old(content) {
        animation: 300ms ease-out slide-out-right;
    }
    &::view-transition-new(content) {
        animation: 300ms ease-in slide-in-left;
    }
}

@keyframes slide-out-left {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(-100%); opacity: 0; }
}

@keyframes slide-in-right {
    from { transform: translateX(100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}

@keyframes slide-out-right {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100%); opacity: 0; }
}

@keyframes slide-in-left {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}

```

---

## Melewatkan transisi (Skipping transitions)

Gunakan `skip: true` untuk menonaktifkan transisi pada **actions** tertentu:

```php
#[Transition(skip: true)]
public function reset()
{
    $this->step = 1;
}

```

Ini berguna untuk **actions** seperti "reset" atau "cancel" yang harus diperbarui secara instan tanpa animasi.

---

## Parameter

| Parameter | Tipe | Deskripsi |
| --- | --- | --- |
| `type` | `string` | Nama tipe transisi (misal: `'forward'`, `'backward'`) |
| `skip` | `bool` | Atur ke `true` untuk menonaktifkan transisi pada action ini |

---

## Pendekatan alternatif

### Menggunakan transition()

Untuk tipe transisi dinamis yang bergantung pada logika saat aplikasi berjalan (*runtime*), gunakan metode `transition()` sebagai gantinya:

```php
public function goToStep($step)
{
    $this->transition(type: $step > $this->step ? 'forward' : 'backward');

    $this->step = $step;
}

```

### Menggunakan skipTransition()

Anda juga dapat melewatkan transisi secara imperatif:

```php
public function reset()
{
    $this->skipTransition();

    $this->step = 1;
}

```

---

## Pelajari lebih lanjut

Untuk informasi lebih lanjut tentang **view transitions**, lihat [dokumentasi wire:transition](https://www.google.com/search?q=/docs/4.x/wire-transition).
