Direktif `@persist` menjaga elemen tetap ada di berbagai navigasi halaman saat menggunakan `wire:navigate`, mempertahankan **state** elemen tersebut dan menghindari inisialisasi ulang.

## Penggunaan dasar

Bungkus sebuah elemen dengan `@persist` dan berikan nama yang unik untuk mempertahankannya di berbagai kunjungan halaman:

```blade
@persist('player')
    <audio src="{{ $episode->file }}" controls></audio>
@endpersist

```

Saat menavigasi ke halaman baru yang juga berisi elemen persisten dengan nama yang sama, Livewire menggunakan kembali elemen DOM yang sudah ada alih-alih membuat yang baru. Untuk pemutar audio, ini berarti pemutaran musik tetap berlanjut tanpa terhenti.

> [!tip] Memerlukan wire:navigate
> Direktif `@persist` hanya berfungsi ketika navigasi ditangani oleh fitur `wire:navigate` milik Livewire. Pemuatan halaman standar (refresh penuh) tidak akan mempertahankan elemen.

---

## Kasus penggunaan umum

**Audio/video players**

```blade
@persist('podcast-player')
    <audio src="{{ $episode->audio_url }}" controls></audio>
@endpersist

```

**Chat widgets**

```blade
@persist('support-chat')
    <div id="chat-widget">
        </div>
@endpersist

```

**Third-party widgets**

```blade
@persist('analytics-widget')
    <div id="analytics-dashboard">
        </div>
@endpersist

```

---

## Penempatan dalam layouts

Elemen yang dipersistenkan biasanya harus ditempatkan di luar komponen Livewire, umumnya di dalam **layout** utama Anda:

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
        <main>
            {{ $slot }}
        </main>

        @persist('player')
            <audio src="{{ $episode->file }}" controls></audio>
        @endpersist

        @livewireScripts
    </body>
</html>

```

---

## Mempertahankan posisi scroll

Untuk elemen persisten yang dapat di-*scroll*, tambahkan `wire:navigate:scroll` untuk menjaga posisi *scroll*:

```blade
@persist('scrollable-list')
    <div class="overflow-y-scroll" wire:navigate:scroll>
        </div>
@endpersist

```

---

## Highlight link aktif

Di dalam elemen yang dipersistenkan, gunakan `wire:current` alih-alih kondisional sisi server untuk menyoroti tautan yang aktif:

```blade
@persist('navigation')
    <nav>
        <a href="/dashboard" wire:navigate wire:current="font-bold">Dashboard</a>
        <a href="/posts" wire:navigate wire:current="font-bold">Posts</a>
        <a href="/users" wire:navigate wire:current="font-bold">Users</a>
    </nav>
@endpersist

```

---

## Cara kerjanya

Saat menavigasi dengan `wire:navigate`:

1. Livewire mencari elemen dengan nama `@persist` yang cocok di kedua halaman.
2. Jika ditemukan, elemen yang sudah ada dipindahkan ke DOM halaman baru.
3. **State**, *event listeners*, dan data Alpine pada elemen tersebut tetap terjaga.

---

## Referensi

```blade
@persist(string $key)
    @endpersist

```

| Parameter | Tipe | Default | Deskripsi |
| --- | --- | --- | --- |
| `$key` | `string` | *required* | Nama unik yang mengidentifikasi elemen agar tetap ada di berbagai navigasi halaman. |
