# State Machine Nedir?

Makinenin o an tam olarak hangi adımda olduğunu bir tam sayı (Integer) değişkeniyle takip ettiğimiz mimaridir. Makine aynı anda sadece 1 adımda bulunabilir. Bir adımın işi bitince, değişkenin değeri değiştirilerek bir sonraki adıma atlanır.

**10'ar 10'ar Atlama**: Adım numaraları (Step Numbers) asla 1, 2, 3 diye verilmez. Her zaman 10, 20, 30 diye verilir. Çünkü aylar sonra 10. ile 20. adım arasına yeni bir özellik ("15: Kapağı Kilitle" gibi) eklemek istersek, tüm kodun adım numaralarını değiştirmek zorunda kalmayız.

**Çıkışları CASE Dışında Atamak**: CASE bloğunun içinde çıkışlara (`#Motor := TRUE;` gibi) değer atamak çok tehlikelidir, çıkışlar asılı kalabilir (gizli mühürleme). En temiz yöntem, CASE içinde sadece adım numaralarını (Step) değiştirmek; fiziksel çıkışları ise CASE bloğunun en altında doğrudan mantıksal olarak atamaktır.


# EX: SIVI KARIŞIM TANKI SIRALAYICISI

// 1. ZAMANLAYICI (TIMER) ÇAĞRIMI
// Timer, makine sadece Adım 20'de (Karıştırma) iken çalışsın.
#stat_TimerMix(IN := (#stat_iStep = 20),
               PT := T#5s);

// 2. STATE MACHINE (DURUM MAKİNESİ) - Sadece adım geçişlerini yönetir
CASE #stat_iStep OF
    0:  // IDLE (BEKLEME) ADIMI
        IF #i_bStart AND NOT #i_bSensorHigh THEN
            #stat_iStep := 10; // Dolum adımına geç
        END_IF;
        
    10: // DOLUM ADIMI
        IF #i_bSensorHigh THEN
            #stat_iStep := 20; // Doldu, karıştırma adımına geç
        END_IF;
        
    20: // KARIŞTIRMA ADIMI
        IF #stat_TimerMix.Q THEN
            #stat_iStep := 30; // Süre bitti, boşaltma adımına geç
        END_IF;
        
    30: // BOŞALTMA ADIMI
        // i_bSensorLow FALSE ise (su bittiyse)
        IF NOT #i_bSensorLow THEN
            #stat_iStep := 0; // İşlem bitti, başa dön
        END_IF;
        
    ELSE
        // GÜVENLİK: Bilinmeyen bir adıma geçerse sistemi başa al
        #stat_iStep := 0;
END_CASE;

// 3. FİZİKSEL ÇIKIŞLAR (CLEAN CODE YAKLAŞIMI)
// Hangi çıkışın hangi adımda aktif olacağını tek satırda asılı kalma riski olmadan tanımlıyoruz.
#q_bFillValve  := (#stat_iStep = 10);
#q_bMixer      := (#stat_iStep = 20);
#q_bDrainValve := (#stat_iStep = 30);

// HMI ekranında göstermek için anlık adımı Output'a aktar
#q_iCurrentStep := #stat_iStep;