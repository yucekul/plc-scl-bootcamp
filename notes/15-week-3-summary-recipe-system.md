# Dinamik Dizi İndeksleme (Dynamic Array Indexing)

Dizilerin içine sabit sayı (`Recipe[1]`) yazmak yerine bir değişken (`Recipe[Secilen_Urun]`) yazarak istediğimiz kutucuktaki veriyi çekebiliriz.


# EX: RECIPE BATCH CONTROLLER

// 1. DİZİ SINIR KONTROLÜ (Array Out of Bounds Koruması)
// Operatör 1 ile 5 dışında bir reçete numarası girdiyse sistemi kilitle!
IF (#i_iRecipeSelect < 1) OR (#i_iRecipeSelect > 5) THEN
    #q_bRecipeError := TRUE;
    #stat_tActiveTime := T#0s;
ELSE
    #q_bRecipeError := FALSE;
    // Seçilen reçetenin süresini diziden dinamik olarak çek
    #stat_tActiveTime := #i_tRecipeTimes[#i_iRecipeSelect];
END_IF;

// 2. ZAMANLAYICI (TP) İLE ÜRETİM (Sadece reçete hatası yoksa başlatılabilir)
#stat_TimerTP(IN := #i_bStartBatch AND NOT #q_bRecipeError,
              PT := #stat_tActiveTime);

// Karıştırıcı motor, TP zamanlayıcısı aktif olduğu sürece çalışır
#q_bMixerMotor := #stat_TimerTP.Q;

// 3. ÜRETİM SAYICISI (CTU)
// Motor her çalıştığında (başladığı yükselen kenarda) üretim sayısını 1 artır.
#stat_BatchCTU(CU := #q_bMixerMotor,
               R  := #i_bResetCounter,
               PV := 9999);

#q_iTotalBatches := #stat_BatchCTU.CV;


# Good-to-know:

1- FB'nin Static kısmına veya Global DB'ye açtığın reçete dizilerinin yanındaki Retain (Kalıcı) kutucuğunu mutlaka işaretlemelisin. Eğer işaretlemezsen, fabrikada hafta sonu şalterler kapatılıp pazartesi açıldığında operatörün girdiği 100 tane reçetenin tamamı sıfırlanır.

2- Zamanlayıcı blokları asla IF içine hapsedilmez! Her zaman ana gövdede açıkta çağrılır; sadece .Q çıkışları IF içinde sorgulanır.