`wire:transition` memungkinkan animasi yang mulus saat elemen muncul, hilang, atau berubah menggunakan browser native [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API).

Berbeda dengan pustaka animasi berbasis JavaScript, View Transitions dipercepat oleh perangkat keras (*hardware-accelerated*) dan ditangani secara native oleh browser, menghasilkan animasi yang lebih mulus dengan *overhead* yang lebih sedikit.

## Basic usage

Tambahkan `wire:transition` ke elemen apa pun yang mungkin ditambah, dihapus, atau diubah selama pembaruan Livewire:

```php
class ShowPost extends Component
{
    public Post $post;

    public $showComments = false;
}

```

```blade
<div>
    <button wire:click="$toggle('showComments')">Toggle comments</button>

    @if ($showComments)
        <div wire:transition> @foreach ($post->comments as $comment)
                <div>{{ $comment->body }}</div>
            @endforeach
        </div>
    @endif
</div>

```

Ketika komentar muncul atau hilang, browser akan melakukan *crossfade* secara mulus alih-alih menampilkannya atau menyembunyikannya secara tiba-tiba.

---

## Named transitions

Secara default, Livewire menetapkan `view-transition-name` `match-element` ke elemen dengan `wire:transition`. Anda dapat memberikan nama kustom untuk mengaktifkan efek transisi yang lebih canggih:

```blade
<div wire:transition="sidebar">...</div>

```

Ini mengatur properti CSS `view-transition-name` elemen menjadi `sidebar`, yang dapat Anda targetkan dengan CSS untuk animasi kustom.

---

## Menyesuaikan animasi dengan CSS

View Transitions dikendalikan sepenuhnya melalui CSS. Anda dapat menyesuaikan animasi dengan menargetkan *pseudo-elements* view-transition:

```css
/* Sesuaikan transisi untuk elemen tertentu */
::view-transition-old(sidebar) {
    animation: 300ms ease-out slide-out;
}

::view-transition-new(sidebar) {
    animation: 300ms ease-in slide-in;
}

@keyframes slide-out {
    to { transform: translateX(-100%); }
}

@keyframes slide-in {
    from { transform: translateX(100%); }
}

```

View Transitions API menyediakan tiga *pseudo-elements* yang dapat Anda beri gaya:

* `::view-transition-old(name)` — Snapshot elemen yang keluar
* `::view-transition-new(name)` — Snapshot elemen yang masuk
* `::view-transition-group(name)` — Kontainer untuk kedua snapshot tersebut

---

## Transition types

Untuk skenario yang lebih kompleks seperti *step wizards* di mana Anda memerlukan animasi berbeda berdasarkan arah, Anda dapat menggunakan **transition types**. Ini memungkinkan Anda menganimasi "maju" (*forward*) dan "mundur" (*backward*) secara berbeda.

Gunakan metode `$this->transition()` untuk menetapkan tipe transisi:

```php
class Wizard extends Component
{
    public $step = 1;

    public function goToStep($step)
    {
        $this->transition(type: $step > $this->step ? 'forward' : 'backward');

        $this->step = $step;
    }
}

```

Kemudian targetkan tipe tersebut di CSS Anda menggunakan selektor `:active-view-transition-type()`:

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

```

Untuk metode yang selalu bertransisi ke arah yang sama, Anda dapat menggunakan atribut `#[Transition]` sebagai gantinya:

```php
use Livewire\Attributes\Transition;

class Wizard extends Component
{
    public $step = 1;

    #[Transition(type: 'forward')]
    public function next()
    {
        $this->step++;
    }

    #[Transition(type: 'backward')]
    public function previous()
    {
        $this->step--;
    }
}

```

---

## Melewatkan transisi (Skipping transitions)

Terkadang Anda mungkin ingin menonaktifkan transisi untuk tindakan tertentu—misalnya, tombol "reset" yang harus langsung melompat ke langkah pertama tanpa animasi.

Gunakan `$this->skipTransition()` untuk menonaktifkan transisi pada request saat ini:

```php
public function reset()
{
    $this->skipTransition();

    $this->step = 1;
}

```

Atau gunakan atribut `#[Transition]` dengan `skip: true`:

```php
use Livewire\Attributes\Transition;

#[Transition(skip: true)]
public function reset()
{
    $this->step = 1;
}

```

---

## Menghormati reduced motion

Livewire secara otomatis menghormati pengaturan `prefers-reduced-motion` pengguna. Saat diaktifkan, transisi dinonaktifkan untuk menghindari ketidaknyamanan bagi pengguna yang sensitif terhadap gerakan.

---

## Browser support

View Transitions didukung di Chrome 111+, Edge 111+, dan Safari 18+. Pada browser yang tidak mendukung View Transitions, elemen akan muncul dan hilang tanpa animasi—fungsionalitas tetap berjalan, hanya saja tanpa transisi visual.

> [!warning] Firefox memiliki dukungan terbatas
> Firefox 144+ mendukung transisi tampilan dasar, tetapi tidak mendukung tipe transisi (*transition types*).

---

## See also

* **[wire:show](https://www.google.com/search?q=/docs/4.x/wire-show)** — Ganti visibilitas dengan CSS display
* **[Loading States](https://www.google.com/search?q=/docs/4.x/loading-states)** — Tampilkan indikator pemuatan selama request
* **[Alpine Transitions](https://alpinejs.dev/directives/transition)** — Untuk kebutuhan animasi yang lebih kompleks

---

## Referensi

```blade
wire:transition="name"

```

| Ekspresi | Deskripsi |
| --- | --- |
| (tidak ada) | Menggunakan `match-element` sebagai view-transition-name |
| `"name"` | Menggunakan string yang diberikan sebagai view-transition-name |

Direktif ini tidak memiliki **modifiers**.
