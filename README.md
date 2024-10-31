![b82aa7ac3f736dd78570dd3fa3fa9e24-java-programming-language-icon](https://github.com/user-attachments/assets/1425ea7f-f748-47f6-9d52-c27765cdbf0e)

#   Aplikasi Penghitung Umur

Sebuah aplikasi untuk menghitung umur pengguna secara akurat dengan menampilkan umur, bulan dan hari. Menampilkan kapan ulang tahun pengguna selanjutnya, serta menampilkan peristiwa peristiwa penting dalam sejarah pada tanggal ulang tahun pengguna yang di translate ke dalam Bahasa Indonesia.

Menggunakan bahasa pemrograman java dengan berbasis JFrameForm dan menggunakan library json (https://github.com/stleary/JSON-java?tab=readme-ov-file) yang melakukan fetching terhadap url (https://byabbe.se/on-this-day/).



## Fitur
    1. Mengetahui umur kita sekarang dalam tahun, bulan dan hari.
    2. Menampilkan peristiwa penting dalam Bahasa Indonesia berdasarkan tanggal ulang tahun pengguna.
    3. Mengetahui kapan ulang tahun kita yang selanjutnya.
## Dokumentasi
## PenghitungUmurForm.java
- Method tombol hitung (form)
```java
private void btnHitungActionPerformed(java.awt.event.ActionEvent evt) { 
    Date tanggalLahir = dateChooserTanggalLahir.getDate();
    if (tanggalLahir != null) {
        LocalDate lahir = tanggalLahir.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
        LocalDate sekarang = LocalDate.now();
        String umur = helper.hitungUmurDetail(lahir, sekarang);
        txtUmur.setText(umur);

        LocalDate ulangTahunBerikutnya = helper.hariUlangTahunBerikutnya(lahir, sekarang);
        String hariUlangTahunBerikutnya = helper.getDayOfWeekInIndonesian(ulangTahunBerikutnya);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
        String tanggalUlangTahunBerikutnya = ulangTahunBerikutnya.format(formatter);
        
        txtHariUlangTahunBerikutnya.setText(hariUlangTahunBerikutnya + " (" + tanggalUlangTahunBerikutnya + ")");
        
        stopFetching = true;
        if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt(); // Beri sinyal ke thread untuk berhenti
        }
        stopFetching = false;

        // Mendapatkan peristiwa penting secara asinkron
        peristiwaThread = new Thread(() -> {
            try {
                txtAreaPeristiwa.setText("Tunggu, sedang mengambil data...\n");
                helper.getPeristiwaBarisPerBaris(ulangTahunBerikutnya, txtAreaPeristiwa, () -> stopFetching);
                if (!stopFetching) {
                    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append("Selesai mengambil data peristiwa"));
                }
            } catch (Exception e) {
                if (Thread.currentThread().isInterrupted()) {
                    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                }
            }
        });
        peristiwaThread.start();
    }
}

``` 
- Method saat date chooser berubah (form)

```java
private void dateChooserTanggalLahirPropertyChange(java.beans.PropertyChangeEvent evt) {                                                
        txtUmur.setText("");
        txtHariUlangTahunBerikutnya.setText("");
        stopFetching = true;
        if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt();
        }
        txtAreaPeristiwa.setText("");
    }     
```

- Method tombol keluar
``` java
System.exit(0);
```

## PenghitungUmurHelper.java

- Method penghitungan umur secara detail (kelas helper)
``` java
 public String hitungUmurDetail(LocalDate lahir, LocalDate sekarang){
       Period period = Period.between(lahir, sekarang);
       return period.getYears() + " tahun, " + period.getMonths() + " bulan, " + period.getDays() + " hari";
    }
```

- Method perhitungan ulang tahun berikutnya (kelas helper)
```java
 public LocalDate hariUlangTahunBerikutnya (LocalDate lahir, LocalDate sekarang) {
        LocalDate ulangTahunBerikutnya = lahir.withYear(sekarang.getYear());
        if (!ulangTahunBerikutnya.isAfter(sekarang)) {
            ulangTahunBerikutnya = ulangTahunBerikutnya.plusYears(1);
        } return ulangTahunBerikutnya;
    }
```

- Method untuk mengambil nama hari dalam Bahasa Indonesia (kelas helper)
``` java
public String getDayOfWeekInIndonesian (LocalDate date){
        switch (date.getDayOfWeek()){
            case MONDAY :
                return "Senin";
            case TUESDAY :
                return "Selasa";
            case WEDNESDAY :
                return "Rabu";
            case THURSDAY :
                return "Kamis";
            case FRIDAY :
                return "Jum'at";
            case SATURDAY :
                return "Sabtu";
            case SUNDAY :
                return "Minggu";
            default :
                return "";
        }
```

- Method untuk menampilkan peristiwa per baris (kelas helper)
``` java
 public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
        try {
                if (shouldStop.get()) {
                    return;
                }
            String urlString = "https://byabbe.se/on-this-day/" +
            tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
            URL url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection)
            url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");
            int responseCode = conn.getResponseCode();
            if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode +
            ". Silakan coba lagi nanti atau cek koneksi internet.");
        }
        BufferedReader in = new BufferedReader(new
        InputStreamReader(conn.getInputStream()));
        String inputLine;
        StringBuilder content = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            if (shouldStop.get()) {
                in.close();
                conn.disconnect();
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
            content.append(inputLine);
        }
        in.close();
        conn.disconnect();
        JSONObject json = new JSONObject(content.toString());
        JSONArray events = json.getJSONArray("events");
        for (int i = 0; i < events.length(); i++) {
            if (shouldStop.get()) {
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
        JSONObject event = events.getJSONObject(i);
        String year = event.getString("year");
        String description = event.getString("description");
        String translatedDescription = translateToIndonesian(description);
        String peristiwa = year + ": " + translatedDescription;
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append(peristiwa + "\n"));
        }
        if (events.length() == 0) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
        }
        } catch (Exception e) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
        }
    }
```

- Method terjemah peristiwa penting ke bahasa Indonesia
```java
 public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
        try {
                if (shouldStop.get()) {
                    return;
                }
            String urlString = "https://byabbe.se/on-this-day/" +
            tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
            URL url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection)
            url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");
            int responseCode = conn.getResponseCode();
            if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode +
            ". Silakan coba lagi nanti atau cek koneksi internet.");
        }
        BufferedReader in = new BufferedReader(new
        InputStreamReader(conn.getInputStream()));
        String inputLine;
        StringBuilder content = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            if (shouldStop.get()) {
                in.close();
                conn.disconnect();
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
            content.append(inputLine);
        }
        in.close();
        conn.disconnect();
        JSONObject json = new JSONObject(content.toString());
        JSONArray events = json.getJSONArray("events");
        for (int i = 0; i < events.length(); i++) {
            if (shouldStop.get()) {
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
        JSONObject event = events.getJSONObject(i);
        String year = event.getString("year");
        String description = event.getString("description");
        String translatedDescription = translateToIndonesian(description);
        String peristiwa = year + ": " + translatedDescription;
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append(peristiwa + "\n"));
        }
        if (events.length() == 0) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
        }
        } catch (Exception e) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
        }
    }
```

## Screenshots
![Screenshot 2024-10-31 140502](https://github.com/user-attachments/assets/36fe6f1e-aae6-4dd1-b19e-191933e030c5)

## Pembuat

- Nama : Muhammad Raihan Fadhillah
- NPM : 2210010404
