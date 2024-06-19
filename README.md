## 1.	Algoritma berfikir
Cara mengantisipasi dan menanggulangi kemungkinan banyaknya barang kadaluarsa di gudang :
- Dalam hal penataan barang, barang yang sudah lama atau yang lebih dulu masuk harus ditempatkan di depan agar barang tersebut terjual lebih dahulu.
- Agendakan pemeriksaan barang secara rutin untuk memastikan tidak ada barang yang kadaluarsa atau yang sudah mendekati tanggal kadaluarsa. Dengan pemeriksaan rutin, braang tak hanya di cek tanggal kadaluarsanya tapi juga stok dari barang tersebut, hal ini memungkinkan penyetokan ulang barang yang akan mau habis.
- Menempatkan barang sesuai dengan syarat penyimpanan, missal ada beberapa yang memerlukan suhu tertentu untuk menjaga kualitas berserta masa simpan.
- Menggunakan program inventaris yang dapat mencatatn dan melacak tanggal kadaluarsa barang. Dengan adanya program, pegawai dapat dengan mudah melacak barang yang sudah mendekati masa kadaluarsa.


## 2.	Penguasaan query

a.	Query create table gudang dan barang dengan foreign key dan index. Asumsi field atau kolom mengikuti soal 2 b.
- Create table gudang

``CREATE TABLE gudang(
	kode_gudang VARCHAR(100) PRIMARY KEY,
	nama_gudang VARCHAR(100) NOT NULL,
);``

- Create table barang

``CREATE TABLE barang ( 
kode_barang VARCHAR(100) PRIMARY KEY, 
nama_barang VARCHAR(100) NOT NULL,
jumlah_barang INT NOT NULL, 
expired_barang DATETIME NOT NULL,
kode_gudang VARCHAR(100), 
INDEX fk_gudang (kode_gudang), FOREIGN KEY (kode_gudang) REFERENCES gudang(kode_gudang) 
);``

b.	Query store procedure yang menampilkan kode gudang, nama gudang, kode barang, nama barang, harga barang, jumlah barang, expired barang dengan dynamic query dan paging

``CREATE PROCEDURE GetPageInventory(
IN page INT,
IN pageSize INT )
BEGIN 
DECLARE offset INT; 
SET offset = (page - 1) * pageSize; 
SET @sql = CONCAT(
' SELECT g.kode_gudang AS Kode Gudang, g.nama_gudang AS Nama Gudang, b.kode_barang AS Kode Barang, b.nama_barang AS Nama Barang, b.harga_barang AS Harga Barang, b.jumlah_barang AS Jumlah Barang, b.expired_barang AS Expired Barang FROM gudang g JOIN barang b ON g.kode_gudang = b.kode_gudang LIMIT ', offset, ', ', pageSize); 
PREPARE stmt FROM @sql; 
EXECUTE stmt; 
DEALLOCATE PREPARE stmt; 
END``

c.	Query trigger input barang di salah satu Gudang yang memunculkan barang kadaluarsa

``CREATE TRIGGER triger_insert 
AFTER INSERT ON Barang 
FOR EACH ROW 
BEGIN DECLARE currentDate DATE; 
SET currentDate = CURDATE(); 
SELECT b.kode_barang AS Kode Barang, b.nama_barang AS Nama Barang, b.harga_barang AS Harga Barang, b.jumlah_barang AS Jumlah Barang, b.expired_barang AS Expired Barang 
FROM barang b WHERE b.kode_gudang = NEW.kode_gudang AND b.expired_barang < currentDate;
END``

