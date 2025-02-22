.. index:: ! event

.. _events:

******
Events
******

Solidity events memberikan abstraksi di atas fungsi logging EVM.
Aplikasi dapat berlangganan dan mendengarkan event ini melalui antarmuka RPC dari klien Ethereum.

Event adalah anggota kontrak yang dapat diwariskan. Saat Anda memanggil mereka, mereka menyebabkan
argumen disimpan di log transaksi - sebuah struktur data khusus di blockchain.
Log ini dikaitkan dengan alamat kontrak, dimasukkan ke dalam blockchain, dan tetap
di sana selama blok dapat diakses (selamanya sampai sekarang, tetapi ini mungkin
berubah dengan Serenity). Log dan data peristiwanya tidak dapat diakses dari dalam
kontrak (bahkan dari kontrak yang membuatnya).

Dimungkinkan untuk meminta bukti Merkle untuk log, jadi jika entitas eksternal
memberikan kontrak dengan bukti seperti itu, ia dapat memeriksa apakah log tersebut
benar-benar ada di dalam blockchain. Anda harus menyediakan header blok karena
kontrak hanya dapat melihat 256 blok hash terakhir.

Anda dapat menambahkan atribut ``indexed`` ke hingga tiga parameter yang menambahkannya
ke struktur data khusus yang dikenal sebagai :ref:`"topics" <abi_events>` alih-alih
bagian data dari log.
Sebuah topik hanya dapat menampung satu kata (32 byte) jadi jika Anda menggunakan
:ref:`reference type <reference-types>` untuk argumen yang diindeks, hash nilai
Keccak-256 disimpan sebagai topik.

Semua parameter tanpa atribut ``indexed`` adalah :ref:`ABI-encoded <ABI>`
ke dalam bagian data log.

Topik memungkinkan Anda mencari event, misalnya saat memfilter urutan blok untuk
event tertentu. Anda juga dapat memfilter event berdasarkan alamat kontrak yang
mengeluarkan event tersebut.

Misalnya, kode di bawah ini menggunakan metode ``subscribe("logs")``
`web3.js <https://web3js.readthedocs.io/en/1.0/web3-eth-subscribe.html#subscribe-logs> `_ untuk menyaring
log yang cocok dengan topik dengan nilai alamat tertentu:

.. code-block:: javascript

    var options = {
        fromBlock: 0,
        address: web3.eth.defaultAccount,
        topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
    };
    web3.eth.subscribe('logs', options, function (error, result) {
        if (!error)
            console.log(result);
    })
        .on("data", function (log) {
            console.log(log);
        })
        .on("changed", function (log) {
    });


Hash dari signature event adalah salah satu topik, kecuali jika Anda mendeklarasikan event dengan specifier ``anonymous``.
Ini berarti bahwa tidak mungkin untuk memfilter peristiwa anonim tertentu berdasarkan nama, Anda hanya dapat memfilter menurut alamat kontrak.
Keuntungan dari peristiwa anonim adalah bahwa mereka lebih murah untuk digunakan dan dipanggil.
Ini juga memungkinkan Anda untuk mendeklarasikan empat argumen yang diindeks daripada tiga.

.. note::
    Karena log transaksi hanya menyimpan event data dan bukan jenisnya, Anda harus mengetahui
    jenis event, termasuk parameter mana yang diindeks dan apakah event itu anonim untuk menafsirkan
    data dengan benar.
    Secara khusus, dimungkinkan untuk "memalsukan" tanda tangan event lain menggunakan event anonim.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.21 <0.9.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // Events are emitted using `emit`, followed by
            // the name of the event and the arguments
            // (if any) in parentheses. Any such invocation
            // (even deeply nested) can be detected from
            // the JavaScript API by filtering for `Deposit`.
            emit Deposit(msg.sender, _id, msg.value);
        }
    }

Penggunaan dalam JavaScript API adalah sebagai berikut:

.. code-block:: javascript

    var abi = /* abi as generated by the compiler */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

    var depositEvent = clientReceipt.Deposit();

    // watch for changes
    depositEvent.watch(function(error, result){
        // result contains non-indexed arguments and topics
        // given to the `Deposit` call.
        if (!error)
            console.log(result);
    });


    // Or pass a callback to start watching immediately
    var depositEvent = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

Output di atas terlihat seperti berikut (trimmed):

.. code-block:: json

    {
       "returnValues": {
           "_from": "0x1111…FFFFCCCC",
           "_id": "0x50…sd5adb20",
           "_value": "0x420042"
       },
       "raw": {
           "data": "0x7f…91385",
           "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
       }
    }

Sumber Daya Tambahan untuk Memahami Event
=========================================

- `Dokumentasi Javascript <https://github.com/ethereum/web3.js/blob/1.x/docs/web3-eth-contract.rst#events>`_
- `Contoh penggunaan events <https://github.com/ethchange/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `Cara mengaksesnya di js <https://github.com/ethchange/smart-exchange/blob/master/lib/exchange_transactions.js>`_
