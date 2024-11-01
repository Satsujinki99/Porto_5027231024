# Monitoring Jarak Jauh Menggunakan IoT dan ThingSpeak dengan ESP

## Introduction
Di era modern ini, Internet of Things (IoT) telah membawa perubahan besar dalam berbagai aspek kehidupan, memungkinkan perangkat untuk saling terhubung dan berbagi data secara real-time. Salah satu aplikasi populer IoT adalah monitoring jarak jauh, yang memungkinkan pengguna untuk memantau kondisi lingkungan, seperti suhu, kelembapan, dan kualitas udara, tanpa perlu berada di lokasi fisik tersebut.

Dengan menggunakan platform ThingSpeak, monitoring jarak jauh menjadi lebih mudah. ThingSpeak adalah layanan cloud yang dirancang khusus untuk mengumpulkan, menyimpan, dan menganalisis data IoT, serta memberikan visualisasi data secara real-time. Dalam tutorial ini, kita akan membangun sistem IoT sederhana untuk memantau suhu dan kelembapan menggunakan perangkat ESP8266 atau ESP32 yang terhubung ke ThingSpeak.

Tutorial ini dirancang untuk membantu Anda memahami cara merakit perangkat, mengonfigurasi ThingSpeak, dan mengirim data sensor ke cloud. Di akhir tutorial ini, Anda akan memiliki sistem monitoring jarak jauh yang berfungsi dan dapat diperluas untuk berbagai aplikasi lain. Untuk mempermudah visualisasi tutorial, aplikasi sejenis wokwi dapat digunakan untuk menampilkan instruksi pemasangan dan simulasi perangkat.

Berikut adalah alat dan bahan yang dibutuhkan untuk monitoring suhu dan kelembapan menggunakan ThingSpeak:

- Hardware: ESP8266/ESP32, sensor (DHT22 untuk suhu dan kelembapan).
  
- Software: Arduino IDE, akun ThingSpeak. (bisa gunakan wokwi jika tanpa hardware).

- Library: DHT sensor library for ESPx , Wi-Fi library untuk ESP8266/ESP32.


## Langkah-langkah :

1. Siapkan board, sensor, dan jumper cable. Hubungkan board dan sensor dengan cable jumper yang digunakan.
   
  <img width="672" alt="image" src="https://github.com/user-attachments/assets/5f557051-317c-4660-88fe-92d34ffa6fd8">


2, Buat channel ThingSpeak dengan 2 field untuk data suhu dan data kelembapan.
        
  <img width="672" alt="image" src="https://github.com/user-attachments/assets/e5fccaac-c0ac-4533-a646-05a67170248f">



3. Mulai coding board dengan code berikut, pastikan ubah API sesuai dengan channel yang anda buat.

 ```
#include "WiFi.h" // Deklarasi library WiFi
#include <DHTesp.h> // Deklarasi library DHTesp untuk sensor suhu dan kelembapan

WiFiClient client; // Membuat objek client untuk WiFi

const int DHT_PIN = 15; // Pin untuk sensor DHT
DHTesp dhtSensor; // Objek sensor DHT

String thingSpeakAddress = "api.thingspeak.com"; // Alamat server ThingSpeak
String request_string; // Variabel untuk menyimpan URL request

void setup() {
  Serial.begin(115200);
  WiFi.disconnect(); // Memutuskan koneksi WiFi sebelumnya
  WiFi.begin("Wokwi-GUEST", ""); // Menyambungkan ke WiFi dengan SSID dan password
  
  // Tunggu hingga ESP32 tersambung ke WiFi
  while (WiFi.status() != WL_CONNECTED) {
    delay(300);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  
  // Inisialisasi sensor DHT
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {
  delay(2000); // Menunggu dua detik setiap loop

  // Mengambil data suhu dan kelembapan
  TempAndHumidity data = dhtSensor.getTempAndHumidity();
  float t = data.temperature; // Suhu
  float h = data.humidity;    // Kelembapan

  // Periksa validitas data sebelum mengirim ke ThingSpeak
  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Menampilkan data ke Serial Monitor untuk debugging
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.print("°C, Humidity: ");
  Serial.print(h);
  Serial.println("%");

  // Mengirim data suhu dan kelembapan ke ThingSpeak
  kirim_thingspeak(t, h);
}

void kirim_thingspeak(float suhu, float hum) {
  if (client.connect(thingSpeakAddress.c_str(), 80)) {
    // Membuat string request untuk mengirim data suhu dan kelembapan
    request_string = "/update?api_key=";
    request_string += "api_keys"; // API key untuk ThingSpeak
    request_string += "&field1=";
    request_string += String(suhu, 2);    // Suhu ke field1
    request_string += "&field2=";
    request_string += String(hum, 2);     // Kelembapan ke field2

    // Mengirim request ke server dan menampilkan request ke Serial Monitor
    Serial.println("Sending data to ThingSpeak...");
    Serial.println(String("GET ") + request_string + " HTTP/1.1\r\n" +
                   "Host: " + thingSpeakAddress + "\r\n" +
                   "Connection: close\r\n\r\n");

    client.print(String("GET ") + request_string + " HTTP/1.1\r\n" +
                 "Host: " + thingSpeakAddress + "\r\n" +
                 "Connection: close\r\n\r\n");

    // Menunggu respons dari ThingSpeak
    unsigned long timeout = millis();
    while (client.available() == 0) {
      if (millis() - timeout > 5000) {
        Serial.println(">>> Client Timeout!");
        client.stop();
        return;
      }
    }

    // Membaca respons dari server dan menampilkannya di Serial Monitor
    while (client.available()) {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    }

    Serial.println("\nConnection closed");
    client.stop();
  } else {
    Serial.println("Failed to connect to ThingSpeak");
  }
}
```

5. Jalankan simulasi di wokwi atau mulai upload codenya melalui aplikasi Arduino IDE ke boardnya.
6. Lihat hasil visualisasi data di ThingSpeak.
   
   <img width="586" alt="image" src="https://github.com/user-attachments/assets/ec20dbd7-c4cc-46da-8630-f4aa2912531e">



## Conclusion
Dengan mengikuti tutorial ini, seharusnya kita telah berhasil membuat sistem monitoring jarak jauh yang memungkinkan pengumpulan dan visualisasi data suhu dan kelembapan secara real-time menggunakan ESP8266/ESP32 dan ThingSpeak. Prosesnya mencakup langkah-langkah penting, mulai dari menghubungkan perangkat keras, menulis kode untuk mengirim data ke cloud, hingga melihat hasilnya di dashboard ThingSpeak.

Proyek ini menunjukkan bahwa dengan IoT, kita bisa memantau kondisi lingkungan dari mana saja, menjadikannya solusi ideal untuk berbagai aplikasi, mulai dari smart home hingga industri. Anda dapat memperluas proyek ini dengan menambahkan sensor lain, mengatur alarm otomatis, atau mengintegrasikannya dengan layanan seperti IFTTT untuk menciptakan aksi otomatis berdasarkan kondisi tertentu.

Kami harap tutorial ini memberikan pemahaman dasar yang kuat bagi Anda untuk mengeksplorasi lebih jauh dan mengembangkan aplikasi IoT sesuai kebutuhan Anda. Selamat mencoba dan terus eksplorasi dunia IoT!

wokwi project: https://wokwi.com/projects/413049560059403265
