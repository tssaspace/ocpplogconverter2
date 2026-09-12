# Pelacak Log OCPP

Ubah CSV log OCPP dari ekspor CSMS menjadi tabel XLSX siap tempel ke laporan.

**Aplikasi web: <https://tssa.space/ocpplogconverter2/>** (berjalan sepenuhnya di
browser, file tidak dikirim ke mana pun, tanpa dependensi eksternal).

Penerus dari [ocpplogconverter](https://tssa.space/ocpplogconverter/), dengan
format keluaran yang mengikuti berkas laporan `TC-OFF-S2 CHP9 Pt1.xlsx`.

## Yang dihasilkan

Empat kolom: `Waktu (UTC+8)` | `UUID` | `Command` | `Payload`.

Response (CALLRESULT) digabung ke baris request-nya, jadi satu pesan OCPP =
satu baris. Payload diringkas per command:

| Command | Payload |
|---|---|
| Authorize | `` `{"idTag":"0dc25355"}` -> `Accepted` `` |
| StartTransaction | `` `{"connectorId":1,"meterStart":430196429,"transactionId":212657}` `` |
| StatusNotification | `` `{"connectorId":1,"status":"Charging"}` `` |
| MeterValues | `` `{...,"meterValue":[{"value":"430196429","measurand":"Energy.Active.Import.Register"}]}` *(Sampling)* `` |
| Disconnect | `` `{"code":1006,"reason":"Abnormal Closure"}` `` |

XLSX-nya: header biru tebal, freeze pane, autofilter, kolom payload monospace.

## Filter

| Filter | Bawaan |
|---|---|
| Command | semua tampil; ada preset laporan (sembunyikan Heartbeat) |
| MeterValues | aktif - hanya sampel setelah StartTransaction, sebelum StopTransaction, dan setelah StopTransaction |
| Rentang waktu | kosong (seluruh log); ada tombol "Pas ke transaksi" |
| Connector & Transaction ID | semua tercentang |
| Zona waktu | UTC+8 (WITA) |

Peringkas MeterValues mengelompokkan sampel per `transactionId`, jadi log
dengan beberapa sesi di satu connector tetap benar. Kalau StopTransaction
tidak ada (log terputus), sampel terakhir tetap diambil.

Aturan ringkas payload per command bisa diubah sendiri di panel Payload.

CSV wajib punya kolom `created_at` dan `command`; `payload`, `payload_raw`,
`message_id`, `message_type`, dan `connector_number` dipakai kalau ada.

## Struktur

    index.html    seluruh aplikasi dalam satu berkas, tanpa dependensi

Parser CSV, peringkas payload, penulis ZIP/XLSX, dan antarmukanya semua ada
di dalam `index.html`. Tidak memuat apa pun dari jaringan, jadi berkasnya bisa
disimpan dan dibuka langsung dari Finder.
