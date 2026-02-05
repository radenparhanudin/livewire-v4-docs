**Refs** di Livewire menyediakan cara untuk memberi nama, kemudian menargetkan elemen atau *component* individu di dalam Livewire.

Ini sangat berguna untuk mengirimkan *events* atau melakukan *streaming content* ke elemen tertentu. Konsepnya mirip dengan penggunaan *class* atau *id* untuk menargetkan elemen, namun jauh lebih rapi dan terintegrasi dengan ekosistem Livewire.

Berikut adalah daftar kasus penggunaannya:

* Mengirimkan (*dispatching*) *event* ke *component* tertentu.
* Menargetkan elemen menggunakan `$refs`.
* Melakukan *streaming content* ke elemen tertentu.

Mari kita pelajari masing-masing kegunaan tersebut.

---

## Mengirimkan events (Dispatching events)

**Refs** adalah cara yang sangat baik untuk menargetkan *child components* tertentu dalam sistem *event* Livewire.

Pertimbangkan *component* modal Livewire berikut yang mendengarkan *event* `close`:

```php
<?php

new class extends Livewire\Component {
    public bool $isOpen = false;

    // ...

    #[On('close')]
    public function close()
    {
        $this->isOpen = false;
    }
};
?>

<div wire:show="isOpen">
    {{ $slot }}
</div>

```

Dengan menambahkan `wire:ref` pada tag *component*, Anda sekarang dapat mengirimkan *event* `close` secara langsung kepadanya menggunakan parameter `ref:`:

```php
<?php

new class extends Livewire\Component {
    public function save()
    {
        // ...

        $this->dispatch('close')->to(ref: 'modal');
    }
};
?>

<div>
    <livewire:modal wire:ref="modal">
        <button wire:click="save">Save</button>
    </livewire:modal>
</div>

```

---

## Mengakses elemen DOM

Ketika Anda menambahkan `wire:ref` pada elemen HTML, Anda dapat mengaksesnya melalui properti *magic* `$refs`.

Pertimbangkan penghitung karakter yang diperbarui secara *real-time*:

```php
<div>
    <textarea wire:model="message" wire:ref="message"></textarea>

    Characters: <span wire:ref="count">0</span>

    </div>

<script>
    this.$refs.message.addEventListener('input', (e) => {
        this.$refs.count.textContent = e.target.value.length
    })
</script>

```

---

## Mengakses $wire

Jika Anda ingin mengakses properti `$wire` dari sebuah *component* yang memiliki **ref**, Anda dapat melakukannya melalui properti `.$wire` pada elemen tersebut:

```php
<script>
    this.$intercept('save', ({ onFinish }) => {
        onFinish(() => {
            // Mengakses objek $wire milik component modal melalui ref
            this.$refs.modal.$wire.close()
        })
    })
</script>

```

---

## Streaming content

Livewire mendukung *streaming content* langsung ke elemen-elemen di dalam sebuah *component* menggunakan selektor CSS. Namun, `wire:ref` adalah pendekatan yang lebih nyaman dan mudah ditemukan (*discoverable*).

Pertimbangkan *component* berikut yang melakukan *stream* jawaban langsung dari sebuah LLM saat jawaban tersebut dihasilkan:

```php
<?php

new class extends Livewire\Component {
    public $question = '';

    public function ask()
    {
        Ai::ask($this->question, function ($chunk) {
            // Mengirimkan potongan data (chunk) ke elemen dengan ref 'answer'
            $this->stream($chunk)->to(ref: 'answer');
        });

        $this->reset('question');
    }
};
?>

<div>
    <input type="text" wire:model="question">

    <button wire:click="ask">Ask AI</button>

    <h2>Answer:</h2>

    <p wire:ref="answer"></p>
</div>

```

---

## Dynamic refs

**Refs** bekerja dengan sempurna di dalam perulangan (*loops*) dan konteks dinamis lainnya. Berikut adalah contoh dengan beberapa instansi modal:

```blade
@foreach($users as $user)
    <livewire:modal
        wire:key="{{ $user->id }}"
        wire:ref="{{ 'user-modal-' . $user->id }}"
    >
        </livewire:modal>
@endforeach

```

---

## Perilaku Scoping

**Refs** dibatasi (*scoped*) pada *component* saat ini. Ini berarti Anda dapat menargetkan elemen apa pun di dalam *component* tersebut, tetapi tidak dapat menargetkan elemen di *component* lain pada halaman yang sama.

Jika beberapa elemen memiliki nama **ref** yang sama di dalam satu *component*, maka elemen pertama yang ditemukanlah yang akan digunakan.

---

## Referensi

```blade
wire:ref="name"

```

Direktif ini tidak memiliki **modifiers**.
