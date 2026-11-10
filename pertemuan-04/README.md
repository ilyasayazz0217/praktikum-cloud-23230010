1. Container Alpine berjalan di host Ubuntu, tetapi uname -r keduanya sama. Apa yang dibuktikan hasil ini tentang perbedaan container dan VM?
Bukti: Hasil ini membuktikan bahwa container berbagi kernel sistem operasi yang sama dengan host-nya (Ubuntu).
Perbedaan dengan VM (Virtual Machine):
Virtual Machine menjalankan sistem operasi tamu (guest OS) lengkap dengan kernelnya sendiri di atas hypervisor, sehingga membutuhkan resource yang jauh lebih besar.
Container melakukan virtualisasi pada level sistem operasi (OS-level virtualization). Container mengisolasi proses, network, dan filesystem menggunakan fitur Linux namespaces dan cgroups, tetapi tetap menumpang pada satu kernel host yang sama. Itulah sebabnya perintah uname -r menunjukkan versi kernel yang sama persis (6.8.0-147-generic).

2. Apa perbedaan image dan container? Gunakan contoh dari Langkah 8, ketika satu image Nginx dapat menjadi beberapa container.
Perbedaan Utama:
Image: Adalah template atau cetak biru (blueprint) yang bersifat read-only (tidak bisa diubah). Image berisi kode program, dependensi, perpustakaan, dan instruksi untuk menjalankan aplikasi.
Container: Adalah instance atau wujud aktif dari sebuah image yang sedang berjalan. Container menambahkan writable layer (lapisan yang bisa ditulisi) di atas image read-only tersebut sehingga proses aplikasi dapat dieksekusi.
Contoh: Berdasarkan praktik, satu image dasar seperti nginx atau nginx:alpine dapat di-deploy atau dijalankan menjadi beberapa container terpisah secara bersamaan (misalnya container web di port 8080, web-alpine di port 8081, dan web-lama di port 8082). Setiap container berjalan secara independen meskipun berasal dari jenis image yang sama.

3. Pada perintah docker run -p 8081:80 nginx:alpine, angka mana yang merupakan port host dan mana port container? Mengapa ketiga container Nginx boleh sama-sama memakai port 80 di dalam container?
Identifikasi Port (8081:80):
8081 adalah port pada host (mesin Ubuntu luar).
80 adalah port pada container (tempat layanan Nginx di dalam berjalan).
Alasan boleh sama-sama pakai port 80 di dalam container:
Setiap container memiliki namespace jaringan (network namespace) sendiri yang terisolasi. Artinya, setiap container memiliki alamat IP virtual dan ruang port internal yang independen satu sama lain. Port 80 di dalam container web, port 80 di dalam container web-alpine, dan port 80 di dalam container web-lama berada di ruang yang terpisah dan tidak saling bertabrakan. Yang tidak boleh sama adalah port sisi host-nya (8080, 8081, dan 8082) jika ingin diakses dari luar melalui port host yang sama.

4. Mengapa perubahan halaman di Langkah 5 hilang setelah docker rm lalu docker run, tetapi tidak hilang setelah docker stop lalu docker start?
Setelah docker stop lalu docker start: Perintah ini hanya mematikan lalu menyalakan kembali container yang sama tanpa menghapus filesystem-nya. Writable layer container tetap dipertahankan, sehingga modifikasi file index.html yang dilakukan pada Langkah 5 tetap ada.
Setelah docker rm lalu docker run: Perintah docker rm menghapus container lama beserta seluruh data di writable layer-nya secara permanen. Ketika Anda melakukan docker run lagi, Docker membuat container baru dari image aslinya (yang belum dimodifikasi), sehingga halaman HTML kembali ke tampilan default bawaan Nginx.

5. Bandingkan waktu dan langkah memasang Nginx di pertemuan 2 (dengan apt) dan di pertemuan ini (dengan Docker). Apa keuntungan container bagi tim yang harus menjalankan aplikasi yang sama di banyak server cloud?
Perbandingan Waktu & Langkah:
Menggunakan apt (pertemuan 2): Membutuhkan langkah instalasi manual yang panjang (update repository, apt install nginx, konfigurasi file satu per satu, penyesuaian dependensi sistem operasi, dan rentan terhadap perbedaan versi antar server). Waktu instalasi dan konfigurasi relatif lebih lama.
Menggunakan Docker: Hanya memerlukan satu baris perintah (docker run -d ...), otomatis mengunduh image yang sudah siap pakai dalam hitungan detik, tanpa pusing memikirkan konflik dependensi sistem operasi host.
Keuntungan bagi Tim di Cloud:
Konsistensi ("It works on my machine" is solved): Lingkungan aplikasi di laptop developer, staging, hingga production di berbagai server cloud dijamin 100% sama persis karena dibungkus dalam image yang identik.
Portabilitas & Kecepatan Deployment: Image dapat dipindahkan dan dijalankan di server cloud mana pun (AWS, GCP, Azure, DigitalOcean) dengan cepat tanpa konfigurasi ulang yang rumit dari awal.
