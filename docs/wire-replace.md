*DOM diffing* pada Livewire sangat berguna untuk memperbarui elemen yang ada di halaman Anda, tetapi sesekali Anda mungkin perlu memaksa beberapa elemen untuk di-*render* dari awal guna menyetel ulang (*reset*) *internal state*.

Dalam kasus ini, Anda dapat menggunakan direktif `wire:replace` untuk menginstruksikan Livewire agar melewatkan proses *DOM diffing* pada elemen anak (*children*), dan sebagai gantinya mengganti seluruh konten dengan elemen baru dari server secara total.

Hal ini paling berguna dalam konteks penggunaan pustaka JavaScript pihak ketiga dan *custom web components*, atau ketika penggunaan kembali elemen (*element re-use*) dapat menyebabkan masalah dalam mempertahankan *state*.

Di bawah ini adalah contoh membungkus sebuah *web component* dengan *shadow DOM* menggunakan `wire:replace` sehingga Livewire mengganti elemen tersebut sepenuhnya, memungkinkan *custom element* tersebut menangani siklus hidupnya (*life-cycle*) sendiri:

```blade
<form>
    <div wire:replace>
        <json-viewer>@json($someProperty)</json-viewer>
    </div>

    </form>

```

Anda juga dapat menginstruksikan Livewire untuk mengganti elemen target beserta semua elemen anaknya menggunakan `wire:replace.self`.

```blade
<div x-data="{open: false}" wire:replace.self>
  </div>

```

---

## Referensi

```blade
wire:replace

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.self` | Mengganti elemen itu sendiri dan semua elemen anaknya, bukan hanya elemen anak saja |
