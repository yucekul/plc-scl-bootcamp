# TIA Portal'daki En Meşhur SCL Tuzağı

Bir `TON` bloğunu doğrudan `CASE` içindeki bir adımın (`20:`) içine yazarsanız, makine 20. adımdan 30. adıma geçtiği an o `TON` bloğu bir daha çağrılmaz (çalıştırılmaz).
`IN` girişi hiç `FALSE` göremediği için Timer kendini sıfırlayamaz (`Q` çıkışı `TRUE` olarak donup kalır).
Makine bir sonraki çevrimde 20. adıma tekrar girdiğinde, Timer 3 saniye beklemeden anında "Süre doldu!" deyip adımı atlar!


## Altın Kural: Timer Dışarıda Çağrılır, İçeride Yönetilir

`TON` blokları `CASE OF` yapısının **dışında** (altında) her PLC taramasında kesintisiz çağrılır.
`CASE` içindeki adımlar sadece "Bu adımda Timer çalışsın (`t_bRunStepTimer := TRUE`) ve süresi şu olsun (`t_tStepPT := T#3s`)" der.
**Adım Değişim Koruması (`stat_iState <> stat_iPrevState`)**: Eğer peş peşe iki adım da aynı Timer'ı kullanıyorsa, adım değiştiği o tek PLC taramasında Timer'ın `IN` bacağı 1 tarama boyunca `FALSE` yapılarak temizce sıfırlanır.


## Adım Zaman Aşımı (Step Timeout / Watchdog)

Hareket içeren her adımda (Örn: Piston aşağı iniyor), bir bekçi zamanlayıcı çalışır. Piston örneğin 5 saniye içinde hedef sensöre basamazsa mekanik sıkışma veya sensör arızası var demektir; sistem otomatik olarak `99: FAULT` adımına atlar.


# EX: PRESS CYCLE TIMER

// 1. HER TARAMA BAŞINDA GEÇİCİ İSTEKLERİ SIFIRLA
// Hangi adımda olursak olalım, o adım özellikle talep etmedikçe
// zamanlayıcılar varsayılan olarak kapalı (FALSE) başlar.
#t_bRunStepTimer  := FALSE;
#t_tStepPT        := T#0ms;
#t_bMonitorMotion := FALSE;

// 2. DURUM MAKİNESİ (STATE MACHINE - FSM)
CASE #stat_iState OF
        
    0: // HAZIR VE BEKLEME (IDLE)
        #q_bPressValve := FALSE;
        
        // Pres yukarıdaysa (Home) ve Start geldiyse başla
        IF #i_bStart AND #i_bSensorUp THEN
            #stat_iState := 10;
        END_IF;
        
    10: // PRES AŞAĞI İNİYOR (HAREKET ADIMI)
        #q_bPressValve    := TRUE;
        #t_bMonitorMotion := TRUE; // Sıkışma bekçisini devreye al!
        
        IF #i_bSensorDown THEN
            #stat_iState := 20;
        END_IF;
        
    20: // BASKIDA BEKLEME (ZAMANLAYICI ADIMI 1)
        #q_bPressValve   := TRUE;
        #t_bRunStepTimer := TRUE;          // Ortak adım zamanlayıcısını çalıştır
        #t_tStepPT       := #i_tPressTime; // Hedef süreyi ver (Örn: 3 sn)
        
        IF #stat_StepTimer.Q THEN
            #stat_iState := 30;
        END_IF;
        
    30: // PRES YUKARI ÇIKIYOR (HAREKET ADIMI)
        #q_bPressValve    := FALSE;
        #t_bMonitorMotion := TRUE; // Geri dönüşte de sıkışma kontrolü yap!
        
        IF #i_bSensorUp THEN
            #stat_iState := 40;
        END_IF;
        
    40: // SOĞUMA / DİNLENME BEKLEMESİ (ZAMANLAYICI ADIMI 2)
        // Dikkat: Adım 20'de kullandığımız AYNI Timer'ı burada 2 saniye için yeniden kullanıyoruz!
        #q_bPressValve   := FALSE;
        #t_bRunStepTimer := TRUE;
        #t_tStepPT       := T#2s;
        
        IF #stat_StepTimer.Q THEN
            #stat_iState := 0; // Çevrim bitti, başa dön
        END_IF;
        
    99: // ARIZA DURUMU (TIMEOUT / SIKIŞMA)
        #q_bPressValve := FALSE; // Güvenlik için presi anında serbest bırak
        
        // Operatör sıkışmayı giderip Reset'e bastığında başa dön
        IF #i_bReset THEN
            #stat_iState := 0;
        END_IF;
        
    ELSE
        #stat_iState := 0;
        
END_CASE;

// 3. ADIM DEĞİŞİM TESPİTİ VE MERKEZİ ZAMANLAYICILAR
// Eğer yukarıdaki CASE içinde adım numarası değiştiyse (Örn: 20 -> 30),
// t_bStateChanged o tek tarama için TRUE olur ve Timer'ların IN girişini keserek sıfırlar.
#t_bStateChanged := (#stat_iState <> #stat_iPrevState);

// A. Ortak Adım Bekleme Zamanlayıcısı (Her taramada çalışır!)
#stat_StepTimer(IN := #t_bRunStepTimer AND NOT #t_bStateChanged,
                PT := #t_tStepPT);

// B. Hareket Sıkışma (Timeout) Zamanlayıcısı (Her taramada çalışır!)
#stat_TimeoutTimer(IN := #t_bMonitorMotion AND NOT #t_bStateChanged,
                   PT := #i_tTimeoutLimit);

// Eğer bir hareket adımında süre dolduysa sistemi anında 99 (ARIZA) adımına at!
IF #stat_TimeoutTimer.Q THEN
    #stat_iState := 99;
END_IF;

// Bir sonraki tarama için mevcut adımı hafızaya kaydet
#stat_iPrevState := #stat_iState;

// 4. DURUM ÇIKIŞLARI
#q_iState      := #stat_iState;
#q_bTimeoutErr := (#stat_iState = 99);


# Good-to-know:

1- Bir TON bloğunu doğrudan bir CASE adımının içine yazmak sahadaki en yaygın FSM hatasıdır. Süre dolup bir sonraki adıma geçildiği an eski adım artık taranmaz; zamanlayıcının `IN` bacağına hiç `FALSE` gelmediği için `.Q` çıkışı hafızada `TRUE` olarak donar. Makine bir sonraki çevrimde aynı adıma tekrar girdiğinde, zamanlayıcı daha ilk milisaniyede eski TRUE değerini okur ve beklemesi gereken adımı sıfır saniyede atlayıp geçer. Zamanlayıcılar daima `CASE OF` yapısının dışında, her tarama döngüsünde kesintisiz çalıştırılmalı ve adım numarası değiştiği an tek taramalık bir sinyal ile otomatik sıfırlanmalıdır.

2- Eski nesil S7-300/400 PLC'lerin aksine, yeni nesil S7-1200 ve S7-1500 işlemcilerinde IEC zamanlayıcılar (TON, TOF, TP) sadece çağrıldıkları satırda değil, kod içinde .Q veya .ET bacağına her erişildiğinde arka planda anlık güncellenir. Aynı tarama döngüsünün üst satırında FALSE okunan bir .Q çıkışı, süre tam o mikrosaniyede dolduğu için 20 satır aşağıda TRUE dönebilir; bu da tek taramada çelişkili adım geçişlerine (Race Condition) yol açar. Zamanlayıcıyı her taramada tek bir noktada çağır, çıkışını standart bir Bool değişkene aktarıp ve CASE içindeki tüm mantık kontrollerinde sadece bu sabit kopyayı kullanmalıyız.

3- Timeout (Sıkışma) Zamanlayıcıları: Makine Pause modundayken mutlaka kesilmeli veya dondurulmalıdır. Aksi halde sistem güvenle beklerken süre dolar ve operatöre sahte "Mekanik Sıkışma" alarmı düşer.

Uzun Proses Zamanlayıcıları: Fırınlama, karıştırma veya yapıştırıcı kürleme gibi uzun adımlarda standart TON kullanırsan, kapı açılıp kapandığında süre sıfırdan başlar ve ürün yanar ya da bozulur. Kesintiden sonra kaldığı yerden devam etmesi gereken adımlarda mutlaka kalıcı zamanlayıcı (TONR) kullanılmalıdır.

4- Bir mekanik hareketin ideal süresi 2 saniyeyse Timeout limitini 2.1 saniye yapmak sahada sürekli yalancı arıza üretir. Kışın hidrolik yağın kalınlaşması, fabrikanın genel hava basıncının anlık düşmesi veya silindir keçelerinin aşınması hareketi birkaç yüz milisaniye yavaşlatır. Hareket Timeout süreleri genellikle normal strok süresinin 1.5 ila 2 katı (veya normal süre + güvenli tolerans) olarak belirlenir ve HMI üzerinden ayarlanabilir yapılır. Ancak HMI'dan yanlışlıkla 0 girilmesi ihtimaline karşı yazılımda daima bir alt limit koruması (örneğin en az T#500ms) bırakılmalıdır.

5- 40 adımlı bir makinede tüm zaman aşımlarını doğrudan tek bir hata adımına yönlendirdiğinde, aktif adım numarası silinir ve ekranda sadece genel bir "Timeout Hatası" görünür. Bakımcı hangi pistonun veya sensörün beklemede kaldığını bulmak için makineyi tek tek incelemek zorunda kalır. Hata durumuna geçiş yapmadan hemen önce, zaman aşımının yaşandığı adım numarası ayrı bir teşhis (diagnostic) değişkenine kilitlenmelidir. Böylece HMI Text List üzerinden doğrudan arızalı hareketin adı (örneğin "Adım 30: Tutucu Geri Sensörü Gelmedi") operatöre bildirilir.