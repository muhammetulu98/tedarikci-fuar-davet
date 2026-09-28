# Fuar Davet – Tedarikçi Ekranı: Çalışma Mantığı ve Analiz Dokümanı

Bu doküman, `tedarikci-fuar-davet` prototipindeki ekranın nasıl çalışması gerektiğini yazılımcıya anlatmak için hazırlanmıştır. Prototip Nuxt 4 ile yapılmıştır, **backend yoktur**: tüm veri tarayıcıda örnek olarak tutulur ve sayfa yenilenince sıfırlanır. Gerçek sistemde bu davranışların sunucu tarafında (ERP/veritabanı) karşılanması gerekir.

Kaynak kod: `app/pages/index.vue` (ekran ve tüm mantık), `app/components/DateRange.vue` (tarih aralığı kutusu), `app/components/MoneyInput.vue` (tutar kutusu).

---

## 1. Amaç

Müşteri fuar davet ekranının **tedarikçiler için** olan karşılığıdır. Fuara davet edilecek tedarikçiler seçilir, fuar koşulları (vade, iskonto, sevkiyat, prim, konaklama, alan, masraf) girilir ve kayıt **yönetim onayına** gönderilir. Onay sonucunda tedarikçi fuara davet edilmiş sayılır.

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
- **Proje / Fuar:** açılır liste. (Fuar yılı ve fuar no alanları kullanılmaz.)
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
Tedarikçi (ad + cari kod), **Markalar**, **Alt Kategoriler**, Durum, Sorumlu, Vade, Alış İskontosu (örn. `%10 + %3`), Satış İskontosu, Hedef Satış Cirosu. Bakiye ve Son Fuar kolonları listede **yoktur**. Markalar ve Alt Kategoriler, carinin birden çok değer taşıyabildiği alanlardır; liste içinde etiket (chip) olarak alt alta sarılarak gösterilir. Arama kutusu marka ve alt kategori adına göre de arar. Satır rengi duruma göre değişir (sarı: yönetim onayında, mor: üst yönetim, yeşil: onaylı, kırmızı: reddedilmiş). Satırda ✎ (kayıt paneli) ve yalnızca adaylarda 🗑 (sil) butonu vardır.

#### Liste verilerinin kaynağı (veritabanı notları)

| Kolon | Kaynak | Not |
|---|---|---|
| **Sorumlu** | `cari_hesaplar` tablosu | Carinin sorumlusu buradan alınır. Prototipte tüm kayıtlarda örnek olarak "Eyüp Ömer Yılmaz" yazılıdır. |
| **Alt Kategoriler** | `stoklar` tablosu | Carinin **kendisine ait ürünlerdeki** alt grupların tekrarsız (distinct) listesi. Bir cari birden çok alt gruba sahip olabilir. |
| **Markalar** | `stoklar` tablosu | Carinin **kendisine ait ürünlerdeki** markaların tekrarsız (distinct) listesi. Bir cari birden çok markaya sahip olabilir. |

Marka ve alt grup, `stoklar` tablosunda ürünün bağlı olduğu cariye göre (ürünün tedarikçi/cari kodu ile) filtrelenerek toplanır. Bu bilgiler kayıt panelinde düzenlenmez, yalnızca listede gösterilir. Prototipte marka ve alt kategori değerleri örnek (uydurma) veridir.

---

## 4. Kayıt Paneli (sağdan açılan form)

Bölümler ve alanlar (ekrandaki sırayla):

Bölüm başlıkları numaralı ve her biri farklı renktedir (yalnızca görsel ayrım içindir).

### 1 · Alış Şartları
Vade, alış iskontoları ve sevkiyat tek bölümde toplanır.
- **Fuar Alış Vadesi:** *Gün* ya da *Nokta tarih* (ikisinden biri). Gün ise sayı, nokta tarih ise tarih girilir.
- **Vade Açıklaması:** kısa serbest metin, en fazla 120 karakter.
- **İskonto 1–4:** dört ayrı yüzde kutusu.
- **Sevkiyat Baremleri:** satırlar halinde; her satırda
  - **Birim:** `Baremsiz`, `Koli` ya da `Palet`.
  - **Alt sınır:** birim Koli ya da Palet ise koli adedi / palet sayısı olarak girilir. **Üst sınır yoktur**, yalnızca alt sınır tutulur.
  - **Baremsiz** seçilirse alt sınır kutusu yerine tek bir **tutar (₺)** kutusu açılır.
  - **Teslim yeri:** `Timon Depo` ya da `Yerinden`.
  - **Nakliye:** `Nakliye Hariç` ya da `Nakliye Dahil`.
  - Teslim yeri ve nakliye her satırda ayrı seçilir; palet ise Timon depo, koli ise yerinden gibi eşleşmeler sabit değildir, **kullanıcı satır satır seçer**.
  - "+ Sevkiyat Baremi Ekle" ile satır eklenir, ✕ ile silinir. En az 1 satır kalır. Yeni satır varsayılan olarak boş (0) gelir.

### 2 · Satış İskontosu ve Hedef
- **Fuar Satış İskontosu (%)**, **İskonto Grubu** (A0 / A1 / A2 listesi), **Hedef Satış Cirosu (₺)**.

### 3 · Primler
- **Personel Primi**, **Sene Sonu Primi** ve **Ciro Primi:** üçü de aynı mantıkla çalışır; her biri için *Baremli* kutusu vardır.
  - Baremli değilse: tek prim oranı (%).
  - Baremliyse: satırlar halinde *alt sınır (₺) – üst sınır (₺) – prim (%)*; "+ Barem Ekle" ile satır eklenir, ✕ ile silinir. Yeni satırın alt sınırı önceki satırın üst sınırıyla otomatik dolar. En az 1 satır kalır.
- **Bozuk İade Bütçesi:** baremsiz, tek bir yüzde (%) oranı. Prim değil, ayrı bir bütçe oranıdır; baremli prim listesinin dışında durur.
- **Ciro Hedefine Göre Altın Ödülü:** satırlar halinde *ciro (₺) – adet – altın türü* (Gram / Çeyrek / Yarım / Tam Altın). Örn. "X ₺ ciroya 10 gram altın".
- Pazarlama primi alanı **kaldırılmıştır**.

### 4 · Tedarikçi Sipariş Sorumlusu
- Ad soyad, e-posta, telefon.

### 5 · Konaklama
- "+ Oda Ekle" / "+ Bir Oda Daha Ekle" ile oda eklenir; her oda için:
  - **Oda tipi** (SNG, DBL, DBL+1, TRPL, TRPL+1, FAM; listede fiyatı yazar),
  - **Giriş – Çıkış tarihi** (tek kutuda; tıklayınca iki tarih seçici açılır; giriş çıkıştan sonra olamaz),
  - **Kişiler:** "+ Kişi Ekle" ile istenen sayıda; her kişi için *ad soyad, TC kimlik no (11 hane), doğum tarihi*. Odada en az 1 kişi kalır.
- Oda başına bilgi satırı: `DBL · €324 × 4 gece = €1.296`.

### 6 · Alan Bilgileri
- Duvar (mt), Masa sayısı (adet), Raf (mt).

### Diğer
- **Fuar Kredi Kartı Katılım Bedeli (₺)** ve **Fuar Fatura Katılım Bedeli (₺)**.
- **Gerçekleşen Ciro:** **salt okunur, otomatik gelir, manuel giriş yoktur.** Gerçek sistemde ERP'deki tedarikçinin fuar dönemi cirosundan beslenmelidir.
- **Cirodan Fuar Katılım Primi (%):** tek oran. Gerçekleşen ciro ile çarpılıp masraftan düşülür (bkz. 5).
- **Açıklama:** serbest metin.

### Masraf kutusu
Panelin altında, Onay Geçmişi'nin üstünde. Toplam masrafı ve hesabın dökümünü gösterir (bkz. 5). **EUR Kuru** burada salt okunur görünür (sabit 50 ₺).

### Onay Geçmişi
En yeni işlem üstte.

Alt butonlar: **Kapat**, **Kaydet**, **Kaydet ve Yönetim Onayına Gönder** (kaydeder ve durumu `gm` yapar). Aday dışında yalnızca *Kapat* görünür.

### Tutar kutularının davranışı
Tüm ₺ / € tutar kutuları (hedef satış cirosu, katılım bedelleri, barem alt/üst sınırları, altın ödülündeki ciro, baremsiz sevkiyat tutarı, masraf tablosundaki fiyatlar) **otomatik ondalıklı biçimlenir**. Rakamlar sağdan dolar: `1` → `0,01`, `12` → `0,12`, `125075` → `1.250,75`. Nokta/virgül yazmaya gerek yoktur; eksi tutar girilemez. Adet, gün, mt, iskonto % ve prim % gibi alanlar düz sayı olarak kalır.

---

## 5. Hesaplamalar

Tümü ekranda anlık hesaplanır.

```
gece sayısı        = max(0, çıkış tarihi − giriş tarihi)         (tarih yoksa 0)
oda tutarı (€)     = oda tipi gecelik fiyatı × gece sayısı
konaklama (€)      = tüm odaların tutarı toplamı
konaklama (₺)      = konaklama (€) × EUR kuru                    (kur sabit 50)

duvar (₺)          = duvar mt × duvar metretül fiyatı
raf (₺)            = raf mt   × raf metretül fiyatı
masa (₺)           = masa adedi × masa fiyatı
                     (üç kalem birbirinden bağımsız hesaplanır ve ayrı gösterilir)

katılım primi (₺)  = gerçekleşen ciro × cirodan fuar katılım primi %
katılım bedelleri  = kredi kartı katılım bedeli + fatura katılım bedeli

TOPLAM MASRAF (₺)  = konaklama (₺) + duvar + raf + masa
                     − kredi kartı katılım bedeli
                     − fatura katılım bedeli
                     − katılım primi (₺)
```

Kredi kartı ve fatura katılım bedelleri masrafa **eklenmez, masraftan düşülür** (tedarikçiden alınan katkı olarak ele alınır). Bu yüzden toplam masraf **eksi** çıkabilir; bu, masrafın karşılandığı anlamına gelir.

Masraf kutusu rengi:

| Durum | Renk |
|---|---|
| Toplam masraf 0 ya da eksi (karşılanmış) | yeşil |
| Toplam masraf artıda (karşılanmamış) | turuncu |
| Hiç veri girilmemiş | gri |

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
| sorumlu | metin | `cari_hesaplar` tablosundan gelir, düzenlenmez |
| markalar[] | metin listesi | `stoklar` tablosunda carinin ürünlerindeki distinct markalar, düzenlenmez |
| altKategoriler[] | metin listesi | `stoklar` tablosunda carinin ürünlerindeki distinct alt gruplar, düzenlenmez |
| gerceklesenCiro | ondalık | ERP'den **otomatik**, düzenlenmez |
| vadeTip, vadeGun, vadeTarih, vadeAciklama | | Tip `gun` ya da `tarih` |
| alisIskonto1..4 | ondalık % | |
| sevkiyat[] | liste | `{birim, altSinir, tutar, teslimYeri, nakliye}` (aşağıya bakın) |
| satisIskonto | ondalık % | |
| iskontoGrubu | metin | A0 / A1 / A2 |
| hedefSatisCirosu | ondalık | |
| personelPrim, seneSonuPrim, ciroPrim | nesne | `{baremli, sabitOran, baremler[]}` |
| bozukIadeButcesiOrani | ondalık % | Baremsiz, tek oran |
| katilimPrimOrani | ondalık % | Cirodan fuar katılım primi |
| altinOdulleri[] | liste | `{ciro, adet, tur}` |
| siparisSorumlusu | nesne | `{adSoyad, mail, telefon}` |
| odalar[] | liste | `{tip, giris, cikis, kisiler[]}` |
| kisiler[] (oda içinde) | liste | `{adSoyad, tc, dogumTarihi}` |
| duvarMt, masaSayisi, rafMt | sayı | |
| krediKartiKatilimBedeli, faturaKatilimBedeli | ondalık ₺ | |
| aciklama | metin | |

**Sevkiyat satırı:** `birim` (`baremsiz` / `koli` / `palet`), `altSinir` (koli/palet için), `tutar` (yalnızca baremsizde, ₺), `teslimYeri` (`timon` / `yerinden`), `nakliye` (`haric` / `dahil`).

**Prim barem satırı:** `{altSinir, ustSinir, primOrani}`

**Onay geçmişi (`FuarDavetLog`):** kayıtId, tarih-saat, kullanıcı, eylem/açıklama.

**Masraf gider parametreleri:** fuarId, odaTipi + euroFiyat, duvarMtFiyat, rafMtFiyat, masaFiyat.

---

## 8. Açık Noktalar / Yazılımcıyla Netleştirilecekler

Prototipte **varsayım** olarak yapılanlar veya henüz karara bağlanmamış konular:

1. **Roller ve yetki:** Kim aday ekler/gönderir, kim Genel Müdür, kim Üst Yönetim? Butonlar role göre açılmalı; Genel Müdür yalnızca `gm` durumundakileri, üst yönetim `ust` durumundakileri işlemeli. Geri çekme ve sıfırlama yetkisi kimde?
2. **Zorunlu alanlar:** Onaya göndermeden önce hangi alanlar zorunlu (örn. vade, iskonto, hedef ciro, sipariş sorumlusu)? Prototipte hiçbiri zorunlu değil.
3. **Sevkiyat baremi:** Koli/palet alt sınırının anlamı (adet mi, bu adetten itibaren geçerli mi) ve barem neye göre işleyecek (koli adedi, palet sayısı, sipariş tutarı, ağırlık)? Baremsiz satırdaki **tutar** neyi ifade ediyor (sabit nakliye bedeli mi, sipariş başına mı)? Üst sınır olmadığı için satırların çakışma/sıralama kuralı da tanımlanmalı.
4. **Oda fiyatı birimi:** Oda fiyatları **gecelik** kabul edildi. Konaklama süresi boyunca sabit tutarsa gece çarpanı kaldırılmalı.
5. **Duvar / raf / masa fiyatı para birimi:** ₺ kabul edildi, tutarlar henüz girilmedi.
6. **EUR kuru:** Şimdilik **sabit 50 ₺**, değiştirilemez. Gerçekte günlük/dönemsel kur mu kullanılacak?
7. **Altın ödülü:** Ödül tutarı ekranda hesaplanmaz, yalnızca koşul olarak saklanır. Gram altın için gram miktarı mı (adet = gram), yoksa başka bir tanım mı olacak?
8. **Prim hesabı:** Personel, sene sonu ve ciro primi baremlerinin nasıl uygulanacağı (dilimli/kademeli mi, ulaşılan baremin oranı tüm ciroya mı uygulanır) bu ekranın kapsamında değil; ayrıca tanımlanmalı. Barem satırlarında çakışma/boşluk kontrolü de yok. Cirodan fuar katılım primi ise tek oran olup gerçekleşen ciro ile çarpılır. Sene sonu priminin hangi dönemin cirosuna göre ve ne zaman hesaplanacağı, **bozuk iade bütçesi** oranının hangi tutar üzerinden uygulanacağı (ciro mu, alış mı) tanımlanmalıdır.
9. **Doğrulamalar:** TC kimlik numarası doğrulaması (11 hane + algoritma), e-posta ve telefon formatı, tarih mantığı (giriş < çıkış — şu an tarih seçicide kısıtlı).
10. **Red gerekçesi:** Reddederken gerekçe alanı yok; eklenmesi önerilir.
11. **Bildirim:** Onaya gönderilince / karar verilince ilgili kişiye e-posta ya da uygulama bildirimi gidecek mi?
12. **Excel'e Aktar / Excel'den Karar Aktar:** Prototipte yalnızca mesaj gösterir. Gerçek işlev (dosya formatı, hangi kolonlar, toplu karar nasıl işlenecek) tanımlanmalı.
13. **Masraf formülü:** Kredi kartı ve fatura katılım bedelleri ile katılım primi masraftan düşülüyor; toplam eksi çıkabilir. Bu mantığın doğrulanması, nakliye bedelinin ve diğer primlerin toplama girip girmeyeceğinin netleştirilmesi gerekir.
14. **Fuar Masraf Tablosu kapsamı:** Tüm fuarlar için tek tablo mu, fuar bazında mı? Değişiklik yapılınca daha önce onaylanmış kayıtların tutarı değişmemeli (fiyatın kayıt anında dondurulması önerilir).
15. **Cari verileri:** Sorumlu `cari_hesaplar` tablosundan; marka ve alt kategori `stoklar` tablosundan (carinin kendi ürünlerinden) okunmalı; kayıt paneli bunları değiştirmez. Bakiye ve son fuar cirosu listede gösterilmez. Stok kartında marka ya da alt grubu boş olan ürünlerin nasıl ele alınacağı (listeden hariç mi, "Tanımsız" mı) netleştirilmeli.

---

## 9. Prototipi Çalıştırma

```bash
cd ~/Desktop/tedarikci-fuar-davet
npm install      # ilk seferde
npx nuxt dev --port 3010
```

Tarayıcı: http://localhost:3010
