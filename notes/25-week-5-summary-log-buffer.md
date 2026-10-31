# Mimari: Array of UDT (UDT Dizisi)

Geçmiş kayıtları tutarken 3 ayrı dizi (Saatler Dizisi, Barkodlar Dizisi, Durumlar Dizisi) oluşturmak "Spagetti Kod" yaklaşımıdır.
Doğru mimari:
Tek bir kaydın şablonunu UDT olarak tasarlamak.
Bu UDT'den oluşan 10 elemanlı tek bir Array tanımlamak.
Bu sayede Shift Register (Kaydırma) işlemi yaparken tek bir satır kodla o ürünün saati, barkodu ve kalite durumu aynı anda alt satıra kayar.


# EX: PRODUCTION LOGGER

// 1. ÜRETİM BİTTİ SİNYALİNİ (YÜKSELEN KENAR) YAKALAMA
#t_bEdge := #i_bTrigger AND NOT #stat_bPrevTrigger;
#stat_bPrevTrigger := #i_bTrigger;

// 2. EĞER YENİ BİR ÜRÜN ÇIKTIYSA LOG TABLOSUNU GÜNCELLE
IF #t_bEdge THEN
    
    // A. SHIFT REGISTER (Geçmiş verileri 1 alt satıra kaydır - FIFO)
    // En eski veri olan 10. satır silinir, 9. satır 10'a geçer.
    FOR #t_i := 10 TO 2 BY -1 DO
        // UDT dizisi olduğu için tek hamlede saati, barkodu ve durumu aşağı kaydırmış oluyoruz!
        #stat_aLogBuffer[#t_i] := #stat_aLogBuffer[#t_i - 1];
    END_FOR;
    
    // B. YENİ ÜRÜNÜ EN ÜSTE (1. SATIRA) YAZ
    // Dışarıdan gelen barkodu ve kalite durumunu kaydet.
    #stat_aLogBuffer[1].Barcode := #i_sCurrentBarcode;
    #stat_aLogBuffer[1].IsOK    := #i_bStatusOK;
    
    // C. ÜRETİM SAATİNİ PLC'DEN OKUYUP DOĞRUDAN 1. SATIRA DAMGALA
    #t_iRetVal := RD_LOC_T(#stat_aLogBuffer[1].Timestamp);
    
END_IF;


# Good-to-know:

1- Log Buffer DB'sini kesinlikle Retain (Kalıcı) yapmalıyız. Yoksa operatör hafta sonu şalteri kapattığında fabrikanın tüm üretim geçmişi silinir. Ancak her CPU'nun kalıcı hafızası kısıtlıdır (Örn: S7-1200'lerde genellikle 10 KB veya 14 KB civarıdır). İçinde String ve DTL olan büyük UDT'lerden 10.000 elemanlı dizi açıp Retain yapmaya çalışırsak PLC projeyi yüklemeyi reddeder. (Eğer binlerce kayıt tutmak gerekiyorsa, bunu PLC'de değil SCADA/SQL tarafında yapmalıyız).

2- Eğer kodun başında "Eğer indeks 0 ise 1 yap" gibi bir güvenlik koymazsak, ilk kayıtta PLC 0 numaralı çekmeceye (dizi 1..100 tanımlı olduğu için) yazmaya çalışır ve anında STOP'a düşer.

3- HMI tarafında bu Buffer'ı bir tablo olarak göstereceksek, operatör tablonun en üstünde her zaman "En son üretileni" görmek ister. SCADA yazılımları bu Dairesel Buffer yapısını doğrudan destekler; SCADA'ya sadece Dizinin başlangıç adresini ve senin Pointer'ını (İndeksi) verirsin, o tabloyu tarihe göre otomatik olarak yukarıdan aşağıya (sıralı) dizer.