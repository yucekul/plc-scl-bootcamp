# IEC Sayıcıları (Counters - CTU, CTD, CTUD)

**CTU (Count Up - Yukarı Sayıcı)**: CU (Count Up) girişine gelen her yükselen kenarda (FALSE'tan TRUE'ya geçiş) mevcut değeri 1 artırır. CV (Current Value), PV (Preset Value) değerine ulaştığında çıkış (Q) TRUE olur. R (Reset) girişi sayıcıyı sıfırlar.

**CTD (Count Down - Aşağı Sayıcı)**: CD girişine gelen sinyalle değeri 1 azaltır. Sayı 0'a ulaştığında çıkış TRUE olur. LD (Load) girişi sayıcıyı tekrar başlangıç değerine (PV) çeker.

**CTUD (Count Up/Down - Çift Yönlü Sayıcı)**: Hem yukarı hem aşağı sayabilen, otopark kapasite takibi gibi işlerde kullanılan birleştirilmiş bloktur.


# EX: CTU SAYAÇ

// 1. SAĞLAM KUTU SAYICISI (CTU)
// Sensör her gördüğünde (yükselen kenarında) sayıyı 1 artırır.
#stat_CounterCTU(CU := #i_bSensorBox AND NOT #i_bSensorReject, // Bozuk değilse say
                 R  := #i_bReset,
                 PV := #i_iTargetCount);

// Sayıcının mevcut değeri (CV) ekrana gönderilir.
#q_iGoodBoxes := #stat_CounterCTU.CV;

// Sayıcı hedefe ulaştığında (.Q) paket doldu sinyali ver.
#q_bPackageFull := #stat_CounterCTU.Q;

// 2. BOZUK KUTU SAYICISI (CTU)
// Bunun bir hedefi yok (PV'ye yüksek bir rakam veriyoruz), sadece istatistik için sayıyoruz.
#stat_CounterRej(CU := #i_bSensorReject,
                 R  := #i_bReset,
                 PV := 9999);

#q_iRejectBoxes := #stat_CounterRej.CV;


# Good-to-know:

1- IEC sayıcıları kendi içlerinde donanımsal olarak kenar algılama özelliğine sahiptir, `R_TRIG` koymaya gerek yoktur.

2- INT en fazla 32.767'ye kadar sayabilir. Eğer bir konveyör banttaki toplam paket sayısını aylık olarak sayacaksan bu sayı birkaç günde dolar! Sayıcıyı oluştururken açılır menüden tipini IEC_Counter yerine IEC_DCounter (DINT), hatta eksiye hiç düşmeyeceksen IEC_UDCounter (UDINT - 4.2 Milyar) olarak seçmelisin.

3- Bir sayıcının Reset (veya Load) bacağı TRUE olduğu sürece, sayma bacağına (CU veya CD) trilyonlarca sinyal gelse bile sayıcı çalışmaz, 0'da çakılı kalır. Sistemin neden saymadığını ararken her zaman önce Reset bacağına gelen sinyale (belki de takılı kalmış bir butona) bakmalısın.