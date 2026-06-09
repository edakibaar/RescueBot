# RescueBot
# RescueBot: Çok Boğumlu Modüler Arama-Kurtarma Robotu

Bu depo (repository), afet ve enkaz sahalarında lojistik, canlı tespiti ve teleoperasyon faaliyetlerini yürütmek üzere geliştirilen **RescueBot** mekatronik platformunun tüm kaynak kodlarını ve yazılım mimarisini barındırmaktadır. Proje; dağıtık işlemci mimarisi, gerçek zamanlı kablosuz veri iletimi ve donanımsal emniyet kilitleri üzerine inşa edilmiştir.

## 🛠️ Donanım ve İşlemci Mimarisi

Sistem, görev paylaşımı yapılmış 3 bağımsız ana işlem biriminden oluşmaktadır:
1. **ESP32-CAM (Ağ Geçidi ve Sunucu):** Operatör arayüzünü (HTML/CSS/JS) lokal ağ üzerinden sunar. Kameradan kesintisiz 30 FPS video yayını akıtır ve gelen sürüş komutlarını Master birimine UART üzerinden köpürüler.
2. **Master Arduino Uno (Merkezi Yürütücü):** L293D Motor Shield ile robotun itki sistemini (4x DC Motor) ve belden kırma kinematik yapısını (2x Servo) yönetir. Bloklamasız `millis()` zamanlayıcısı ile her 500 ms'de bir Slave biriminden veri talep eder.
3. **Slave Arduino Uno (Sensör Veri Merkezi):** Robotun ön boğumunda yer alır. Canlı tespiti için HC-SR501 (PIR) ve mesafe kontrolü için HC-SR04 (Ultrasonik) sensörlerini sürekli tarayarak verileri I2C veri yolu üzerinden Master işlemciye paketler.

## 📡 Haberleşme Protokolleri

* **UART (Gürültü Korumalı Paketleme):** ESP32 ve Master Uno arasındaki seri iletişim hat gürültülerinden arındırılmak adına `<KOMUT>` (Örn: `<F>`, `<EMG>`) sarmal paket yapısıyla filtrelenir.
* **I2C Hat Kesmesi:** Master ve Slave Uno arasındaki veri lojistiği `Wire.h` kütüphanesi ve `volatile` değişken takılarıyla donanımsal interrupt bazlı (Adres: 8) asenkron olarak yürütülür.

## ⚠️ Mühendislik Notu: Çözülen Kritik Hata

Projenin entegrasyon fazında `SoftwareSerial` kütüphanesinin analog `A0-A1` pinlerinde kullanılması, dahili zamanlayıcı (`Timer1`) donanımının `Servo.h` kütüphanesiyle çakışmasına ve servolarda sürekli titremeye (jitter) neden olmuştur. Yazılımsal seri port tamamen kaldırılarak ESP32 doğrudan donanımsal **Hardware Serial (RX/TX)** hattına taşınmış; böylece zamanlayıcı çakışmaları çözülerek hem veri akışı hem de aktüatör kararlılığı %100 güvenceye alınmıştır.

---
*Bu proje Halit Yarımbatman ve Eda Kibar tarafından geliştirilmiştir.*
