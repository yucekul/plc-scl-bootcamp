# Filtre Mantığı

Sıcaklık veya seviye gibi fiziksel değerler saniyeler içinde aniden 50 dereceden 80 dereceye fırlayamaz. Eğer PLC'de böyle bir sıçrama görüyorsak bu elektriksel bir gürültüdür. Sistemi bu sahte gürültülere karşı korumak için yazılımsal filtreler (Yumuşatma/Smoothing) kullanırız.


# EWMA (Exponentially Weighted Moving Average) Mantığı

Filtrenin bir "Yumuşatma Çarpanı" (Alpha, \alpha) vardır. \alpha değeri 0.0 ile 1.0 arasındadır.

// 1. GÜVENLİK: Alpha çarpanını 0.0 ile 1.0 arasına sıkıştır.
// Yanlışlıkla 5.0 gibi bir değer girilirse formül patlar ve sistem kararsızlaşır.
#t_rSafeAlpha := LIMIT(MN := 0.0, IN := #i_rAlpha, MX := 1.0);

// 2. MATEMATİKSEL FİLTRE (EWMA) UYGULAMASI
// Yeni değerin belli bir yüzdesini al, eski değerin geri kalan yüzdesiyle topla.
#iq_rFiltered := (#i_rRawValue * #t_rSafeAlpha) + (#iq_rFiltered * (1.0 - #t_rSafeAlpha));


# Good-to-know:

1- EWMA formülü zaman bazlı çalışmaz, "döngü (scan)" bazlı çalışır.
Eğer bu kodu OB1 (Ana döngü) içine yazarsan, OB1 bazen 5 milisaniyede, bazen 10 milisaniyede döndüğü için filtrenin yumuşatma karakteri sürekli değişir. EWMA ve PID gibi zamana hassas algoritmalar her zaman OB30 (Cyclic Interrupt) gibi sabit zamanlı (örneğin tam 100ms'de bir çalışan) blokların içinde çağrılmalıdır.

2- EWMA'nın en ölümcül hatası budur. PLC ilk açıldığında #iq_rFiltered sıfırdır (0.0). Eğer kazandaki su o an 100°C ise ve sen Alpha'yı çok yavaş bir değer (0.01) yaptıysan, filtrenin 0'dan 100'e tırmanması dakikalar sürer! Bu sürede PLC suyu 10, 20, 30 sanıp sisteme "LL (Çok Soğuk)" acil durum alarmı verdirir. PLC ilk başladığında (First Scan bit'i %M1.0 gibi veya bir başlatma butonu ile) filtreyi "Ham değere eşitleyerek" doldurmalısın.

IF #i_bFirstBit THEN
    #iq_rFiltered := #i_rRawValue; // Filtreyi şarj et!
ELSE
    // EWMA formülü buraya yazılır.
END_IF;

3- Sıcaklık (Zaten yavaş değişir): Alpha'yı büyük tutabilirsin (0.2 - 0.5 arası).
Su Seviyesi (Dalgalanır): Alpha'yı biraz daha düşürebilirsin (0.05 - 0.1).
Basınç (Çok hızlı ve gürültülüdür): Alpha'yı çok çok düşük tutmalısın (0.01 - 0.05).