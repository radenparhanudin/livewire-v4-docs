# CSP (Content Security Policy) Build

Livewire menawarkan build yang aman untuk CSP (*CSP-safe build*), yang memungkinkan Anda menggunakan aplikasi Livewire di lingkungan dengan header Content Security Policy (CSP) ketat yang melarang penggunaan `'unsafe-eval'`.

## Apa itu Content Security Policy (CSP)?

Content Security Policy (CSP) adalah standar keamanan yang membantu mencegah berbagai jenis serangan, termasuk Cross-Site Scripting (XSS) dan serangan injeksi kode. CSP bekerja dengan mengizinkan pengembang web untuk mengontrol sumber daya mana saja yang diizinkan untuk dimuat dan dieksekusi oleh browser.

Salah satu direktif CSP yang paling restriktif adalah `'unsafe-eval'`. Jika direktif ini ditiadakan, browser akan mencegah JavaScript mengeksekusi kode dinamis melalui fungsi seperti `eval()`, `new Function()`, dan konstruksi serupa yang mengompilasi dan mengeksekusi string sebagai kode pada saat *runtime*.

### Mengapa CSP Mempengaruhi Livewire

Secara default, Livewire (dan framework Alpine.js yang menjadi dasarnya) menggunakan deklarasi `new Function()` untuk mengompilasi dan mengeksekusi ekspresi JavaScript dari atribut HTML seperti:

```html
<button wire:click="$set('count', count + 1)">Increment</button>
<div wire:show="user.role === 'admin'">Admin panel</div>

```

Meskipun pendekatan ini jauh lebih cepat dan aman daripada menggunakan `eval()` secara langsung, hal ini tetap melanggar direktif CSP `'unsafe-eval'` yang diterapkan oleh banyak aplikasi yang mengutamakan keamanan.

## Mengaktifkan CSP-Safe Mode

Untuk mengaktifkan mode aman CSP di Livewire, Anda perlu mengubah konfigurasi aplikasi Anda:

### Konfigurasi

Di file `config/livewire.php` Anda, atur opsi `csp_safe` menjadi `true`:

```php
'csp_safe' => true,

```

## Dampak pada Alpine.js

**Penting**: Saat Anda mengaktifkan mode aman CSP di Livewire, hal ini juga memengaruhi semua fungsionalitas Alpine.js di aplikasi Anda. Alpine secara otomatis akan menggunakan *evaluator* khusus CSP, yang berarti semua ekspresi Alpine di seluruh aplikasi Anda akan tunduk pada batasan parsing yang sama.

Di sinilah sebagian besar pengembang akan melihat adanya batasan, karena ekspresi Alpine cenderung lebih kompleks daripada ekspresi Livewire pada umumnya.

---

## Apa yang Didukung

Build CSP mendukung sebagian besar ekspresi JavaScript umum yang biasa Anda gunakan di Livewire:

### Ekspresi Dasar Livewire

```html
<button wire:click="increment">+</button>
<button wire:click="decrement">-</button>
<button wire:click="reset">Reset</button>
<button wire:click="save">Save</button>
<input wire:model="name">
<input wire:model.live="search">

```

### Pemanggilan Metode dengan Parameter

```html
<button wire:click="updateUser('John', 25)">Update User</button>
<button wire:click="setCount(42)">Set Count</button>
<button wire:click="saveData({ name: 'John', age: 30 })">Save Object</button>

```

### Akses dan Pembaruan Properti

```html
<input wire:model="user.name">
<input wire:model="settings.theme">
<button wire:click="$set('user.active', true)">Activate</button>
<div wire:show="user.role === 'admin'">Admin Panel</div>

```

### Ekspresi Dasar di Alpine

```html
<div x-data="{ count: 0, name: 'Livewire' }" wire:ignore>
    <button x-on:click="count++">Increment</button>
    <span x-text="count"></span>
    <span x-text="'Hello ' + name"></span>
    <div x-show="count > 5">Count is high!</div>
</div>

```

---

## Apa yang Tidak Didukung

Beberapa fitur JavaScript tingkat lanjut tidak akan berfungsi dalam mode aman CSP:

### Ekspresi JavaScript Kompleks

```html
<button wire:click="items.filter(i => i.active).length">Count Active</button>
<div wire:show="users.some(u => u.role === 'admin')">Has Admin</div>
<button wire:click="(() => console.log('Hi'))()">Complex Function</button>

```

### Template Literals dan Sintaks Tingkat Lanjut

```html
<div x-text="`Hello ${name}`">Bad</div>
<div x-data="{ ...defaults }">Bad</div>
<button x-on:click="() => doSomething()">Bad</button>

```

### Akses Properti Dinamis

```html
<div wire:show="user[dynamicProperty]">Bad</div>
<button wire:click="this[methodName]()">Bad</button>

```

---

## Mengatasi Batasan

Untuk ekspresi Alpine yang kompleks, gunakan `Alpine.data()` atau pindahkan logika ke dalam metode:

```html
<div x-data="users">
    <div x-show="hasActiveAdmins">Admin panel available</div>
    <span x-text="activeUserCount">0</span>
</div>

<script nonce="[nonce]">
    Alpine.data('users', () => ({
        users: ...,

        get hasActiveAdmins() {
            return this.users.filter(u => u.active && u.role === 'admin').length > 0;
        },

        get activeUserCount() {
            return this.users.filter(u => u.active).length;
        }
    }));
</script>

```

---

## Contoh Header CSP

Berikut adalah contoh header CSP yang dapat berfungsi dengan *CSP-safe build* Livewire:

```
Content-Security-Policy: default-src 'self';
                        script-src 'nonce-[random]' 'strict-dynamic';
                        style-src 'self' 'unsafe-inline';

```

Poin pentingnya:

* Hapus `'unsafe-eval'` dari direktif `script-src` Anda.
* Gunakan pemuatan skrip berbasis nonce dengan `'nonce-[random]'`.
* Pertimbangkan untuk menambahkan `'strict-dynamic'` untuk kompatibilitas yang lebih baik dengan skrip yang dimuat secara dinamis.

---

## Pertimbangan Performa

Build aman CSP menggunakan *expression evaluator* berbeda yang memiliki karakteristik:

* **Parsing**: Proses parsing awal ekspresi sedikit lebih lambat (biasanya tidak terasa).
* **Runtime**: Performa runtime serupa untuk ekspresi sederhana.
* **Ukuran Bundel**: Ukuran bundel JavaScript sedikit lebih besar karena adanya parser kustom.

---

## Kapan Harus Menggunakan CSP-Safe Mode

Gunakan mode aman CSP saat:

* Aplikasi Anda membutuhkan kepatuhan CSP yang ketat.
* Anda membangun aplikasi untuk lingkungan yang sensitif terhadap keamanan.
* Kebijakan keamanan organisasi Anda melarang penggunaan `'unsafe-eval'`.
* Anda melakukan deployment ke platform dengan pembatasan CSP wajib.
