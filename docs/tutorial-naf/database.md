---
siebar_position: 2
---

# Database

Misal apabila hendak membuat storage untuk tabel USER, kita susun struktur modelnya seperti ini:

Dan untuk model `Mysql.php` bisa menurunkan/extend dari model `Neuron\Db\Storage.php` seperti ini.

```js
class Mysql extends \Neuron\Db\Storage
{

}
```

Setelah diturunkan maka model `Mysql.php` akan memiliki metode dan atribute yang bisa digunakan untuk mengakses tabel, seperti operasi select, insert, update dan delete. Sedangkan untuk model Oci8.php harus extend dari Mysql.php.

## Select

Gunakan metode select() untuk melakukan query.

```js
$select = $this->select();
$select->from($this->_tables['users']);
$select->where($this->__(array('user_id' => $id)));
return $this->fetchRow($select);
```

Kode diatas akan menampilkan satu row data dari tabel USER sesuai dengan USER_ID yang diberikan.

```js
$select = $this->select();
$select->from($this->_tables['users']);
return $this->fetchAll($select);
```

Kode diatas akan menampilkan semua record yang ada di tabel USER. Untuk record yang banyak pastikan hasil query dibatasi dengan `limit()`.

Untuk contoh select yang lebih banyak bisa dilihat di sini.

## Insert

Gunakan metode `insert()` untuk memasukkan record baru ke tabel.

```js
$insert = $this->insert();
$insert->into($this->_tables['users']);
$insert->autoincrement($this->_('user_id'));
$insert->values($this->__($user->pull()));
if($id = $this->execute($insert))
{
return $id;
}
return false;
```

Kode diatas akan memasukkan record baru sesuai dengan field-field yang ada di model User.php (balikan dari metode `pull()`). `autoincrement()` menentukan field mana di tabel yang akan diisi dengan sequence secara otomatis. Untuk MySQL ini ditentukan dari opsi AUTO INCREMENT saat DDL tabel. Untuk Oracle kita harus definisikan sequence sesuai nama tabelnya. Misal untuk keperluan auto increment di tabel USER, buat sequence dengan nama SEQ_USER, SEQ_JOB untuk tabel JOB, dst.

## Update

Gunakan metode `update()` untuk update data tabel.

```js
$update = $this->update();
$update->table($this->_tables['users']);
$update->set($this->__($user->pull()));
$update->where($this->\_\_(array('user_id' => $user->id)));
if($this->execute($update))
{
return $user->id;
}
return false;
```

Kode diatas akan update record di tabel sesuai data yang didapatkan dari balikan metode `pull()`.

## Delete

Gunakan metode `delete()` untuk menghapus record dari table. Pastikan kondisi where sudah sesuai untuk menghindari salah hapus record.

```js
$delete = $this->delete();
$delete->from($this->_tables['users']);
$delete->where($this->__(array('user_id' => $user->id)));
return $this->execute($delete);
```

Kode diatas akan menghapus record dari table sesuai kondisi where yang diterapkan.

Reference
Untuk referensi tambahan bisa melihat dokumentasi Zend Framework 2.

https://framework.zend.com/manual/2.4/en/modules/zend.db.sql.html
https://framework.zend.com/apidoc/2.4/classes/Zend.Db.Sql.Select.html

## Pager

Minimun Version: v1.5.7
Pager `(\Neuron\Db\Storage\Pager)` digunakan untuk mempermudah proses sanitasi input dan generate result output hasil query yang menggunakan limit dan offset.
Dengan menggunakan pager, developer tidak perlu memikirkan validasi data yang dikirim dari front end dalam generate query untuk paging.

## Sanitize

Fungsi `sanitize()` pada Pager digunakan untuk menerima input parameter paging dan melakukan validasi dan input tersebut sehingga menghasilkan nilai yang valid dan dapat langsung digunakan untuk menambahkan limit dan offset pada query.
Fungsi ini menerima inputan array dimana key array page adalah nomor halaman yang akan ditampilkan, dan key array records adalah jumlah record yang akan ditampilkan.
Contoh penggunaan `sanitize()` adalah:

```js
// Pager dengan input wajar
$pager = \Neuron\Db\Storage\Pager::sanitize(array('page' => 8, 'records' => 10));

echo($pager->getLimit());  //akan menghasilkan output: 10 (jumlah record per halaman)
echo($pager->getOffset()); //akan menghasilkan output: 70 (start offset record yang tampil)
echo($pager->getPage()); //akan menghasilkan output: 8 (halaman ke-8)

// Pager dengan input halaman non numerik
$pager = \Neuron\Db\Storage\Pager::sanitize(array('page' => 'abc', 'records' => 20));

echo($pager->getLimit());  //akan menghasilkan output: 20 (jumlah record per halaman)
echo($pager->getOffset()); //akan menghasilkan output: 0 (start offset record yang tampil)
echo($pager->getPage()); //akan menghasilkan output: 1 (halaman ke-1)

// Pager dengan kedua input tidak valid
$pager = \Neuron\Db\Storage\Pager::sanitize(array('page' => 'xyz', 'records' => 'ijk'));

echo($pager->getLimit());  //akan menghasilkan output: 999999 (jumlah record per halaman)
echo($pager->getOffset()); //akan menghasilkan output: 0 (start offset record yang tampil)
echo($pager->getPage()); //akan menghasilkan output: 1 (halaman ke-1)

// Pager dengan input negatif
$pager = \Neuron\Db\Storage\Pager::sanitize(array('page' => -10, 'records' => 40));

echo($pager->getLimit());  //akan menghasilkan output: 40 (jumlah record per halaman)
echo($pager->getOffset()); //akan menghasilkan output: 0 (start offset record yang tampil)
echo($pager->getPage()); //akan menghasilkan output: 1 (halaman ke-1)
```

## Factory

Selain `sanitize()`, Pager juga memiliki method factory untuk menghasilkan instance obyek Pager itu sendiri. Method ini memiliki parameter $page yaitu nomor halaman, dan $records yaitu jumlah record per halaman.
Contoh penggunaan `factory()` adalah:

```js
$pager = \Neuron\Db\Storage\Pager::factory(5, 15);

echo($pager->getLimit());  //akan menghasilkan output: 15 (jumlah record per halaman)
echo($pager->getOffset()); //akan menghasilkan output: 60 (start offset record yang tampil)
echo($pager->getPage()); //akan menghasilkan output: 5 (halaman ke-5)

Penggunaan Pada Select
Karena Pager hanya berfungsi sebagai sanitasi input, kita harus menambahkan secara manual limit dan offset dari obyek Pager ke statement select yang ingin kita set pagingnya.
Contoh aplikasi Pager pada statement select adalah:

// Asumsi input dari front end / ajax
$input = array('page' => 2, 'records' => 10); //halaman ke-2, 10 record per halaman

// Buat pager
$pager = \Neuron\Db\Storage\Pager::sanitize($input);

// Generate select dari tabel user dan limit menggunakan pager
$select = $this->select()
   ->from('users')
   ->limit($pager->getLimit())
->offset($pager->getOffset());

// Execute select
return $this->fetchAll($select);
```

## Result

Obyek `Neuron\Db\Result` dapat menampung hasil query disertai dengan data-data tambahan berupa total keseluruhan record di sebuah tabel dan data dari obyek Pager.
Obyek Result dapat dihasilkan dari obyek Pager menggunakan fungsi `result()`.
Contoh penggunaan obyek Result adalah sebagai berikut:

```js
// Asumsi input dari front end / ajax
$input = array('page' => 1, 'records' => 10); //halaman ke-1, 10 record per halaman

// Buat pager
$pager = \Neuron\Db\Storage\Pager::sanitize($input);

// Generate select dari tabel user dan limit menggunakan pager
$select = $this->select()
   ->from('users')
   ->limit($pager->getLimit())
->offset($pager->getOffset());

// Execute select untuk menghasilkan 10 record (karena di limit)
$rows = $this->fetchAll($select);

// Generate result menggunakan obyek pager
$result = $pager->result($rows);

// Pemanfaatan obyek result, output data halaman
echo($result->pager->getLimit());  //akan menghasilkan output: 10 (jumlah record per halaman)
echo($result->pager->getOffset()); //akan menghasilkan output: 0 (start offset record yang tampil)
echo($result->pager->getPage()); //akan menghasilkan output: 1 (halaman ke-1)

// Pemanfaatan obyek result, output total record di tabel
echo($result->total);       //akan menghasilkan jumlah total record di tabel user
echo($result->countTotal); //sama dengan diatas, disediakan untuk backward compatibility

// Pemanfaatan obyek result, loop record hasil query
foreach ($result as $record) {
   print_r($record); //print data record ke-n
}

// Sama dengan loop diatas, disediakan untuk backward compatibility
foreach ($result->list as $record) {
   print_r($record);
}
```

### Catatan

Dengan adanya obyek Result, maka metode lama untuk menghasilkan data-data tambahan paging dianggap obsolete.
Contoh metode lama:

```js
// Execute select
$rows = $this->fetchAll($select);

// Output result select dengan data paging tambahan (metode lama -- obsolete)
$result = new \stdClass();
$result->list = $rows;
$result->pager = $pager;
$result->countTotal = $rows->total;

return $result;
```

## Filter dan Sort

Tersedia sejak: v1.5.8
Fungsi filtering dan sorting dinamis di sediakan untuk mempermudah pembuatan filter dari front end. Fungsi ini memungkinkan penyusunan kondisi dan ordering select dengan menggunakan parameter berformat Array yang mudah di buat dan di kirim dari sisi front end.
Tujuan dari fitur ini adalah agar front end dapat secara mudah membuat parameter filter dalam bentuk JSON / Array, lalu mengirimkan parameter tersebut menggunakan REST API, tanpa harus melakukan generate `select->where()` sendiri dari sisi kode.

### Penggunaan

Untuk menggunakan, di sediakan fungsi filter dan sort yang dapat di panggil dari obyek sql Select dari dalam model storage:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([array])
    ->sort([array]);

```

### Filter

### Filter Sederhana

Filter sederhana paling umum adalah filtering dengan kondisi AND, contoh select:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 = 'A') AND (kolom_2 = 'B');
```

Dapat di representasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
        'kolom_2' => 'B',
    ]);

```

Ini sama saja dengan penggunaan where berikut:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->where([
        'kolom_1' => 'A',
        'kolom_2' => 'B',
    ]);

```

Anda juga bisa membuat filter menggunakan simbol & (ampersand) untuk merepresentasikan AND sebagai key array, contoh di bawah ini akan menghasilkan statement select yang sama:

```php
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
        '&' => [
            'kolom_2' => 'B',
        ]
    ]);
```

### Filter Dengan OR

Penggunaan kondisi OR dapat secara mudah dilakukan menggunakan simbol | untuk merepresentasikan OR, sebagai contoh statement SQL berikut:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 = 'A') OR (kolom_2 = 'B');
```

Dapat di representasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
        '|' => [
            'kolom_2' => 'B',
        ]
    ]);
```

Perhatikan bahwa untuk membuat kondisi OR digunakan simbol `|` (vertical bar) sebagai key array.

### Filter Kompleks / Nested

Filtering yang lebih kompleks pun dapat dilakukan secara mudah menggunakan kombinasi simbol dan array, sebagai contoh statement:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 = 'A') OR ((kolom_2 = 'B') AND (kolom_3 = 99));
```

Dapat direpresentasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
        '|' => [
            'kolom_2' => 'B',
            'kolom_3' => 99,
        ]
    ]);
```

Contoh lainnya adalah statement SQL berikut:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 = 'A') OR (kolom_2 = 'B') OR ((kolom_3 = 99 AND kolom_4 = 'X'));
```

Dapat direpresentasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
        '|' => [
            'kolom_2' => 'B',
        ],
        '|' => [
            'kolom_3' => 99,
            'kolom_4' => 'X',
        ],
    ]);
```

## Modifier

Modifier digunakan jika ingin membuat statement yang menggunakan operator lain selain =, sebagai contoh SQL statement berikut:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 <> 'A');
```

Dapat di representasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => '!A',
    ]);
```

Perhatikan penggunaan modifier ! sebelum value A untuk membuat operator `<>` (tidak sama dengan) pada saat generate query.
Modifier adalah 1 karakter spesial sebelum value.

Modifier yang tersedia antara lain:

```
!: Tidak sama dengan

<: Lebih kecil

>: Lebih besar

-: Lebih kecil atau sama dengan

+: Lebih besar atau sama dengan

~: Like

#: Not like
```

Jadi statement select berikut:

```sql
SELECT * FROM nama_tabel WHERE (kolom_1 LIKE 'A%') OR ((kolom_2 < 99) AND (kolom_3 >= 77));
```

Dapat di representasikan dalam kode sbb:

````js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => '~A%',
        '|' => [
            'kolom_2' => '<99',
            'kolom_3' => '+77',
        ],
    ]);
    ```
````

## Alias

Penggunaan alias nama tabel pun menjadi lebih mudah, sebagai contoh statement berikut ini:

```sql
SELECT t.* FROM nama_tabel AS t WHERE (t.kolom_1 = 'A') AND (t.kolom_2 <> 'C');
```

Dapat di representasikan dalam kode sbb:

```js
$select = $this->select()
    ->from(['t' => 'nama_tabel'])
    ->filter([
        't.' => [
            'kolom_1' => 'A',
            'kolom_2' => '!C',
        ],
    ]);
```

Perhatikan bahwa jika key array diakhiri dengan karakter . (titik), dianggap sebagai alias tabel.
Contoh lain yang lebih kompleks adalah untuk statement SQL berikut:

```sql
SELECT t.*, u.kolom_lainnya FROM nama_tabel AS t
INNER JOIN tabel_lainnya AS u ON t.id_lainnya = u.id_lainnya
WHERE (t.kolom_1 = 'A') OR ((u.kolom_2 > 10) AND (u.kolom_2 < 100));
```

Dapat di representasikan sbb:

```js
$select = $this->select()
    ->from(['t' => 'nama_tabel'])
    ->join(
        ['u' => 'tabel_lainnya'],
        t.id_lainnya = u.id_lainnya,
        [kolom_lainnya]
    )
    ->filter([
        't.' => [
            'kolom_1' => 'A',
        ],
        '|' => [
            'u.' => [
                'kolom_2' => '>10',
                'kolom_2' => '<100',
            ],
        ],
    ]);
```

## Sort

Pengurutan pada Select
Pengurutan dari statement select dapat secara mudah dilakukan menggunakan fungsi sort, sebagai contoh statement berikut:

```sql
SELECT  FROM nama_tabel WHERE (kolom_1 = 'A') ORDER BY kolom_2 ASC, kolom_3 DESC;
```

Dapat di representasikan dalam kode sbb:

```js
$select = $this->select()
    ->from('nama_tabel')
    ->filter([
        'kolom_1' => 'A',
    ])
    ->sort([
        'kolom_2' => 'asc',
        'kolom_3' => 'desc',
    ]);
```

Pengurutan Menggunakan Alias
Seperti pada filter, alias juga dapat digunakan pada sort contohnya pada statement berikut:

```sql
SELECT t.*, u.kolom_lainnya FROM nama_tabel AS t
INNER JOIN tabel_lainnya AS u ON t.id_lainnya = u.id_lainnya
WHERE (t.kolom_1 = 'A')
ORDER BY t.kolom_1 ASC, t.kolom_2 DESC, u.kolom_lainnya DESC;
```

Dapat di representasikan sbb:

```js
$select = $this->select()
    ->from(['t' => 'nama_tabel'])
    ->join(
        ['u' => 'tabel_lainnya'],
        t.id_lainnya = u.id_lainnya,
        [kolom_lainnya]
    )
    ->filter([
        't.' => [
            'kolom_1' => 'A',
        ],
    ])
    ->sort([
        't.' => [
            'kolom_1' => 'asc',
            'kolom_2' => 'desc',
        ],
        'u.' => [
            'kolom_lainnya' => 'desc',
        ],
    ]);
```

### Filter pada Update

Selain pada statement Select, filter juga dapat digunakan pada statement Update.
Sebagai contoh statement update berikut:

```sql
UPDATE nama_tabel SET kolom_4 = 'test' WHERE (kolom_1 = 'A') OR ((kolom_2 = 'B') AND (kolom_3 = 99));
```

Dapat direpresentasikan dalam kode sbb:

````js
$update = $this->update()
    ->table('nama_tabel')
    ->set(['kolom_4' => 'test'])
    ->filter([
        'kolom_1' => 'A',
        '|' => [
            'kolom_2' => 'B',
            'kolom_3' => 99,
        ]
    ]);
    ```
````
