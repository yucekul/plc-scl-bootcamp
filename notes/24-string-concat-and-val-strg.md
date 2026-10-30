# CONCAT (Concatenate) Nedir?

İki veya daha fazla String (Metin) ifadesini uç uca ekleyerek tek bir String haline getiren komuttur.
`CONCAT(IN1:='PLC', IN2:=' SCL')` ->  `'PLC SCL'`


# Sayıları Metne Çevirme (VAL_STRG vs INT_TO_STRING)

Klasik `INT_TO_STRING` komutu, çevirdiği sayının başına inatla `+` veya `-` işareti koyar (Örn: `2026` sayısını `'+2026'` yapar). Bu da barkodlarda istenmeyen bir durumdur.
Endüstri standardında temiz metinler üretmek için **`VAL_STRG` (Value to String)** fonksiyonu kullanılır. Bu fonksiyon:
Sayının başına artı/eksi konulup konulmayacağını (FORMAT),
Kaç hanelik yer kaplayacağını (SIZE),
Virgülden sonraki ondalık kısmını (PREC) detaylıca ayarlamanızı sağlar.


# EX: BARCODE GENERATOR

// 1. SAYILARI METNE ÇEVİRME VE FORMATLAMA (VAL_STRG)
// Yıl bilgisini formatla (İşaretsiz, ondalıksız).
VAL_STRG(IN     := #i_iYear, 
         SIZE   := 4,       // 4 hanelik yer ayır (Örn: 2026)
         PREC   := 0,       // Ondalık kısım yok
         FORMAT := 16#0000, // İşaretsiz (Signless) format
         P      := 1,       // Metne 1. karakterden itibaren yaz
         OUT    => #t_sYear);

// Seri numarasını formatla.
VAL_STRG(IN     := #i_iSerialNumber, 
         SIZE   := 4,       // En fazla 4 hane
         PREC   := 0,       // Ondalık kısım yok
         FORMAT := 16#0000, // İşaretsiz (Signless) format
         P      := 1, 
         OUT    => #t_sSerial);

// 2. METİNLERİ BİRLEŞTİRME (CONCAT)
// SCL'de CONCAT fonksiyonu aynı anda sadece 2 metni birleştirebilir.
// Bu yüzden işlemi C veya Python'daki gibi (A + "-" + B) iç içe (nested) yapıyoruz.

// Adım 1: "MOT" ile "-2026-" kısmını birleştir -> Sonuç: "MOT-2026-"
#t_sPart1 := CONCAT(IN1 := #i_sPrefix, 
                    IN2 := CONCAT(IN1 := '-', IN2 := CONCAT(IN1 := #t_sYear, IN2 := '-')));
                    
// Adım 2: "MOT-2026-" ile "852" (veya t_sSerial içindeki değer) kısmını birleştir. -> "MOT-2026-852"
#q_sFinalBarcode := CONCAT(IN1 := #t_sPart1, 
                           IN2 := #t_sSerial);


# Good-to-know:

1- VAL_STRG komutu bir sayıyı metne çevirirken, sayının eksi (-) veya artı (+) olma ihtimaline karşı baş tarafına (ilk karaktere) görünmez bir boşluk (Space) koyabilir. Barkod yazıcısı "PRD_ 105" şeklinde bir boşluk görürse barkodu basmayabilir veya sistem hatası verebilir.
VAL_STRG'den çıkan metnin içinde boşluk varsa (Örn: FIND(IN:=' ')), o boşluğu temizlemek (kırpmak) için 21. günde öğrendiğimiz RIGHT veya MID komutunu kullanarak saf metni filtrelemelisin.

2- Geçici birleştirme değişkenlerini her zaman ihtiyacın olan maksimum uzunlukta kısıtlamalısın (Örn: String[20], String[40] gibi).