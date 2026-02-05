Kemampuan Livewire untuk melakukan pembaruan pada halaman adalah hal yang membuatnya terasa "hidup". Namun, ada kalanya Anda mungkin ingin mencegah Livewire memperbarui bagian tertentu dari halaman tersebut.

Dalam kasus ini, Anda dapat menggunakan direktif `wire:ignore` untuk menginstruksikan Livewire agar mengabaikan konten dari elemen tertentu, meskipun konten tersebut berubah di antara *requests*.

Hal ini paling berguna dalam konteks penggunaan pustaka JavaScript pihak ketiga (*third-party libraries*) untuk input formulir kustom dan sejenisnya.

Di bawah ini adalah contoh membungkus elemen yang digunakan oleh pustaka pihak ketiga dengan `wire:ignore` sehingga Livewire tidak mengganggu HTML yang dihasilkan oleh pustaka tersebut:

```blade
<form>
    <div wire:ignore>
        <input id="id-for-date-picker-library">
    </div>

    </form>

```

Anda juga dapat menginstruksikan Livewire untuk hanya mengabaikan perubahan pada atribut elemen *root* itu sendiri, daripada mengabaikan seluruh isinya, dengan menggunakan `wire:ignore.self`.

```blade
<div wire:ignore.self>
    </div>

```

---

## Referensi

```blade
wire:ignore

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.self` | Hanya mengabaikan perubahan atribut pada elemen itu sendiri, bukan elemen anak (*children*) di dalamnya |
