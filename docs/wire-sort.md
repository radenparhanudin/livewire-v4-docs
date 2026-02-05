Livewire menyediakan kemampuan pengurutan *drag-and-drop* yang kuat melalui direktif `wire:sort` dan `wire:sort:item`. Dengan alat ini, Anda dapat membuat daftar elemen yang dapat diurutkan dengan animasi yang mulus—semuanya ditangani secara otomatis untuk Anda.

## Penggunaan dasar

Untuk membuat daftar dapat diurutkan, tambahkan `wire:sort` pada elemen *parent* dan tentukan nama metode untuk menangani *event* pengurutan. Kemudian, tambahkan `wire:sort:item` pada setiap elemen *child* dengan pengenal unik (*unique identifier*).

Berikut adalah contoh dasar daftar tugas (*todo list*) yang dapat diurutkan:

```php
<?php

use Livewire\Component;
use Livewire\Attributes\Computed;

new class extends Component {
    public Todo $todo;

    public function sortItem($item, $position)
    {
        $item = $this->todo->items()->findOrFail($item);

        // Perbarui posisi item di database dan urutkan kembali item lainnya...
    }
};

```

```blade
<ul wire:sort="sortItem">
    @foreach ($todo->items as $item)
        <li wire:sort:item="{{ $item->id }}">
            {{ $item->title }}
        </li>
    @endforeach
</ul>

```

Ketika pengguna menyeret dan melepas item ke posisi baru, Livewire akan memanggil metode `sortItem` Anda dengan dua parameter: pengenal unik item (dari `wire:sort:item`) dan posisi baru dalam daftar yang berbasis nol (*zero-based*).

Anda bertanggung jawab untuk menyimpan urutan baru tersebut di database Anda. Hal ini biasanya melibatkan pembaruan posisi item yang dipindahkan dan menyesuaikan posisi item lain yang terpengaruh.

---

## Pengurutan antar grup (Sorting across groups)

Jika Anda memiliki beberapa daftar yang dapat diurutkan dalam satu halaman dan ingin mengizinkan pengguna menyeret item di antara daftar tersebut, Anda dapat menggunakan `wire:sort:group` untuk membuat grup bersama.

Dengan menetapkan nama grup yang sama ke beberapa kontainer yang dapat diurutkan, item dapat diseret dari satu daftar ke daftar lainnya:

```php
<?php

use Livewire\Component;
use Livewire\Attributes\Computed;

new class extends Component {
    public User $user;

    public function sortItem($item, $position)
    {
        // Logika untuk menangani pemindahan antar grup...
    }
};

```

```blade
<div>
    @foreach ($user->todoLists as $todo)
        <ul wire:sort="sortItem" wire:sort:group="todos">
            @foreach ($todo->items as $item)
                <li wire:sort:item="{{ $item->id }}">
                    {{ $item->title }}
                </li>
            @endforeach
        </ul>
    @endforeach
</div>

```

Ketika sebuah item diseret ke grup yang berbeda, hanya *handler* dari grup tujuan yang akan menerima *event* pengurutan. *Handler* Anda perlu mendeteksi bahwa item tersebut milik *parent* yang berbeda, mengaitkannya kembali dengan model *parent* yang baru, dan memperbarui posisi urutan untuk item pada *parent* lama maupun baru.

---

## Sort handles

Secara default, pengguna dapat menyeret item dengan mengeklik dan menyeret di mana saja pada elemen yang dapat diurutkan. Namun, Anda dapat membatasi penyeretan pada pegangan (*handle*) tertentu dengan menggunakan `wire:sort:handle`.

Ini berguna ketika Anda memiliki elemen interaktif di dalam item yang dapat diurutkan dan ingin mencegah penyeretan yang tidak disengaja:

```blade
<ul wire:sort="sortItem">
    @foreach ($todo->items as $item)
        <li wire:sort:item="{{ $item->id }}">
            <div wire:sort:handle>
                </div>

            {{ $item->title }}
        </li>
    @endforeach
</ul>

```

Sekarang pengguna hanya dapat menyeret item dengan mengeklik dan menyeret elemen yang ditandai dengan `wire:sort:handle`.

---

## Mengabaikan elemen (Ignoring elements)

Anda dapat mencegah area tertentu dalam item yang dapat diurutkan agar tidak memicu operasi penyeretan dengan menggunakan `wire:sort:ignore`. Ini sangat berguna ketika Anda memiliki tombol atau elemen interaktif lainnya di dalam item tersebut:

```blade
<ul wire:sort="sortItem">
    @foreach ($todo->items as $item)
        <li wire:sort:item="{{ $item->id }}">
            {{ $item->title }}

            <div wire:sort:ignore>
                <button type="button">Edit</button>
            </div>
        </li>
    @endforeach
</ul>

```

Mengeklik dan menyeret di dalam elemen yang ditandai dengan `wire:sort:ignore` tidak akan memberikan efek apa pun, memungkinkan pengguna untuk berinteraksi dengan tombol dan kontrol lainnya tanpa sengaja memicu operasi pengurutan.

---

## Referensi

```blade
wire:sort="method"
wire:sort:item="id"
wire:sort:group="name"
wire:sort:handle
wire:sort:ignore

```

Direktif ini tidak memiliki **modifiers**.
