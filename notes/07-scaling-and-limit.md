# SCALE_X (Skalalama)

**Skalalama**: Normalize edilmiş bir oranı (0.0 - 1.0), fiziksel bir aralığa oturtma işlemidir (Real).

// 1. GÜVENLİK: Gelen oranı 0.0 ile 1.0 arasına sınırla (Kelepçele)
// Eğer i_rNormValue 0.0'dan küçükse 0.0 yapar, 1.0'dan büyükse 1.0 yapar.
#t_rSafeNorm := LIMIT(MN := 0.0, IN := #i_rNormValue, MX := 1.0);

// 2. SKALALAMA MANTIĞI: Y = (Fark * Oran) + Min
#q_rPhysicalValue := ((#i_rScaleMax - #i_rScaleMin) * #t_rSafeNorm) + #i_rScaleMin;


# Good-to-know: 

1- 4-20mA "Kopuk Kablo" (Wire Break) Teşhisi: Eğer sensörün 4-20mA çıkışlıysa, PLC'nin analog kartı 4mA akım gördüğünde 0 değerini üretir, 20mA gördüğünde 27648 üretir.
Peki ya kablo koparsa veya sensör bozulursa? Akım 0mA'e düşer. Bu durumda PLC negatif bir değer (yaklaşık -6912) okur!
Bunu koda şöyle bir güvenlik duvarı (Alarm) olarak eklemelisin:

IF #i_rNormValue < -100.0 THEN
    #q_bFaultAlarm := TRUE;
    #t_rSafeNorm := 0.0; // Sistemi korumaya al.
ELSE
    // NORM_X ve SCALE_X kodları buraya yazılır...