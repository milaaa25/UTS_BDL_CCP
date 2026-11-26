| Keputusan | Nilai | Alasan |
|---|---|---|
| Nama topic | CCP-wiki-raw | Awalan kelompok mencegah tabrakan dengan kelompok lain di broker yang sama; akhiran 'raw' menandai data mentah sebelum diolah. |
| Jumlah partisi | 8 | Sama dengan total core executor (4 Worker × 2 core): satu batch Kafka langsung menjadi 8 task paralel dan habis dibagi 4 consumer (2 partisi per Worker). Partisi bisa ditambah, tidak bisa dikurangi. |
| Key pesan | wiki\|title | Suntingan halaman yang sama selalu masuk partisi yang sama (urutan per halaman terjaga, berguna untuk deteksi edit war), sementara beban tersebar karena jumlah halaman sangat banyak. Duplikat akibat reconnect SSE memiliki key sama sehingga jatuh di partisi yang sama. |
| Replikasi | 1 | Lab hanya memiliki 1 broker dan faktor replikasi tidak boleh melebihi jumlah broker. Risiko: jika PC Master mati, data tidak tersedia (keterbatasan). |
| Retensi | 72 jam (retention.ms=259200000) | Cukup untuk mengumpulkan ≥200.000 pesan, mengulang snapshot, dan menjalankan ulang streaming selama masa ujian tanpa memenuhi disk PC Master. |
| Consumer group | CCP-wiki-pemroses (4 Worker); Spark menyimpan offset di checkpoint | Dalam satu group, tiap partisi dibaca tepat satu Worker (pembagian kerja + failover saat Worker mati). Spark Structured Streaming tidak memakai commit group; offset dicatat di checkpointLocation. |
| Kompresi & keandalan producer | lz4, acks=all, enable.idempotence=true | lz4 menghemat jaringan dengan CPU rendah; acks=all + idempotence mencegah pesan hilang atau ganda dari sisi producer. |