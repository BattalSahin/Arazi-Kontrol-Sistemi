# 🛰️ Uydu GPS Pro (v2.5) — Kapsamlı Saha, Uydu Harita ve Navigasyon Aracı

Bu proje, bilgisayar, tablet ve akıllı telefonlarda çalışan, yüksek çözünürlüklü uydu görüntüleri, **Anti-Drift (GPS titreşim/sapma engelleme)**, **hedef koordinat yaklaşma takibi**, **sesli/titreşimli varış ikazı**, **vektörel çizim araçları** ve **özel harita/parsel fotoğrafı perspektif hizalama** yeteneklerine sahip profesyonel bir web uygulamasıdır.

Build adımı veya harici bağımlılık kurulumu gerektirmez. Vanilla HTML, CSS ve JavaScript ile çalışır.

---

## 📂 Dosya Yapısı

- **`index.html`** — Giriş Portalı: Cihaz sensörleri (GPS, pusula, Web Audio) hazırlık durumu, özellik tanıtımları ve `app.html` sayfasına yönlendiren şık "Uydu Sayfasına Git" karşılama ekranı.
- **`app.html`** — Ana Navigasyon ve Harita Uygulaması: Tüm harita katmanları, Anti-Drift filtresi, hedef koordinat takibi, sesli melodi sentezleyici, çizim motoru, fotoğraf eşleme ve dışa/içe aktarma motoru.
- **`README.md`** — Proje dökümantasyonu ve saha test rehberi.

---

## 🎯 Temel Özellikler

### 1. 🛰️ 8 Ultra Net Uydu ve Arazi Haritası
Üst barda yer alan **"Uydu Seç"** butonu ile açılan pencereden arazi koşuluna en uygun katman seçilebilir:
1. **Google Uydu (Maxar HD) [Varsayılan]:** Zoom 22 seviyesine kadar (~30 cm çözünürlük) dağlık ve kayalık arazilerde en net, en güncel ve en keskin görüntü.
2. **Google Hibrit:** Yüksek çözünürlüklü uydu zeminine entegre yol ağları, yerleşim adları ve sınırlar.
3. **Esri Clarity (Kristal Netlik):** Bulut, pus ve atmosferik gölgelerden tamamen arındırılmış yüksek kontrastlı uydu görüntüleri.
4. **Yandex Uydu:** ScanEx ve Airbus lisanslı, özellikle Doğu Avrupa, Türkiye ve kırsal alanlarda çok zengin detay sunan alternatif uydu katmanı.
5. **Esri World Imagery:** Doğal renk dengesine sahip dengeli küresel uydu mozaikleri.
6. **OpenTopoMap (Eş Yükselti İzohipsleri):** 10m ve 20m aralıklı eş yükselti eğrileri, tepe noktaları, rakım bilgileri ve dağ patikaları.
7. **Google Arazi (Fiziki Rölyef):** Dağ silsileleri, vadiler ve coğrafi yükseklik farklarını gösteren fiziki harita.
8. **Esri 3B Hillshade (Gölge Kabartma):** Güneş açısıyla gölgelendirilmiş 3 boyutlu vadi ve sırt topografyası.

### 2. 🛡️ Kaya Gibi Sabit Anti-Drift (Çift Eşikli Histerezis + Kalman)
- Mobil cihazlar arazide veya sabit dururken bile uydulardan gelen sinyallerde ±3 ile ±15 metre arasında hayali zıplamalar (GPS jitter/drift) oluşur.
- **`AntiDriftKalmanLocator` v2 (Çift Eşikli Durum Makinesi):**
  - **Çift Eşikli Histerezis:** Tekil anlık GPS gürültüleri durağan kilitlenmeyi ASLA bozamaz. Hareket moduna geçmek için ardışık en az 2 örneğin $\ge 3.8\text{ m}$ mesafe veya $\ge 1.2\text{ km/s}$ hız üretmesi gerekir.
  - **Sıfır Titreme (Dead-band Lock):** Sabit moddayken nokta anchor koordinatına kenetlenir; harita ve marker mikro titreşimleri sıfıra indirilir.
  - **Akıllı Harita Takibi:** Harita her saniye `map.setView` ile sarsılmak yerine, sadece kullanıcı merkezden belirgin şekilde uzaklaştığında (`> 3.5m`) akıcı `map.panTo` ile kaydırılır.
  - **İpeksi Marker Geçişi:** Leaflet marker'ına eklenen CSS eğrisi sayesinde kullanıcı adımları takılmadan, su gibi akar.

### 2.1. 🧭 İpeksi Yumuşaklıkta Yön & Pusula Senkronizasyonu (HeadingEngine)
- **60fps/120fps Donanımsal Senkronizasyon:** `requestAnimationFrame` döngüsüyle ekran tazeleme hızına kilitlenmiştir.
- **Çakışan CSS Gecikmesi Kaldırıldı:** 60Hz sensör verisi ile CSS transition arasındaki yarış durumu ve takılmalar tamamen temizlendi.
- **En Kısa Açısal Yol (Shortest Path Slerp):** $359^\circ \to 1^\circ$ Kuzey geçişlerinde yaşanan 358 derecelik fırıldak dönmesi ve zıplamalar yok edildi.
- **Ölü Bölge (Deadband):** $0.5^\circ$ altındaki el titremesi mikro salınımları filtrelenerek koninin zangırdaması engellendi; dönüşlerde dinamik $\alpha$ katsayısı ile anında ve akıcı tepki verir.
- **Harita Döndürme Telafisi:** Harita elle veya serbest açıyla döndürüldüğünde bile koni açısı anlık olarak `ekranAçısı = cihazYönü - haritaAçısı` formülü ile milimetrik senkronize kalır.

### 3. 🎯 Hedef Koordinat Belirleme & Canlı Yön/Adım Kılavuzu
- Sağ alttaki **Hedef (Nişangah FAB)** butonu ile açılan panelden:
  - Elle **Enlem** ve **Boylam** girilebilir (Örn: `39.920800, 32.854100`).
  - Panodan kopyalanan koordinat tek tıkla yapıştırılabilir.
  - Anlık GPS konumu hedef olarak atanabilir.
  - **"Haritadan Tıklayarak Seç"** butonuyla harita üzerinde herhangi bir kayalık, tepe veya nokta işaretlenebilir.
- **Canlı Turn-by-Turn Yön & Adım Rehberi:**
  - Cihazın baktığı yön ile hedef açısı dinamik kıyaslanır: `"Düz ilerle"`, `"Hafif sağa dön"`, `"Sağa dön"`, `"Arkana dön"`, `"Sola dön"`, vb.
  - Adım hesabı (1 yetişkin adımı ~ 0.75m): Örneğin `"Sola dön, tahmini 14 adım (10 m) yürü."` veya `"Arkana dön, tahmini 65 adım (49 m) yürü."`
  - HUD üzerindeki yön oku, dönmeniz gereken açıya göre gerçek zamanlı olarak döner.

### 4. 🔊 Yaklaştıkça Sıklaşan Bip Sesi (Sonar) & Türkçe Sesli Konuşma
- **Proximity Sonar Beeper (Mesafe Yaklaştıkça Hızlanan Bip Sesi):**
  - Hedefe yaklaştıkça periyot kısalır (uzaktayken 3.2 saniyede bir yumuşak bip, 50 metrede 1.2s, 10 metrede 400ms, 4 metrede hızlı 200ms geiger vuruşu).
  - Target HUD üzerinden tek dokunuşla sessize alınabilir (`Bip Sesi: Açık/Kapalı`).
- **Türkçe Sesli Asistan (Web Speech API):**
  - Hedef belirlendiğinde, yön değişikliklerinde veya mesafe eşiklerinde (100m, 50m, 20m, 10m) talimatı Türkçe sesli olarak okur.
  - **"Şimdi Seslendir"** butonuna basarak anlık talimatı istediğiniz her an sesli dinleyebilirsiniz.
- **Hedefe Varış Onay Melodisi & Titreşim:**
  - Noktanın tam üzerine gelindiğinde ($\le 3.5\text{ m}$) ve cihaz orada **3 saniye boyunca sabit durduğunda**:
    - Web Audio API tarafından sentezlenen 4 tonlu pozitif varış melodisi çalar.
    - Mobil cihazlarda titreşim (`navigator.vibrate`) tetiklenir.
    - Ekranda koordinat ve varış bilgilerini teyit eden yeşil onay modalı açılır.

### 5. 🧪 Saha GPS Hassasiyet Test Modu & Sade Arayüz
- **Üst Bar Kısayolları & 3 Çizgili Menü (☰):**
  - Üst bar: Sol başta **☰ Menü**, ortada **🛰️ Uydu Seçici**, sağ tarafta doğrudan tek dokunuşla açılan **🛰️ GPS & Telemetri** ve **📐 Çizim & Ölçüm Araçları** kısayolları yer alır.
  - Hedef belirleme butonu ise sağ alttaki ana işlem çubuğunda (FAB stack) merkezi olarak konumlandırılmıştır.
  - Test modu çalışırken menü ikonunda nabız atan mor bildirim rozeti yanar.
- **Kompakt Panel Tasarımı:**
  - **Hedef Koordinat Paneli:** Dikey yüksekliği %50 azaltıldı, gereksiz metinler kaldırıldı, tek satırlık hızlı butonlar ([Yapıştır], [Konumum], [Haritadan Seç]) eklendi.
  - **Target HUD:** Üst şerit daraltıldı; butonlar (`Bip: Açık`, `Ses: Açık`, `Seslendir`) tek satıra sığacak şekilde optimize edildi.
  - **Test Modu Paneli:** Mesafe, adım tahmini, maksimum sapma ve başlangıç noktasına dönüş oku içeren saha odaklı kompakt panel.

### 6. 🧭 Uydu Bakış Yönünü Döndürme (180° Ters Bakış, Serbest Açı & Canlı Heading-Up)
Arazide güneye veya farklı bir yöne doğru yürürken sabit Kuzey odaklı harita kafa karışıklığı yaratır. Bu sorunu tamamen ortadan kaldıran dinamik döndürme motoru:
- **🔄 180° Ters Çevir (Güney Bakışı):** Tek dokunuşla haritayı 180 derece ters yüz eder. Ters yöne (Güneye) bakarken haritayı önünüze göre hizalar, arazi gözünüzün gördüğü gibi ekrana gelir.
- **🧭 İnteraktif Pusula Gülü (Sağ Alt FAB):**
  - Kırmızı ucu daima Gerçek Kuzeyi gösteren 3 boyutlu dinamik pusula ibresi.
  - Harita döndüğünde ibre açıyı gösterir, **dokunulduğunda tek hamlede Kuzeye (0°) sıfırlar**.
  - Açı rozetine dokunulduğunda detaylı yön paneli açılır.
- **🧭 Canlı Pusula Takip Modu (Heading-Up):**
  - Açıldığında telefonun dahili manyetik pusulası haritaya kilitlenir; siz arazide vücudunuzu nereye dönerseniz uydu görüntüsü de anlık olarak o yöne döner (ileri yön = ekranın üstü).
- **👆 İki Parmakla Serbest Çevirme (Touch Gesture):**
  - Telefonda iki parmakla harita üzerinde burkma hareketi yaparak veya bilgisayarda açı sliderı ile haritayı 0°-359° arasında dilediğiniz açıya çevirebilirsiniz.
- **4 Ana Yön ve Kademeli Açı:** [Kuzey (0°)], [Doğu (90°)], [Güney (180°)], [Batı (270°)] ve [-90°, -15°, +15°, +90°] hassas adım tuşları.

### 7. ✏️ Vektörel Çizim ve Arazi Ölçümü (Leaflet Geoman)
- Sol kenardaki çizim çubuğu ile:
  - Nokta / İşaretçi (Marker)
  - Çember (Daire ve Yarıçap)
  - Çokgen (Parsel, Tarla veya Bölge Çizimi)
  - Kırık Çizgi (Yürüyüş Rotası, Patika, Dere Yatağı)
  - Dikdörtgen
  - **Serbest El Kalem Çizimi (Freehand):** Kalem aracı seçildiğinde **uydu harita tamamen dondurulur (kayma engellenir)**; telefon ekranında parmakla/stylus kalemle veya bilgisayarda fareyle doğrudan harita üstüne serbest çizim yapılabilir. "Seç" moduna geçildiğinde uydu tekrar serbestçe kaydırılabilir.
  - Metin / Not Ekleme
  - Mesafe ve Alan Ölçümü
- Çizilen nesnelere tıklanarak renk, çizgi kalınlığı ve dolgu şeffaflığı canlı olarak değiştirilebilir.

### 7. 🧩 Akıllı Puzzle Eşleştirme & Resim Hizalama (Auto-Snap, EXIF GPS ve 2 Noktalı Kilit)
- Sağ alttaki **"+"** butonu ile cihazdan bir kroki, kadastro planı veya havadan/dronedan çekilmiş fotoğraf seçildiğinde sistem 3 farklı akıllı yöntem sunar:
  1. **🛰️ Otomatik EXIF / Drone GPS Yerleşimi:** Fotoğraf bir akıllı telefon veya drone (DJI vb.) ile çekilmişse, gömülü GPS koordinatları milisaniyeler içinde okunur, harita doğrudan çekim alanına uçar ve fotoğraf gerçek dünya koordinatlarına yerleştirilir.
  2. **✨ Uyduyla Otomatik Eşleştir (Auto-Snap / AI Görsel Puzzle):** Uydu görüntüleri ve yüklenen fotoğraf taranır, Sobel kenar gradyanları ve yol/tarla sınırları çıkarılır. Çok ölçekli açı ve konum korelasyonu ile fotoğraf puzzle gibi uyduya kilitlenir.
  3. **🎯 2 Noktaya Dokunarak Kilitle (Nokta Atışı Georeferencing):** Kroki, kadastro paftası veya elle çizilmiş planlar için kusursuz matematiksel çözüm:
     - 1. Adım: Fotoğraftaki 1. belirgin noktaya dokunun (ör. bina köşesi).
     - 2. Adım: Uydudaki aynı 1. noktaya dokunun.
     - 3. Adım: Fotoğraftaki 2. belirgin noktaya dokunun.
     - 4. Adım: Uydudaki aynı 2. noktaya dokunun.
     - Sistem iki vektörün açı farkını, ölçek katsayısını ve ötelemesini hesaplayarak fotoğrafı milimetrik sıfır kaymayla uydu zeminine kenetler!
- Manuel araçlar (Taşı, Döndür, Ölçekle, Köşe Bük) ve parlaklık/kontrast/filtre ayarları ile ince ayar yapılabilir. **"Sabitle"** dendiğinde yerel **IndexedDB** veritabanına kalıcı olarak kaydedilir.

### 8. 💾 Yedekleme: Dışa ve İçe Aktarma (.ugpspro)
- Tüm çizimler, işaretlenen hedefler ve eklenen fotoğraflar tek tıkla **`.ugpspro`** (JSON) dosyası olarak cihaza indirilebilir.
- Başka bir cihazda veya tarayıcıda **"İçe Aktar"** denilerek aynı çalışma alanına kalınan yerden devam edilebilir.

---

## 🚀 Yerel Olarak Çalıştırma (XAMPP)

Proje `C:\xampp\htdocs\Yeni klasör` klasöründe bulunmaktadır.

1. **XAMPP Control Panel**'i açın ve **Apache** servisini başlatın (`Start`).
2. Bilgisayarınızın tarayıcısından şu adresi açın:
   ```
   http://localhost/Yeni%20klasör/index.html
   ```
3. Açılan şık karşılama portalında sensörlerin hazır olduğunu görün ve **"Uydu Sayfasına Git"** butonuna basarak haritayı başlatın.

---

## 📱 Cep Telefonunda Gerçek GPS ve Pusula ile Test Etme

> [!IMPORTANT]
> Mobil tarayıcılarda (Chrome, Safari) **yüksek hassasiyetli GPS ve pusula/oryantasyon sensörleri güvenlik gereği yalnızca HTTPS bağlantılarda çalışır**.

### Yöntem A: Netlify Drop ile Ücretsiz HTTPS (En Kolay)
1. Bilgisayarınızdan **[app.netlify.com/drop](https://app.netlify.com/drop)** adresini açın.
2. `C:\xampp\htdocs\Yeni klasör` klasörünü doğrudan tarayıcı penceresine sürükleyip bırakın.
3. Netlify size anında ücretsiz bir `https://xxxxx.netlify.app` adresi verir.
4. Bu bağlantıyı telefonunuzun kamerasından veya Chrome/Safari tarayıcısından açın.
5. Konum izni verin; Anti-Drift filtreli canlı konumunuz ve yön okunuz ekrana gelecektir.

### Yöntem B: Yerel Ağda ngrok ile HTTPS Tüneli
Bilgisayarınızda komut satırından:
```powershell
ngrok http 80
```
ngrok'un sağladığı `https://xxxx.ngrok-free.app/Yeni%20klasör/index.html` linkini telefonda açabilirsiniz.

---

## 📋 Saha Test Kontrol Listesi

- [ ] **Açılış Portalı:** `index.html` açılıyor, "Uydu Sayfasına Git" butonu `app.html` sayfasına geçiyor mu?
- [ ] **Uydu Seçimi:** Üstteki "Google Uydu (Maxar)" butonuna basınca 8 harita katmanı açıklamalarıyla listeleniyor ve seçim sorunsuz yükleniyor mu?
- [ ] **Anti-Drift Filtresi:** Telefon masada sabitken GPS rozeti `🟢 Sabit / Filtrelendi` yazıyor ve mavi nokta zıplamadan sabit kalıyor mu?
- [ ] **Hedef Koordinat:** Nişangah butonuna basıp bir enlem/boylam girildiğinde veya haritaya tıklandığında hedef çizgisi ve mesafe şeridi çıkıyor mu?
- [ ] **Varış Sinyali:** Hedefe 3.5 metre mesafe içine girip 3 saniye durulduğunda sesli melodi ve titreşim tetikleniyor mu?
- [ ] **Bakış Yönü & Döndürme:** Sağ alttaki pusulaya dokunup "180° Ters Çevir" veya "Canlı Pusula Takibi" ile uydu görüntüsünü çevirebiliyor, pusulaya basınca tek tıkla Kuzeye sıfırlayabiliyor musunuz?
- [ ] **Çizim:** Kalem butonuna basıp haritada bir bölge veya patika çizip kaydedebiliyor musunuz?
- [ ] **Fotoğraf & Puzzle Hizalama:** "+" butonundan resim ekleyip Auto-Snap, EXIF GPS veya 2 Noktalı Kilit ile uyduya kilitliyor ve sabitleyebiliyor musunuz?
- [ ] **Yedek:** "Dışa Aktar" ile `.ugpspro` dosyası iniyor, "İçe Aktar" ile geri yükleniyor mu?
