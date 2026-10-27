# String (Karakter Dizisi) Veri Tipi

String veri tipi, metinleri hafızada tutar. TIA Portal'da standart bir String 254 karakter boyutundadır (Maksimum boyut ve o anki mevcut uzunluk için ekstra 2 byte daha harcar).


# Hayati String Fonksiyonları

Metinleri bölmek ve incelemek için SCL'de gömülü fonksiyonlar kullanırız.

**LEN (Length)**: Metnin kaç karakterden oluştuğunu sayar. (Örn: `LEN('PLC')` sonucu 3'tür).

**FIND**: Bir metnin içinde belirli bir karakteri arar ve kaçıncı sırada olduğunu (Integer) döndürür. (Örn: `FIND(IN1:='A-B', IN2:='-')` sonucu 2'dir).

**LEFT / RIGHT**: Metnin en solundan veya en sağından istediğiniz sayıda karakteri kesip almanızı sağlar.

**MID**: Metnin ortasından (belirli bir başlangıç noktasından itibaren) parça almanızı sağlar.


# Tip Dönüşümü (String to Int)

Eğer barkoddan `'4598'` gibi bir sayı metni kopardıysak, bunu matematiksel işlemlerde kullanabilmek için Integer'a çevirmeliyiz:

`#q_iDeger := STRING_TO_INT(#t_sKoparılanMetin);`


# EX: BARCODE PARSER

// 1. UZUNLUK VE TİRE (-) POZİSYONUNU BUL
// Gelen metnin uzunluğunu ve ayırıcı karakterin (-) nerede olduğunu hesaplıyoruz.
#t_iTotalLength := LEN(#i_sRawBarcode);
#t_iDashPos := FIND(IN1 := #i_sRawBarcode, IN2 := '-');

// 2. BARKOD GEÇERLİLİK KONTROLÜ
// Eğer metnin içinde '-' yoksa (FIND 0 döndürürse) barkod formata uymuyordur!
IF #t_iDashPos = 0 THEN
    #q_bError := TRUE;
    #q_sPrefix := 'ERR';
    #q_iProductCode := 0;
    
ELSE
    // Barkod geçerli, hatayı temizle.
    #q_bError := FALSE;
    
    // 3. SOL KISMI (PREFIX) AL
    // Tireden bir önceki karaktere kadar olan kısmı en soldan kes.
    // Örn: 'PT-8520' için tire 3. sıradadır. Biz soldan (3-1) = 2 karakter alacağız ('PT').
    #q_sPrefix := LEFT(IN := #i_sRawBarcode, L := (#t_iDashPos - 1));
    
    // 4. SAĞ KISMI (NUMARA) AL
    // Tireden sonraki kısmı bulmak için toplam uzunluktan tirenin pozisyonunu çıkarıyoruz.
    // L = 7 - 3 = 4 karakteri en sağdan kes.
    #t_sNumberPart := RIGHT(IN := #i_sRawBarcode, L := (#t_iTotalLength - #t_iDashPos));
    
    // 5. METNİ SAYIYA ÇEVİR
    // Kopardığımız '8520' metnini gerçek bir INT sayısına dönüştür.
    #q_iProductCode := STRING_TO_INT(#t_sNumberPart);
    
END_IF;


# Good-to-know:

1- String operasyonları PLC işlemcisinin en çok yorulduğu işlemlerdir. Tek bir metin kopyalama veya arama işlemi yüzlerce makine komutu üretir. String parçalama işlemleri yalnızca barkod okunduğu an (Yükselen Kenar - R_TRIG) tek bir PLC taraması boyunca çalıştırılmalı, diğer turlarda atlanmalıdır.

2- Eğer barkod eksik veya hatalı basılmışsa (örneğin ayırıcı olan _ işareti yoksa), FIND komutu 0 değerini üretir.0 değerini doğrudan MID komutunun parametresine verirseniz, PLC geçersiz adres okumaya çalışır ve işlemci alarm günlüğüne "Length of string exceeded" hatası kaydeder. MID çalıştırmadan önce FIND > 0 kontrolü yapmak zorunludur.

3- Kameradan gelen REAL bir sayıda (örneğin "12.5") ondalık ayırıcı nokta (.) iken, TIA Portal sistem bölge ayarlarınız virgül (,) bekliyor olabilir. VAL_STRG komutunun ondalık basamak ayarı format parametresiyle (FORMAT := 16#0001 vb.) sabitlenmelidir; aksi takdirde 12.5 sayısı PLC'de 125 veya 0 olarak okunur.