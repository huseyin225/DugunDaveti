# Firebase Kurulum & Bağlantı Rehberi

Bu proje tamamen ön uç (frontend-only) olarak geliştirilmiştir; herhangi bir arka uç (backend) sunucuya ihtiyaç duymadan doğrudan Google Firebase Firestore veritabanına bağlanır.

---

## 1. Firebase Projesi Oluşturma (2 Dakika)

1. [Firebase Console](https://console.firebase.google.com/) adresine gidin ve Google hesabınızla giriş yapın.
2. **"Proje Ekle" (Add Project)** butonuna tıklayın.
3. Projenize bir ad verin (Örn: `dugun-daveti`) ve devam edin. (Google Analytics isteğe bağlıdır, kapatabilirsiniz).
4. Projeniz oluştuktan sonra sol menüden **"Build" > "Firestore Database"** bölümüne tıklayın.
5. **"Create database" (Veritabanı oluştur)** butonuna tıklayın.
6. Konum olarak `eur3 (europe-west)` veya size en yakın konumu seçin.

---

## 2. Firestore Güvenlik Kuralları (Security Rules)

Firebase Console > **Firestore Database** > **"Rules" (Kurallar)** sekmesine gidin, aşağıdaki kuralları yapıştırıp **"Publish" (Yayınla)** butonuna tıklayın:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 1. L.C.V. / Rezervasyonlar
    match /reservations/{reservationId} {
      // Davetlilerin rezervasyon yapabilmesi için doğrudan yazmaya izin ver
      allow create: if true;
      // Güvenlik: Misafirlerin başkalarının telefon ve isim bilgilerini siteden okumasını engelle
      allow read, update, delete: if false;
    }

    // 2. Fotoğraf Yüklemeleri Galerisi (upload.html)
    match /wedding_photos/{photoId} {
      // Misafirlerin fotoğraf bilgilerini kaydetmesine izin ver
      allow create: if true;
      // Misafirlerin yüklenen fotoğrafları galeride görüntülemesine izin ver
      allow read: if true;
      allow update, delete: if false;
    }
  }
}
```

---

## 3. Firebase Storage (Fotoğraf Alanı) Kurulumu & Kuralları

`upload.html` sayfasından yüklenen fotoğrafların Firebase Storage'a kaydedilebilmesi için:

1. Firebase Console sol menüsünden **"Build" > "Storage"** sekmesine tıklayın.
2. **"Get Started" (Başlayın)** butonuna tıklayın.
3. Konumu seçip Storage'ı etkinleştirin.
4. **"Rules" (Kurallar)** sekmesine gidin ve aşağıdaki kuralı yapıştırıp **"Publish"** deyin:

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /wedding-photos/{allPaths=**} {
      // Misafirlerin fotoğraf yüklemesine ve görüntülemesine izin ver
      allow read, write: if true;
    }
  }
}
```

---

## 4. Yapılandırma Durumu (Tamamlandı)

Verdiğiniz Firebase anahtarları (`AIzaSyCeKa...`) projenin tüm sayfalarına entegre edilmiştir:
- **[`index.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/index.html)**: Ana Davetiye & L.C.V. Rezervasyon
- **[`participants.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/participants.html)**: Katılımcı Yönetimi & İsim/Telefon Listesi & Excel/CSV İndirme
- **[`upload.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/upload.html)**: Fotoğraf Yükleme Paneli
- **[`gallery.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/gallery.html)**: Canlı Düğün Fotoğraf Galerisi & Lightbox

Yukarıdaki 2. ve 3. adımlardaki Firebase güvenlik kurallarını Firebase konsolunuzda onaylayıp **Publish** ettikten sonra:
- **`index.html`** üzerindeki L.C.V. formu hem yerel depolamaya anında yazar hem de arka planda doğrudan **`reservations`** koleksiyonuna aktarır (kullanıcıyı asla dondurmaz veya bekletmez).
- **`participants.html`** sayfasında tüm katılımcıları kişi sayıları ve telefonlarıyla canlı olarak görebilir, filtreleyebilir ve Excel/CSV olarak indirebilirsiniz.
- **`upload.html`** sayfasından yüklenen tüm fotoğraflar **Firebase Storage** ve **`wedding_photos`** koleksiyonuna yazılacaktır.
- **`gallery.html`** sayfası yüklenen fotoğrafları anlık (`onSnapshot`) olarak sergileyecektir.

---

## 5. Gelen Rezervasyonları ve Fotoğrafları Görme

1. [Firebase Console](https://console.firebase.google.com/) > Projeniz:
   - **Firestore Database > `reservations`**: Rezervasyon yapan misafirlerin adı, telefon numarası ve kişi sayıları.
   - **Firestore Database > `wedding_photos`**: Yüklenen fotoğrafların açıklamaları, yükleyen kişi ve URL'leri.
   - **Storage > `wedding-photos`**: Yüklenen orijinal yüksek çözünürlüklü fotoğraflar.
2. Web Üzerinden:
   - **[`participants.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/participants.html)**: Katılımcıların tümünü liste halinde görüntüleme, manuel katılımcı ekleme/silme ve Excel/CSV dışa aktarma.
   - **[`gallery.html`](file:///c:/Users/husey/OneDrive/Desktop/ba%C4%9F%C4%B1ms%C4%B1z/d%C3%BC%C4%9F%C3%BCndaveti/gallery.html)**: Misafirlerin yüklediği fotoğrafları canlı görme, seçip indirme veya silme.
