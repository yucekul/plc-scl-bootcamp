# Word Veri Tipi Nedir?

Bir `Word` veri tipi, yan yana dizilmiş 16 adet kutucuktan (0'dan 15'e kadar bitler) oluşur.
Ekrana tek bir değişken olarak (Örn: `16#000F` Hex formatında) gider, ancak ekran bunu çözüp 16 ayrı alarm lambası yakabilir.


# SHL (Shift Left - Sola Kaydır)

Bir sayının bitlerini istenilen adım kadar sola iter.
`SHL(IN := 16#0001, N := 3)` -> `0001` (Yani 0. bit) sayısını 3 adım sola kaydırır. Sonuç `16#0008` (Yani 3. bit) olur.


# Bitwise OR (VEYA ile Maskeleme)

İki sayının bitlerini alt alta koyup mantıksal VEYA işlemine sokar. 
Mevcut bir Word paketinin içindeki eski alarmları bozmadan, yeni bir alarmı o pakete eklemek için `OR` işlemi kullanılır.
Formül: `MevcutPaket := MevcutPaket OR YeniAlarmMaskesi;`


# EX: ALARM PACKER

// 1. BAŞLANGIÇ: PAKETİ TEMİZLE
// İşleme başlamadan önce geçici Word'ün içindeki tüm bitleri 0 yapıyoruz (16#0000).
#t_wTempWord := 16#0000;

// 2. 16 BİT İÇİN TARAMA DÖNGÜSÜ (0'dan 15'e kadar)
FOR #t_i := 0 TO 15 DO
    
    // Eğer dizideki o anki alarm TRUE ise (Örn: 3. sıradaki alarm geldi)
    IF #i_aAlarms[#t_i] THEN
        
        // A. MASKE ÜRETME (SHL)
        // 16#0001 sayısını (Sadece 0. biti 1 olan sayı), t_i kadar sola kaydır.
        // Eğer t_i = 3 ise, 1 sayısını 3 kere sola kaydırıp "0000 0000 0000 1000" maskesi üretir.
        
        // B. PAKETE EKLEME (OR)
        // Üretilen bu maskeyi, mevcut paketimizle VEYA (OR) işlemine sok.
        // Bu sayede eski eklenen alarmlar bozulmadan yeni alarm da pakete yazılmış olur.
        #t_wTempWord := #t_wTempWord OR SHL(IN := 16#0001, N := #t_i);
        
    END_IF;
    
END_FOR;

// 3. OLUŞTURULAN PAKETİ ÇIKIŞA AKTAR
#q_wAlarmWord := #t_wTempWord;


# Good-to-know:

1- Sürücüden gelen 16-bitlik veriyi Data Block'ta INT olarak tanımlayıp SHR (Sağa Kaydırma) yapmaya çalışırsak PLC saçmalar. INT değişkeninin 15. biti (En sol bit) İşaret Bitidir (Sign Bit - Artı/Eksi). Sağa kaydırma yaptığımızda PLC en sola 0 koymak yerine sayının eksi kalmasını sağlamak için 1 kopyalayarak doldurur! Bitwise işlemler, maskeleme ve kaydırma işlemleri ASLA INT veya DINT üzerinde yapılmaz. Değişkenin donanım tipi her zaman WORD veya DWORD olmalıdır.

2- Diyelim ki sürücüye "Çalış" (Bit 0) komutunu OR maskesiyle zorla yolladık: `#q_wCmd := #q_wCmd OR 16#0001;`. Operatör stop'a bastığında bu biti kapatmak için doğrudan `#q_wCmd := 0;` dersek, paketin içindeki diğer bitleri de (örneğin Hata Onaylama veya Yön biti) sıfırlamış oluruz. Sadece o biti kapatmak (Clear) istiyorsan Ters (INV) maske ile AND yapmalıyız. Veya hata riskini sıfıra indirmek için TIA Portal'a özel Slice Access (`#q_wCmd.%X0 := FALSE;`) kullanmalıyız.