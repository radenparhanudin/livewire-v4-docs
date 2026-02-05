Livewire memungkinkan Anda melakukan *streaming* konten ke halaman web sebelum sebuah *request* selesai melalui API `wire:stream`. Fitur ini sangat berguna untuk hal-hal seperti *AI chat-bot* yang mengirimkan respons segera setelah dihasilkan.

> [!warning] Tidak kompatibel dengan Laravel Octane
> Saat ini Livewire tidak mendukung penggunaan `wire:stream` bersama Laravel Octane.

Untuk mendemonstrasikan fungsionalitas paling dasar dari `wire:stream`, berikut adalah komponen `CountDown` sederhana yang ketika tombol ditekan akan menampilkan hitung mundur kepada pengguna dari "3" ke "0":

```php
use Livewire\Component;

class CountDown extends Component
{
    public $start = 3;

    public function begin()
    {
        while ($this->start >= 0) {
            // Melakukan stream angka saat ini ke browser...
            $this->stream(  // [tl! highlight:4]
                to: 'count',
                content: $this->start,
                replace: true,
            );

            // Jeda selama 1 detik di antara angka...
            sleep(1);

            // Mengurangi penghitung...
            $this->start = $this->start - 1;
        };
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            <button wire:click="begin">Start count-down</button>

            <h1>Count: <span wire:stream="count">{{ $start }}</span></h1> </div>
        HTML;
    }
}

```

Berikut adalah apa yang terjadi dari sudut pandang pengguna saat mereka menekan "Start count-down":

* "Count: 3" ditampilkan di halaman.
* Mereka menekan tombol "Start count-down".
* Satu detik berlalu dan "Count: 2" ditampilkan.
* Proses ini berlanjut hingga "Count: 0" ditampilkan.

Semua hal di atas terjadi hanya dalam satu *network request* yang dikirim ke server.

Berikut adalah apa yang terjadi dari sudut pandang sistem saat tombol ditekan:

* Sebuah *request* dikirim ke Livewire untuk memanggil metode `begin()`.
* Metode `begin()` dipanggil dan perulangan `while` dimulai.
* `$this->stream()` dipanggil dan segera memulai "streamed response" ke browser.
* Browser menerima *streamed response* dengan instruksi untuk mencari elemen dalam komponen dengan `wire:stream="count"`, dan mengganti isinya dengan *payload* yang diterima ("3" dalam kasus angka pertama yang di-*stream*).
* Metode `sleep(1)` menyebabkan server tertahan selama satu detik.
* Perulangan `while` diulang dan proses *streaming* angka baru setiap detik berlanjut hingga kondisi `while` bernilai *falsy*.
* Ketika `begin()` selesai berjalan dan semua hitungan telah di-*stream* ke browser, Livewire menyelesaikan siklus hidup *request*-nya, me-*render* komponen, dan mengirimkan respons akhir ke browser.

---

## Streaming respons chat-bot

Kasus penggunaan umum untuk `wire:stream` adalah melakukan *streaming* respons *chat-bot* saat diterima dari API yang mendukung *streamed responses* (seperti [OpenAI ChatGPT](https://chat.openai.com/)).

Berikut adalah contoh penggunaan `wire:stream` untuk membuat antarmuka seperti ChatGPT:

```php
use Livewire\Component;

class ChatBot extends Component
{
    public $prompt = '';
    public $question = '';
    public $answer = '';

    function submitPrompt()
    {
        $this->question = $this->prompt;
        $this->prompt = '';

        // Memicu metode ask() melalui JavaScript agar input segera bersih
        $this->js('$wire.ask()');
    }

    function ask()
    {
        $this->answer = OpenAI::ask($this->question, function ($partial) {
            $this->stream(to: 'answer', content: $partial); // [tl! highlight]
        });
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            <section>
                <div>ChatBot</div>

                @if ($question)
                    <article>
                        <hgroup>
                            <h3>User</h3>
                            <p>{{ $question }}</p>
                        </hgroup>

                        <hgroup>
                            <h3>ChatBot</h3>
                            <p wire:stream="answer">{{ $answer }}</p> </hgroup>
                    </article>
                @endif
            </section>

            <form wire:submit="submitPrompt">
                <input wire:model="prompt" type="text" placeholder="Send a message" autofocus>
            </form>
        </div>
        HTML;
    }
}

```

Berikut adalah penjelasan proses pada contoh di atas:

* Pengguna mengetikkan pertanyaan pada bidang teks berlabel "Send a message".
* Mereka menekan tombol [Enter].
* Sebuah *network request* dikirim ke server, mengatur pesan ke properti `$question`, dan mengosongkan properti `$prompt`.
* Respons dikirim kembali ke browser dan input dikosongkan. Karena `$this->js('...')` dipanggil, *request* baru dipicu ke server untuk memanggil metode `ask()`.
* Metode `ask()` memanggil API ChatBot dan menerima potongan respons (*partial*) melalui parameter `$partial` di dalam *callback*.
* Setiap `$partial` di-*stream* ke browser ke dalam elemen `wire:stream="answer"` di halaman, menampilkan jawaban yang muncul secara bertahap kepada pengguna.
* Ketika seluruh respons telah diterima, *request* Livewire selesai dan pengguna menerima respons lengkap.

---

## Replace vs. Append

Saat melakukan *streaming* konten ke elemen menggunakan `$this->stream()`, Anda dapat menginstruksikan Livewire untuk mengganti (*replace*) isi elemen target atau menambahkannya (*append*) ke konten yang sudah ada.

Keduanya berguna tergantung pada skenarionya. Misalnya, saat melakukan *streaming* respons dari *chatbot*, biasanya *append* yang diinginkan (dan merupakan default). Namun, saat menampilkan sesuatu seperti hitung mundur, *replace* lebih sesuai.

Anda dapat mengonfigurasi salah satunya dengan memberikan parameter `replace:` ke `$this->stream` dengan nilai boolean:

```php
// Menambahkan konten (Append)...
$this->stream(to: 'target', content: '...');

// Mengganti konten (Replace)...
$this->stream(to: 'target', content: '...', replace: true);

```

Pengaturan *append/replace* juga dapat ditentukan pada level elemen target dengan menambahkan atau menghapus **modifier** `.replace`:

```blade
// Menambahkan konten (Append)...
<div wire:stream="target">

// Mengganti konten (Replace)...
<div wire:stream.replace="target">

```

---

## Referensi

```blade
wire:stream="name"

```

### Modifiers

| Modifier | Deskripsi |
| --- | --- |
| `.replace` | Mengganti isi elemen alih-alih menambahkannya |
