# FB (Function Block) Mimarisi ve TON Zamanlayıcısı

**FC (Function)**: Hafızası yoktur (Amnesia). Sadece o anki girişlere göre bir sonuç üretir. Hafıza gerekiyorsa `IN_OUT` ile global bir adres (`%M`) referans gösterilmek zorundadır.

**FB (Function Block)**: Kendi özel hafızası (Instance DB) vardır (Fil hafızası). Blok içindeki `Static` (stat) bölümünde tanımlanan her değişken kalıcıdır. PLC enerjisi kesilmedikçe veya kodla sıfırlanmadıkça değerini korur. Mühürleme için artık `IN_OUT` kullanmaya gerek yoktur.


# IEC Timers (TON - Timer On Delay)

TON (Çekmede Gecikmeli Zamanlayıcı), endüstride en çok kullanılan timer'dır. 

**IN**: Timer'ı başlatan sinyal (TRUE olduğu sürece sayar).

**PT (Preset Time)**: Hedeflenen bekleme süresi (T#5s gibi).

**Q**: IN sinyali TRUE kalıp, PT süresi dolduğunda TRUE olan çıkış.

**ET (Elapsed Time)**: O an geçen süre.


# EX: TON

// 1. SİSTEM MÜHÜRLEMESİ (Artık IN_OUT bacağına ihtiyacımız yok!)
// FB'nin kendi Static hafızası olduğu için stat_bActive değerini saklayabilir.
#stat_bActive := (#i_bStart OR #stat_bActive) AND NOT #i_bStop;

// 2. ZAMANLAYICI (TON) ÇAĞRIMI
// Sistem aktif olduğu anda Timer saymaya başlasın.
#stat_TimerTON(IN := #stat_bActive,
               PT := #i_tDelay);

// 3. ÇIKIŞ KONTROLLERİ
// Eğer sistem aktifse (TRUE) VE Timer henüz süreyi doldurmadıysa (.Q = FALSE) Uyarı sireni çalsın.
#q_bSiren := #stat_bActive AND NOT #stat_TimerTON.Q;

// Timer süresini doldurduğunda (.Q = TRUE), motor devreye girsin.
#q_bMotor := #stat_TimerTON.Q;


# Good-to-know:

1- Dışarıdan (HMI ekranından) gelen süreler genelde INT veya DINT (milisaniye cinsinden) gelir. Bunu timer'a bağlamak için DINT_TO_TIME(HMI_Deger) fonksiyonunu kullanmalısın.

2- Parazit engelleme mantığı (Histerezis) TON ile yapılır. Bir seviye sensörü suyu "Gördü/Görmedi" diye dalgalanıyorsa, 2-3 sn timer saydırdıktan sonra çıkış verilebilir.