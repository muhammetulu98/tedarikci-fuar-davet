# Fuar Davet – Tedarikçi Ekranı: Çalışma Mantığı ve Analiz Dokümanı

Bu doküman, `tedarikci-fuar-davet` prototipindeki ekranın nasıl çalışması gerektiğini yazılımcıya anlatmak için hazırlanmıştır. Prototip Nuxt 4 ile yapılmıştır, **backend yoktur**: tüm veri tarayıcıda örnek olarak tutulur ve sayfa yenilenince sıfırlanır. Gerçek sistemde bu davranışların sunucu tarafında (ERP/veritabanı) karşılanması gerekir.

Kaynak kod: `app/pages/index.vue` (ekran ve tüm mantık), `app/components/DateRange.vue` (tarih aralığı kutusu).

---

## 1. Amaç

Müşteri fuar davet ekranının **tedarikçiler için** olan karşılığıdır. Fuara davet edilecek tedarikçiler seçilir, fuar koşulları (vade, iskonto, prim, konaklama, alan, masraf) girilir ve kayıt **yönetim onayına** gönderilir. Onay sonucunda tedarikçi fuara davet edilmiş sayılır.

Müşteri ekranından farkı: finans/cari onay adımları yoktur; yalnızca yönetim onayı vardır.

---

## 2. Onay Akışı (durum makinesi)

Bir kaydın 5 durumu vardır:

| Kod | Ekranda görünen | Anlamı |
|---|---|---|
| `aday` | Aday | Tedarikçi eklendi, bilgiler giriliyor. Düzenlenebilir. |
| `gm` | Yönetim Onayında | Genel Müdür'ün onayını bekliyor. |
| `ust` | Üst Yönetim Onayında | Genel Müdür lüzum görüp üst yönetime iletti. |
| `ok` | Onaylandı | Davet kesinleşti. |
| `red` | Reddedildi | Yönetim reddetti (kimin reddettiği satırda görünür). |

### Geçişler

| Kimden | Eylem | Kime | Kim yapar (öneri) |
|---|---|---|---|
| `aday` | Yönetim Onayına Gönder | `gm` | Admin / satın alma |
| `gm` | Onayla | `ok` | Genel Müdür |
| `gm` | Reddet | `red` | Genel Müdür |
| `gm` | **Üst Yönetime Gönder** | `ust` | Genel Müdür (lüzum gördüğünde) |
| `ust` | Onayla | `ok` | Üst Yönetim |
| `ust` | Reddet | `red` | Üst Yönetim |
| `gm`, `ust` | Geri Çek | `aday` | Gönderen kişi |
| `ok`, `red` | Sıfırla | `aday` | Yetkili kullanıcı |
| `aday` | Sil | kayıt silinir | Admin |

Önemli kurallar:

- **Sadece `aday` durumundaki kayıtlar düzenlenebilir ve silinebilir.** Diğer durumlarda kayıt paneli açılır ama tüm alanlar kilitlidir (salt okunur).
- Genel Müdür'ün üst yönetime göndermesi **zorunlu değildir**; kendi yetkisiyle direkt onaylayabilir ya da reddedebilir.
- Her geçişte **onay geçmişine** bir satır (tarih-saat + açıklama) eklenir. Prototipte kullanıcı adı yoktur; gerçek sistemde işlemi yapan kullanıcı da kaydedilmelidir.
- Prototipte rol/yetki kontrolü yoktur, herkes her butonu görebilir. Gerçek sistemde butonlar role göre açılıp kapanmalıdır (bkz. Açık Noktalar).

---

## 3. Ekran Yapısı

### 3.1 Üst alan
- Başlık ve açıklama.
- Sağ üst butonlar: **Fuar Masraf Tablosu** (bkz. 6), **Excel'e Aktar**, **Excel'den Karar Aktar** (prototipte yalnızca bilgi mesajı gösterir, gerçek işlevi yoktur).

### 3.2 Aşama kartları (2 adet)
- **01 Aday Listesi:** tüm kayıtların sayısı.
- **02 Yönetim Onayı:** durumu `gm` veya `ust` olan kayıtların sayısı.
- Karta tıklanınca liste o aşamaya göre filtrelenir; tekrar tıklanınca filtre kalkar.

### 3.3 Filtre paneli
- **Proje / Fuar:** açılır liste.
- **Tedarikçi:** aranabilir seçim kutusu. Ad ya da cari koda göre arar (Türkçe karakter duyarlı), en fazla 50 sonuç gösterir, **daha önce aday eklenmiş tedarikçileri listede göstermez**.
- **Aday Ekle:** seçilen tedarikçiyi `aday` durumunda listeye ekler ve kayıt panelini açar.
- **Durum** filtresi ve **Ara** kutusu (tedarikçi adı, cari kod, sorumlu).

### 3.4 Araç çubuğu (toplu işlemler)
Satırların başındaki kutularla seçim yapılır. Butonlar, **seçili tüm satırların** durumu uygunsa aktif olur:

| Buton | Aktif olma şartı |
|---|---|
| Yönetim Onayına Gönder | hepsi `aday` |
| Onayla / Reddet | hepsi `gm` veya `ust` |
| Üst Yönetime Gönder | hepsi `gm` |
| Sil | hepsi `aday` (silmeden önce onay sorulur) |
| Geri Çek | hepsi `gm` veya `ust` |
| Sıfırla | hepsi `ok` veya `red` |

### 3.5 Liste kolonları
Tedarikçi (ad + cari kod), Durum, Sorumlu, Bakiye, Vade, Alış İskontosu (örn. `%10 + %3`), Satış İskontosu, Hedef Satış Cirosu, Son Fuar. Satır rengi duruma göre değişir (sarı: yönetim onayında, mor: üst yönetim, yeşil: onaylı, kırmızı: reddedilmiş). Satırda ✎ (kayıt paneli) ve yalnızca adaylarda 🗑 (sil) butonu vardır.

---

## 4. Kayıt Paneli (sağdan açılan form)

Sıra ve alanlar:

1. **Vade**
   - Seçim: *Gün* ya da *Nokta tarih* (ikisinden biri). Gün ise sayı, nokta tarih ise tarih girilir.
   - **Vade Açıklaması:** kısa metin, en fazla 120 karakter.
2. **Fuar Alış İskontoları**
   - **İskonto Grubu:** A0 / A1 / A2 listesi.
   - **İskonto 1–4:** dört ayrı yüzde kutusu.
   - **Nakliye:** Hariç / Dahil (liste). **Nakliye Şekli:** Yerinden / Timon Depo (liste). Yan yana durur, birbirinden bağımsızdır.
3. **Satış İskontosu ve Hedef**
   - Fuar Satış İskontosu (%), Hedef Satış Cirosu (₺).
4. **Primler**
   - **Personel Primi, Ciro Primi, Pazarlama Primi:** her biri için *Baremli* kutusu vardır.
     - Baremli değilse: tek prim oranı (%).
     - Baremliyse: satırlar halinde *alt sınır (₺) – üst sınır (₺) – prim (%)*; "+ Barem Ekle" ile satır eklenir, ✕ ile silinir. Yeni satırın alt sınırı önceki satırın üst sınırıyla otomatik dolar. En az 1 satır kalır.
   - **Ciro Hedefine Göre Altın Ödülü:** satırlar halinde *ciro (₺) – adet – altın türü* (Gram / Çeyrek / Yarım / Tam Altın). Örn. "X ₺ ciroya 10 gram altın".
5. **Sipariş Sorumlusu:** ad soyad, e-posta, telefon.
6. **Konaklama**
   - "+ Oda Ekle" / "+ Bir Oda Daha Ekle" ile oda eklenir; her oda için:
     - **Oda tipi** (SNG, DBL, DBL+1, TRPL, TRPL+1, FAM; listede fiyatı yazar),
     - **Giriş – Çıkış tarihi** (tek kutuda; tıklayınca iki tarih seçici açılır; giriş çıkıştan sonra olamaz),
     - **Kişiler:** "+ Kişi Ekle" ile istenen sayıda; her kişi için *ad soyad, TC kimlik no (11 hane), doğum tarihi*. Odada en az 1 kişi kalır.
   - Oda başına bilgi satırı: `DBL · €324 × 4 gece = €1.296`.
7. **Alan Bilgileri:** Duvar (mt), Masa sayısı (adet), Raf (mt).
8. **Diğer**
   - Fuar Kredi Kartı Katılım Bedeli (₺), Fuar Fatura Katılım Bedeli (₺).
   - **Gerçekleşen Ciro:** **salt okunur, otomatik gelir, manuel giriş yoktur.** Gerçek sistemde ERP'deki tedarikçinin fuar dönemi cirosundan beslenmelidir.
   - Cirodan Fuar Katılım Primi (%).
   - Açıklama (serbest metin).
9. **Masraf kutusu** (bkz. 5).
10. **Onay Geçmişi** (en yeni üstte).

Alt butonlar: **Kapat**, **Kaydet**, **Kaydet ve Yönetim Onayına Gönder** (kaydeder ve durumu `gm` yapar). Aday dışında yalnızca *Kapat* görünür.

---

## 5. Hesaplamalar

Tümü ekranda anlık hesaplanır.

```
gece sayısı      = max(0, çıkış tarihi − giriş tarihi)      (tarih yoksa 0)
oda tutarı (€)   = oda tipi gecelik fiyatı × gece sayısı
konaklama (€)    = tüm odaların tutarı toplamı
konaklama (₺)    = konaklama (€) × EUR kuru                  (kur sabit 50)
alan masrafı (₺) = duvar mt × duvar mt fiyatı
                 + raf mt   × raf mt fiyatı
                 + masa adedi × masa fiyatı
katılım (₺)      = kredi kartı katılım bedeli + fatura katılım bedeli

TOPLAM MASRAF (₺) = katılım + konaklama (₺) + alan masrafı
```

Masraf kutusu rengi: toplam masraf veya konaklama tutarı > 0 ise **yeşil**, hiç veri yoksa **gri**. (Hedef cirosuna oran gösterilmez; tedarikçi için gerek yok denildi.)

---

## 6. Fuar Masraf Gider Tablosu (ortak parametreler)

Ekranın sağ üstündeki **Fuar Masraf Tablosu** butonundan açılan panel. Buradaki fiyatlar **tüm adaylar için ortaktır**; değiştirilince tüm masraf hesapları buna göre güncellenir.

| Kalem | Birim | Prototipteki değer |
|---|---|---|
| Oda fiyatları (gecelik) | € | SNG 243, DBL 324, DBL+1 405, TRPL 454, TRPL+1 535, FAM 648 |
| Duvar metretül fiyatı | ₺ / mt | 0 (girilecek) |
| Raf metretül fiyatı | ₺ / mt | 0 (girilecek) |
| Masa fiyatı | ₺ / adet | 0 (girilecek) |

Gerçek sistemde bu tablo bir **parametre tablosu** olarak saklanmalı, yetkili kullanıcı güncellemeli ve **fuar bazında** tutulması değerlendirilmelidir (fiyat her fuarda değişebilir).

---

## 7. Veri Modeli (önerilen)

**Fuar davet kaydı (`FuarDavetTedarikci`)**

| Alan | Tip | Not |
|---|---|---|
| id, fuarId, cariKod | | Bir tedarikçi bir fuara **yalnızca bir kez** eklenebilir |
| durum | enum | `aday, gm, ust, ok, red` |
| redEden | metin | "Yönetim" vb. (kim reddetti) |
| sorumlu | metin | Cari kartından gelir (prototipte hepsi Eyüp Ömer Yılmaz) |
| bakiye, sonFuarTutari | ondalık | ERP'den gelir, düzenlenmez |
| gerceklesenCiro | ondalık | ERP'den **otomatik**, düzenlenmez |
| vadeTip, vadeGun, vadeTarih, vadeAciklama | | Tip `gun` ya da `tarih` |
| iskontoGrubu | metin | A0 / A1 / A2 |
| alisIskonto1..4 | ondalık % | |
| satisIskonto | ondalık % | |
| nakliye, nakliyeSekli | enum | `dahil/haric`, `yerinden/timon` |
| hedefSatisCirosu | ondalık | |
| personelPrim, ciroPrim, pazarlamaPrim | nesne | `{baremli, sabitOran, baremler[]}` |
| katilimPrimOrani | ondalık % | Cirodan fuar katılım primi |
| altinOdulleri[] | liste | `{ciro, adet, tur}` |
| siparisSorumlusu | nesne | `{adSoyad, mail, telefon}` |
| odalar[] | liste | `{tip, giris, cikis, kisiler[]}` |
| kisiler[] (oda içinde) | liste | `{adSoyad, tc, dogumTarihi}` |
| duvarMt, masaSayisi, rafMt | sayı | |
| krediKartiKatilimBedeli, faturaKatilimBedeli | ondalık ₺ | |
| aciklama | metin | |

**Barem satırı:** `{altSinir, ustSinir, primOrani}`

**Onay geçmişi (`FuarDavetLog`):** kayıtId, tarih-saat, kullanıcı, eylem/açıklama.

**Masraf gider parametreleri:** fuarId, odaTipi + euroFiyat, duvarMtFiyat, rafMtFiyat, masaFiyat.

---

## 8. Açık Noktalar / Yazılımcıyla Netleştirilecekler

Prototipte **varsayım** olarak yapılanlar veya henüz karara bağlanmamış konular:

1. **Roller ve yetki:** Kim aday ekler/gönderir, kim Genel Müdür, kim Üst Yönetim? Butonlar role göre açılmalı; Genel Müdür yalnızca `gm` durumundakileri, üst yönetim `ust` durumundakileri işlemeli. Geri çekme ve sıfırlama yetkisi kimde?
2. **Zorunlu alanlar:** Onaya göndermeden önce hangi alanlar zorunlu (örn. vade, iskonto, hedef ciro, sipariş sorumlusu)? Prototipte hiçbiri zorunlu değil.
3. **Oda fiyatı birimi:** Oda fiyatları **gecelik** kabul edildi. Konaklama süresi boyunca sabit tutarsa gece çarpanı kaldırılmalı.
4. **Duvar / raf / masa fiyatı para birimi:** ₺ kabul edildi, tutarlar henüz girilmedi.
5. **EUR kuru:** Şimdilik **sabit 50 ₺**, değiştirilemez. Gerçekte günlük/dönemsel kur mu kullanılacak?
6. **Altın ödülü:** Ödül tutarı ekranda hesaplanmaz, yalnızca koşul olarak saklanır. Gram altın için gram miktarı mı (adet = gram), yoksa başka bir tanım mı olacak?
7. **Prim hesabı:** Barem sınırlarının nasıl uygulanacağı (dilimli/kademeli mi, ulaşılan baremin oranı tüm ciroya mı uygulanır) ve `gerçekleşen ciro` üzerinden primin ne zaman ve nasıl hesaplanacağı bu ekranın kapsamında değil; ayrıca tanımlanmalı. Barem satırlarında çakışma/boşluk kontrolü de yok.
8. **Doğrulamalar:** TC kimlik numarası doğrulaması (11 hane + algoritma), e-posta ve telefon formatı, tarih mantığı (giriş < çıkış — şu an tarih seçicide kısıtlı).
9. **Red gerekçesi:** Reddederken gerekçe alanı yok; eklenmesi önerilir.
10. **Bildirim:** Onaya gönderilince / karar verilince ilgili kişiye e-posta ya da uygulama bildirimi gidecek mi?
11. **Excel'e Aktar / Excel'den Karar Aktar:** Prototipte yalnızca mesaj gösterir. Gerçek işlev (dosya formatı, hangi kolonlar, toplu karar nasıl işlenecek) tanımlanmalı.
12. **Masraf toplamına giren kalemler:** Şu an kredi kartı + fatura katılım bedeli + konaklama + alan (duvar/raf/masa). Nakliye bedeli ve primler toplama **girmez**; girmesi gerekiyorsa belirtilmeli.
13. **Fuar Masraf Tablosu kapsamı:** Tüm fuarlar için tek tablo mu, fuar bazında mı? Değişiklik yapılınca daha önce onaylanmış kayıtların tutarı değişmemeli (fiyatın kayıt anında dondurulması önerilir).
14. **Cari verileri:** Bakiye, sorumlu ve son fuar tutarı ERP'den (Timon) okunmalı; kayıt paneli bunları değiştirmez.

---

## 9. Prototipi Çalıştırma

```bash
cd ~/Desktop/tedarikci-fuar-davet
npm install      # ilk seferde
npx nuxt dev --port 3010
```

Tarayıcı: http://localhost:3010
