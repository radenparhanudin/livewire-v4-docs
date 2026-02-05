Fitur **file downloads** di Livewire bekerja hampir sama dengan Laravel itu sendiri. Biasanya, Anda dapat menggunakan utilitas download Laravel apa pun di dalam **component** Livewire, dan itu akan berfungsi seperti yang diharapkan.

Namun, di balik layar, proses unduh file ditangani secara berbeda daripada aplikasi Laravel standar. Saat menggunakan Livewire, konten file dienkripsi menjadi **Base64**, dikirim ke *frontend*, dan didekode kembali menjadi biner untuk diunduh langsung dari sisi klien.

## Penggunaan dasar

Memicu pengunduhan file di Livewire semudah mengembalikan respons unduhan (*download response*) Laravel standar.

Di bawah ini adalah contoh **component** `show-invoice` yang berisi tombol "Download" untuk mengunduh PDF faktur:

```php
<?php // resources/views/components/⚡show-invoice.blade.php

use Livewire\Component;
use App\Models\Invoice;

new class extends Component {
    public Invoice $invoice;

    public function mount(Invoice $invoice)
    {
        $this->invoice = $invoice;
    }

    public function download()
    {
        return response()->download( // [tl! highlight:2]
            $this->invoice->file_path, 'invoice.pdf'
        );
    }
};

```

```blade
<div>
    <h1>{{ $invoice->title }}</h1>

    <span>{{ $invoice->date }}</span>
    <span>{{ $invoice->amount }}</span>

    <button type="button" wire:click="download">Download</button> </div>

```

Sama seperti di dalam **controller** Laravel, Anda juga dapat menggunakan **facade** `Storage` untuk memulai unduhan:

```php
public function download()
{
    return Storage::disk('invoices')->download('invoice.csv');
}

```

---

## Streaming downloads

Livewire juga dapat melakukan **stream downloads**; namun, proses ini tidak benar-benar di-*stream*. Unduhan tidak akan dipicu sampai seluruh konten file dikumpulkan dan dikirimkan ke browser:

```php
public function download()
{
    return response()->streamDownload(function () {
        echo '...'; // Menampilkan konten unduhan secara langsung...
    }, 'invoice.pdf');
}

```

---

## Testing file downloads

Livewire juga menyediakan metode `->assertFileDownloaded()` untuk menguji dengan mudah bahwa sebuah file telah diunduh dengan nama yang diberikan:

```php
use App\Models\Invoice;

public function test_can_download_invoice()
{
    $invoice = Invoice::factory();

    Livewire::test(ShowInvoice::class)
        ->call('download')
        ->assertFileDownloaded('invoice.pdf');
}

```

Anda juga dapat menguji untuk memastikan bahwa tidak ada file yang diunduh menggunakan metode `->assertNoFileDownloaded()`:

```php
use App\Models\Invoice;

public function test_does_not_download_invoice_if_unauthorised()
{
    $invoice = Invoice::factory();

    Livewire::test(ShowInvoice::class)
        ->call('download')
        ->assertNoFileDownloaded();
}

```
