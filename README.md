# AplikasiPenghitungUmur
 latihan 2- Seidi Rahmat(2210010262)
 Penghitungan Umur Detail
Metode hitungUmurDetail(LocalDate lahir, LocalDate sekarang) menghitung umur dengan menggunakan Period.between() untuk mendapatkan selisih tahun, bulan, dan hari antara dua tanggal.

public String hitungUmurDetail(LocalDate lahir, LocalDate sekarang) {
    Period period = Period.between(lahir, sekarang);
    return period.getYears() + " tahun, " + period.getMonths() + " bulan, " + period.getDays() + " hari";
}

Hari Ulang Tahun Berikutnya
Metode hariUlangTahunBerikutnya(LocalDate lahir, LocalDate sekarang) menentukan ulang tahun berikutnya. Jika ulang tahun tahun ini sudah lewat, akan mencari ulang tahun di tahun berikutnya.

public LocalDate hariUlangTahunBerikutnya(LocalDate lahir, LocalDate sekarang) {
    LocalDate ulangTahunBerikutnya = lahir.withYear(sekarang.getYear());
    if (ulangTahunBerikutnya.isBefore(sekarang) || ulangTahunBerikutnya.isEqual(sekarang)) {
        ulangTahunBerikutnya = ulangTahunBerikutnya.plusYears(1);
    }
    return ulangTahunBerikutnya;
}

Mendapatkan Peristiwa Bersejarah
Metode getPeristiwaBarisPerBaris() mengambil data peristiwa dari API eksternal berdasarkan tanggal ulang tahun pengguna, kemudian menampilkannya satu per satu dalam JTextArea. Proses pengambilan data bisa dibatalkan melalui parameter Supplier<Boolean> shouldStop yang memeriksa apakah thread harus dihentikan.

public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
    try {
        String urlString = "https://byabbe.se/on-this-day/" + tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
        URL url = new URL(urlString);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
        conn.setRequestProperty("User-Agent", "Mozilla/5.0");
        int responseCode = conn.getResponseCode();
        if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode);
        }

        StringBuilder content;
        try (BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
            String inputLine;
            content = new StringBuilder();
            while ((inputLine = in.readLine()) != null) {
                content.append(inputLine);
            }
        }
        conn.disconnect();

        JSONObject json = new JSONObject(content.toString());
        JSONArray events = json.getJSONArray("events");

        for (int i = 0; i < events.length(); i++) {
            JSONObject event = events.getJSONObject(i);
            String year = event.getString("year");
            String description = event.getString("description");
            String translatedDescription = translateToIndonesian(description);
            String peristiwa = year + ": " + translatedDescription;
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append(peristiwa + "\n"));
        }

    } catch (Exception e) {
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
    }
}

Terjemahan Deskripsi Peristiwa
Untuk menerjemahkan deskripsi peristiwa dari bahasa Inggris ke bahasa Indonesia, digunakan API dari lingva.ml. Jika terjemahan gagal, teks asli akan tetap ditampilkan dengan pesan kesalahan.
private String translateToIndonesian(String text) {
    try {
        String urlString = "https://lingva.ml/api/v1/en/id/" + text.replace(" ", "%20");
        URL url = new URL(urlString);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
        conn.setRequestProperty("User-Agent", "Mozilla/5.0");
        int responseCode = conn.getResponseCode();
        if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode);
        }

        StringBuilder content;
        try (BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream(), "utf-8"))) {
            String inputLine;
            content = new StringBuilder();
            while ((inputLine = in.readLine()) != null) {
                content.append(inputLine);
            }
        }
        conn.disconnect();
        JSONObject json = new JSONObject(content.toString());
        return json.getString("translation");
    } catch (Exception e) {
        return text + " (Gagal diterjemahkan)";
    }
}

 Hari Ulang Tahun dalam Bahasa Indonesia
Metode getDayOfWeekInIndonesian() mengubah nama hari ulang tahun menjadi bahasa Indonesia menggunakan DayOfWeek dari Java.

public String getDayOfWeekInIndonesian(LocalDate date) {
    return switch (date.getDayOfWeek()) {
        case MONDAY -> "Senin";
        case TUESDAY -> "Selasa";
        case WEDNESDAY -> "Rabu";
        case THURSDAY -> "Kamis";
        case FRIDAY -> "Jumat";
        case SATURDAY -> "Sabtu";
        case SUNDAY -> "Minggu";
    };
}



