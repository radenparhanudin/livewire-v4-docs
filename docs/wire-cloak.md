`wire:cloak` adalah direktif yang menyembunyikan elemen saat halaman dimuat sampai Livewire sepenuhnya terinisialisasi. Ini sangat berguna untuk mencegah "flash of unstyled content" (kilatan konten yang belum terformat) yang dapat terjadi ketika halaman muncul sebelum Livewire sempat berjalan.

## Penggunaan dasar

Untuk menggunakan `wire:cloak`, tambahkan direktif ini ke elemen apa pun yang ingin Anda sembunyikan selama pemuatan halaman:

```blade
<div wire:cloak>
    Konten ini akan disembunyikan sampai Livewire dimuat sepenuhnya
</div>

```

Agar direktif ini berfungsi, Anda harus menambahkan gaya CSS berikut ke dalam *stylesheet* atau di dalam tag `<style>` pada *layout* aplikasi Anda:

```css
[wire\:cloak] {
    display: none !important;
}

```

### Konten dinamis

`wire:cloak` sangat berguna dalam skenario di mana Anda ingin mencegah pengguna melihat konten dinamis yang belum terinisialisasi, seperti elemen yang ditampilkan atau disembunyikan menggunakan `wire:show`.

```blade
<div>
    <div wire:show="starred" wire:cloak>
        </div>

    <div wire:show="!starred" wire:cloak>
        </div>
</div>

```

Pada contoh di atas, tanpa `wire:cloak`, kedua ikon akan muncul secara bersamaan sebelum Livewire terinisialisasi. Namun, dengan `wire:cloak`, kedua elemen akan disembunyikan sampai proses inisialisasi selesai.

---

## Referensi

```blade
wire:cloak

```

Direktif ini tidak memiliki **modifiers**.
