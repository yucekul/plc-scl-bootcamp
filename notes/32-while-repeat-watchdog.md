# WHILE (İken) Döngüsü

Koşul **DOĞRU (TRUE)** olduğu sürece döner. Önce koşula bakar, doğruysa içeri girer. Koşul baştan yanlışsa hiç dönmeyebilir.


# EX: SEARCH VALUE

// 1. BAŞLANGIÇ AYARLARI (SIFIRLAMA)
// Arama işlemine dizinin 1. kutusundan (indeks 1) başlıyoruz.
#t_i := 1;
#t_bFound := FALSE;
#q_bTimeout := FALSE;
#q_iFoundAt := 0;

// 2. GÜVENLİ WHILE DÖNGÜSÜ İLE ARAMA
// Şart 1: Aradığımız sayıyı henüz BULAMADIYSAK döngüye devam et (NOT t_bFound).
// Şart 2 (Watchdog Koruması): İndeks 100'ü GEÇMEDİĞİ SÜRECE devam et (#t_i <= 100).
WHILE (NOT #t_bFound) AND (#t_i <= 100) DO
    
    // O an baktığımız kutudaki sayı, aradığımız sayıya eşit mi?
    IF #i_aDataBox[#t_i] = #i_iTarget THEN
        // Bulduk! Bayrağı TRUE yapıyoruz. 
        // Bir sonraki mikrosaniyede WHILE şartı bozulacak ve döngüden anında çıkılacak.
        #t_bFound := TRUE;
    ELSE
        // Bulamadık. Bir sonraki kutuya geçmek için manuel sayacımızı 1 artırıyoruz.
        // FOR döngüsündeki otomatik artışın manuel halidir.
        #t_i := #t_i + 1;
    END_IF;
    
END_WHILE;

// 3. SONUÇ DEĞERLENDİRMESİ VE ÇIKIŞLAR
IF #t_bFound THEN
    // Döngüden başarıyla (sayıyı bularak) çıktık.
    // Döngünün durduğu o anki indeksi (sırayı) dışarı aktar.
    #q_iFoundAt := #t_i;
    
ELSIF #t_i > 100 THEN
    // Döngüden, sayıyı bulduğumuz için değil; güvenlik sınırını (100) aştığımız için çıktık.
    // Bu demek oluyor ki aranan sayı dizide hiç yok.
    #q_iFoundAt := 0;
    #q_bTimeout := TRUE; // Operatöre veya sisteme sayının bulunamadığını bildir.
    
END_IF;


# Good-to-know:

1- "Su 100 derece olana kadar bekle" mantığını WHILE içine yazarsan, suyun 100 derece olması dakikalar süreceği için PLC'nin Watchdog'u (150ms) anında patlar. Döngüler sadece matematiksel ve dizisel (Array) hesaplamalar içindir; fiziksel zamanı beklemek için 17. Günde gördüğümüz State Machine (CASE) yapısı kullanılır.

2- Clean Code, sadece bir maksimum indeks sınırlamasıyla FOR döngüsü kullanıp, aranan değer bulunduğunda EXIT ile çıkmak WHILE kullanmaktan çok daha güvenlidir.


# The Code:

#t_i := 1;
#t_bFound := FALSE;
#q_bTimeout := FALSE;
#q_iFoundAt := 0;

WHILE (NOT #t_bFound) AND (#t_i <= 100) DO
    
    IF #i_aDataBox[#t_i] = #i_iTarget THEN
        #t_bFound := TRUE;
    ELSE
        #t_i := #t_i + 1;
    END_IF;
    
END_WHILE;

IF #t_bFound THEN
    #q_iFoundAt := #t_i;
    
ELSIF #t_i > 100 THEN
    #q_iFoundAt := 0;
    #q_bTimeout := TRUE;
    
END_IF;