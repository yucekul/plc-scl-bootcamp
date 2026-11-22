# Öncelik Hiyerarşisi (Priority Hierarchy)

Bir durum makinesinde her komut eşit değildir. Kontrol sırası yukarıdan aşağıya şöyle olmalıdır:
**ERROR (En Yüksek Öncelik - State 99)**: Acil stop, kapak açılması veya motor termiği geldiğinde makine hangi adımda (hatta Pause'da bile) olursa olsun anında 99. adıma atlar ve tüm çıkışları keser.
**PAUSE (Orta Öncelik - State 90)**: Sadece aktif çalışma adımlarındayken (10..40) devreye girer. Aktif adım numarasını `stat_iSavedState` hafızasına yedekler, çıkışları güvenli şekilde kapatır ve bekler.
**NORMAL AKIŞ (State 0..40)**: Hata ve Pause yoksa sekans normal ilerler.


# Pause vs. Error Kurtarma Farkı (Güvenlik Kuralı)

**Pause sonrası (Resume)**: Makine güvenlidir; operatör Resume dediğinde `stat_iSavedState` ile tam kaldığı adıma (ve zamanlayıcıda kalan süreye) geri döner.
**Error sonrası (Reset)**: Fiziksel bir arıza yaşanmıştır; makine asla yarıda kalan adımdan otomatik devam etmez. Arıza giderilip Reset basıldığında her zaman güvenli başlangıç noktasına (**State 0 - IDLE**) döner.


# "Default-Off" (Varsayılan Kapalı) Çıkış Deseni

`CASE OF` yapısının hemen üstünde tüm fiziksel çıkışlar (Valf, Isıtıcı, Motor) `FALSE` yapılır. `CASE` içindeki her adım sadece kendi çalıştırması gereken ekipmanı `TRUE` yapar. Böylece Pause (90) veya Error (99) adımına ani geçişlerde hiçbir çıkış yanlışlıkla açık (mühürlü) kalmaz.


# EX: ENDÜSTRİYEL YIKAMA SEKANSI

// 1. ÖNCELİKLİ GEÇİŞLER (ERROR VE PAUSE)
// Öncelik 1: ARIZA (Her şeyin üstündedir, anında 99'a çeker)
IF #i_bFault THEN
    #stat_iState      := 99;
    #stat_iSavedState := 0;

// Öncelik 2: PAUSE (Sadece aktif çalışma adımlarındayken 10..40 çalışır)
// Zaten 90'daysak veya 0'daysak içeri girmez, böylece SavedState bozulmaz!
ELSIF #i_bPause AND (#stat_iState >= 10) AND (#stat_iState <= 40) THEN
    
    // Hangi adımda olduğumuzu hafızaya kaydet
    #stat_iSavedState := #stat_iState;
    
    // EĞER Yıkama adımındayken (30) Pause basıldıysa, geçen süreyi düşüp KALAN süreyi sakla!
    // Dikkat: Timer aşağıda sıfırlanmadan önce ET (geçen süre) değerini tam burada yakalıyoruz.
    IF #stat_iState = 30 THEN
        IF #stat_WashTimer.ET < #stat_tWashRem THEN
            #stat_tWashRem := #stat_tWashRem - #stat_WashTimer.ET;
        ELSE
            #stat_tWashRem := T#0ms;
        END_IF;
    END_IF;
    
    #stat_iState := 90; // Pause adımına geç
END_IF;

// 2. DEFAULT-OFF DESENİ (GÜVENLİ ÇIKIŞ SIFIRLAMA)
// Tüm fiziksel çıkışlar varsayılan olarak kapalı başlar.
// Aşağıdaki CASE içinde sadece aktif adım kendi ekipmanını açar.
#q_bWaterValve   := FALSE;
#q_bHeater       := FALSE;
#q_bWashMotor    := FALSE;
#q_bDrainPump    := FALSE;
#t_bRunWashTimer := FALSE;

// 3. DURUM MAKİNESİ (STATE MACHINE - FSM)
CASE #stat_iState OF
        
    0: // HAZIR VE BEKLEME (IDLE)
        IF #i_bStart THEN
            #stat_iState := 10;
        END_IF;
        
    10: // ADIM 1: SU ALMA
        #q_bWaterValve := TRUE;
        
        IF #i_bWaterFull THEN
            #stat_iState := 20;
        END_IF;
        
    20: // ADIM 2: SUYU ISITMA
        #q_bHeater := TRUE;
        
        IF #i_bTempOK THEN
            // Yıkama adımına geçmeden önce hedef yıkama süresini kalan süreye yükle
            #stat_tWashRem := #i_tWashTime;
            #stat_iState   := 30;
        END_IF;
        
    30: // ADIM 3: YIKAMA (PAUSE DESTEKLİ ZAMANLAYICI)
        #q_bWashMotor    := TRUE;
        #t_bRunWashTimer := TRUE;
        
        IF #stat_WashTimer.Q THEN
            #stat_iState := 40;
        END_IF;
        
    40: // ADIM 4: BOŞALTMA (TAHLİYE)
        #q_bDrainPump := TRUE;
        
        IF #i_bWaterEmpty THEN
            #stat_iState := 0; // Çevrim başarıyla bitti
        END_IF;
        
    90: // DURAKLATILDI (PAUSE)
        // Default-Off sayesinde tüm valf/motor/ısıtıcı otomatik kapalı!
        // Pause butonu bırakıldıysa (NOT i_bPause) ve Resume basıldıysa kaldığın yere dön
        IF #i_bResume AND NOT #i_bPause THEN
            #stat_iState := #stat_iSavedState;
            
        // Operatör devam etmek istemeyip iptal (Abort) ederse başa dön
        ELSIF #i_bAbort THEN
            #stat_iSavedState := 0;
            #stat_iState      := 0;
        END_IF;
        
    99: // ARIZA KİLİDİ (ERROR)
        // Tüm çıkışlar kapalı. Arıza fiziksel olarak kalkmadan Reset çalışmaz!
        IF (NOT #i_bFault) AND #i_bReset THEN
            #stat_iState := 0;
        END_IF;
        
    ELSE
        #stat_iState := 0;
        
END_CASE;

// 4. MERKEZİ YIKAMA ZAMANLAYICISI (CASE DIŞINDA)
// Pause'a (90) geçildiği an t_bRunWashTimer FALSE olacağı için Timer temizce sıfırlanır.
// Resume ile 30'a dönüldüğünde ise PT olarak güncellenmiş stat_tWashRem (kalan süre) ile sayar!
#stat_WashTimer(IN := #t_bRunWashTimer,
                PT := #stat_tWashRem);

// 5. HMI DURUM BAYRAKLARI
#q_iState  := #stat_iState;
#q_bPaused := (#stat_iState = 90);
#q_bError  := (#stat_iState = 99);


# Good-to-know:

1- Endüstriyel makinelerde komutlar eşit ağırlıkta değildir. Değişmez hiyerarşi şu sıradadır:Acil Stop / Error > Güvenlik Kapısı > Pause > Cycle Stop > Resume / Start. Aynı PLC taramasında hem bir kritik hata biti hem de Resume sinyali `TRUE` gelirse, makine asla yürüyememelidir. Bu yüzden global arıza yakalama bloğu her zaman `CASE` bloğunun en üstünde yer alır ve diğer tüm durumları tek hamlede ezer.

2- Çıkışları her zaman iki sınıfa ayırmalıyız: Enerji kesildiğinde durması gereken tehlikeli hareketler (Motor, Pres, Isıtıcı -> Pause'da kesilir) ve Enerji kesildiğinde tehlike yaratacak tutucular (Clamp, Vakum, Fren -> Pause'da korunur).

3- Bir arıza (ERROR) yaşandığında operatör Reset butonuna bastığı an makine kesinlikle kendiliğinden hareket etmemelidir. Endüstriyel güvenlik standartlarına (ISO 13849) göre Reset yalnızca arızayı onaylar ve sistemi güvenli hazırlık (RESETTING -> IDLE) aşamasına taşır; yeni hareket mutlaka operatörün bilinçli olarak vereceği ayrı bir Start yükselen kenarı (R_TRIG) ile başlamalıdır.

4- Operatör makineyi 20. adımda `PAUSE` yapıp güvenlik kapısını açtığında, içerideki sıkışmış bir parçayı eliyle çıkarmış veya bir pistonu manuel olarak itmiş olabilir. Kapıyı kapatıp `Resume` bastığında makine körlemesine 20. adımdan devam ederse mekanik çarpışma yaşanır. Kritik makinelerde `PAUSE` ile `RUN` arasına kısa bir `UNHOLDING` (Devama Hazırlık) ara durumu konur. Bu adımda önce pnömatik hat basıncı geri verilir, ardından mevcut adımın ön şart sensörlerinin hâlâ yerli yerinde olup olmadığı doğrulanır; sensörler bozulmuşsa Resume reddedilir ve makine `RESETTING` (Home) istemeye zorlanır.

5- Eğer 15 saniyelik bir yapıştırma veya ısıtma adımının 12. saniyesinde güvenlik kapısı açılıp PAUSE olursa, standart TON kullandığında süre sıfırlanır ve Resume sonrası parça fazladan 15 saniye daha baskıda kalarak yanar/ezilir. Proses bekleme zamanlayıcılarını TONR (kalıcı zamanlayıcı) ile kurup sadece adım değişiminde veya IDLE/ERROR durumlarında R bacağından sıfırlamak bu kalite hatasını tamamen ortadan kaldırır.