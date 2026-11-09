# Neden "Kayan" Ortalama Deniyor?

Çünkü statik bir 10 şişenin değil, *zamanla akan* verilerin ortalamasını alır.
Her yeni sinyal (Örn: 100 milisaniyede bir) geldiğinde, dizideki en eski veri (10. sıradaki) silinir.
Diğer veriler bir alt satıra kaydırılır (Shift Register - FIFO).
Yeni gelen ham sensör verisi 1. sıraya yazılır.
Anında dizinin yeni toplamı ve ortalaması hesaplanıp çıkışa verilir.

*Not*: Bu işlem hafıza gerektirdiği (geçmiş 10 veriyi tutacağı) için mecburen **Function Block (FB)** içinde yazılmalıdır.


# EX: KAYAN ORTALAMA

// 1. ÖRNEKLEME TETİĞİ (R_TRIG)
// Her pals geldiğinde (Örn: saniyede bir kez) diziyi güncelleyeceğiz.
#t_bEdge := #i_bSamplePulse AND NOT #stat_bPrevPulse;
#stat_bPrevPulse := #i_bSamplePulse;

// 2. KAYAN ORTALAMA MATEMATİĞİ
IF #t_bEdge THEN
    
    // A. Shift Register (Eski verileri aşağı kaydır, 10. veri çöpe gider)
    FOR #t_i := 10 TO 2 BY -1 DO
        #stat_aBuffer[#t_i] := #stat_aBuffer[#t_i - 1];
    END_FOR;
    
    // B. En yeni sensör verisini 1. sıraya yaz.
    #stat_aBuffer[1] := #i_rRawValue;
    
    // C. Dizideki yeni 10 verinin toplamını hesapla (Önce t_rSum'ı sıfırla!).
    #t_rSum := 0.0;
    FOR #t_i := 1 TO 10 DO
        #t_rSum := #t_rSum + #stat_aBuffer[#t_i];
    END_FOR;
    
    // D. Çıkışa ortalamayı aktar.
    #q_rFilteredVal := #t_rSum / 10.0;
    
END_IF;


# Good-to-know:

1- PLC'lerdeki 32-bit REAL sayıların ondalık hassasiyet (Floating Point Precision) sınırları yüzünden haftalarca kapanmadan çalışan bir makinede küsurat sapmaları birikebilir. Bunu önlemek için, örneğin her 1.000.000 döngüde bir `#t_rSum` değerini klasik FOR döngüsüyle sıfırdan hesaplatıp tazelemek (Kalibrasyon Taraması) endüstri standardıdır.