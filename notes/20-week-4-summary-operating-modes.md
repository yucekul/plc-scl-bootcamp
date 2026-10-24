# Mod Yönetimi (Mode Management)

Endüstride genellikle bir Integer değişken ile sistemin modu belirlenir:

**0 = STOP (Kapalı)**: Makine hiçbir şeye tepki vermez, tüm çıkışlar kesilir.

**1 = MANUEL**: Adım adım (State Machine) ilerleme durur. Operatör ekrandan (HMI) valfleri ve motorları tek tek açıp kapatabilir. Güvenlik sensörleri yine de devrede kalmalıdır.

**2 = OTOMATİK**: Makine kendi State Machine algoritmasına (CASE yapısına) göre çalışır.


# Temel Güvenlik Kuralı

Makine Otomatik modda tam üretim yaparken (örneğin tank dolarken) operatör aniden şalteri Manuel'e çevirirse ne olur? 

Mod değişimi algılandığı an State Machine derhal güvenli adıma (IDLE) sıfırlanmalı ve fiziksel çıkışlar anında kesilmelidir. Aksi takdirde modlar arası geçişte çıkışlar asılı kalır.


# EX: TANK MODES

// 0. MOD GÜVENLİK KONTROLÜ
// Sistem OTOMATİK (2) modda değilse makineyi IDLE adımında kilitle.
IF #i_iMode <> #MODE_AUTO THEN
    #stat_iStep := #IDLE;
END_IF;

// 1. ADIM DEĞİŞİMİNİ YAKALAMA (EDGE DETECTION)
#stat_bStepChanged := (#stat_iStep <> #stat_iPrevStep);
#stat_iPrevStep := #stat_iStep;

// 2. WATCHDOG TIMER (KORUMA KRONOMETRESİ)
#stat_TimerWatchdog(IN := (#stat_iStep > 0) AND (#stat_iStep <> #FAULT) AND NOT #stat_bStepChanged,
                    PT := #i_tMaxStepTime);

IF #stat_TimerWatchdog.Q THEN
    #stat_iStep := #FAULT;
END_IF;

// 3. KARIŞTIRMA (MIX) ADIMI ZAMANLAYICISI
#stat_TimerMix(IN := (#stat_iStep = #MIX),
               PT := T#5s);

// 4. STATE MACHINE (DURUM MAKİNESİ)
CASE #stat_iStep OF
    #IDLE:
        IF #i_bStart AND NOT #i_bSensorHigh THEN
            #stat_iStep := #FILL;
        END_IF;
        
    #FILL:
        IF #i_bSensorHigh THEN
            #stat_iStep := #MIX;
        END_IF;
        
    #MIX:
        IF #stat_TimerMix.Q THEN
            #stat_iStep := #DRAIN;
        END_IF;
        
    #DRAIN:
        IF NOT #i_bSensorLow THEN
            #stat_iStep := #IDLE;
        END_IF;
        
    #FAULT:
        // Arıza durumundayız. Operatör hatayı giderip Reset'e basana kadar bekle.
        IF #i_bResetAlarm THEN
            #stat_iStep := #IDLE;
        END_IF;
        
    ELSE
        #stat_iStep := #IDLE;
END_CASE;

// 5. FİZİKSEL ÇIKIŞ ATAMALARI (MOD SEÇİMİ İLE)
// Çıkış = (Oto Modda İse VE Adımı Geldiyse) YADA (Manuel Modda İse VE Butona Basıldıysa)
#q_bFillValve := (#i_iMode = #MODE_AUTO AND #stat_iStep = #FILL) OR 
                 (#i_iMode = #MODE_MANUAL AND #i_bManFill);

#q_bMixer     := (#i_iMode = #MODE_AUTO AND #stat_iStep = #MIX) OR 
                 (#i_iMode = #MODE_MANUAL AND #i_bManMix);

#q_bDrainValve:= (#i_iMode = #MODE_AUTO AND #stat_iStep = #DRAIN) OR 
                 (#i_iMode = #MODE_MANUAL AND #i_bManDrain);

// Alarm ve anlık adım ekran bilgileri her modda aktiftir.
#q_bTimeoutAlarm := (#stat_iStep = #FAULT);
#q_iCurrentStep  := #stat_iStep;


# Good-to-know: 

1- Manuel modda dahi mekanik limit switch'ler, aşırı basınç kilitleri ve termik röleler mutlaka hareketin önüne kilit (`AND NOT #Limit_Sensor`) olarak konmalıdır. Manuel mod sadece otomatik dizilimi baypas eder, mekanik korumaları asla baypas etmez.