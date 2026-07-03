# UAS_TeknikKompilasi
# Natan Nael
# Nim 231011400113

Penjelasan Tahapan Kompilasi

A. Analisis Leksikal (Lexer)
Pada tahap ini, kode sumber string dipecah menjadi unit terkecil yang disebut Token. Program menggunakan Regular Expression untuk mengidentifikasi apakah sebuah kata termasuk kata kunci (while), operator (<, +), identifier (counter), atau simbol ({, }).
Hasil: Kumpulan token seperti [WHILE: while], [LPAREN: (], dst.


B. Analisis Sintaksis (Parser)
Tahap ini memeriksa apakah urutan token sesuai dengan aturan tata bahasa (Grammar) yang telah didefinisikan dalam BNF. Program menggunakan Recursive Descent Parsing sederhana untuk membangun Abstract Syntax Tree (AST).
Struktur AST: WhileNode memiliki anak berupa BinaryOpNode (untuk kondisi) dan AssignmentNode (untuk isi perulangan).


C. Analisis Semantik
Tahap ini memeriksa makna dari kode tersebut. Dalam implementasi ini, dilakukan pengecekan Variabel Scope/Declaration. Program memeriksa apakah variabel counter dan sum sudah ada dalam symbol_table. Jika kita menggunakan variabel yang tidak terdaftar, compiler akan memberikan error meskipun secara struktur (sintaksis) sudah benar.


D. Generasi Kode Antara (Three-Address Code)
Tahap akhir adalah mengubah AST menjadi format kode antara yang lebih dekat dengan bahasa mesin namun masih terbaca manusia. TAC menggunakan maksimal tiga alamat (dua operan dan satu hasil).
Logika TAC While:
Membuat label awal (L1).
Melakukan pengecekan kondisi. Jika salah, lompat ke akhir (goto L2).
Mengeksekusi isi loop (menggunakan variabel sementara t1).
Lompat kembali ke atas (goto L1) untuk pengecekan ulang.
Label akhir (L2).
