# Makine Neden Adımda Takılı Kalır?

1- Sensör arızası (Kablo kopması, sensörün kirlenmesi).

2- Mekanik sıkışma (Valfe komut gider ama valf fiziksel olarak açılamaz).

3- Pnömatik hava basıncı kaybı.


# Watchdog (Bekçi Köpeği) Timer Mantığı

Makinemiz bir adıma girdiğinde gizli bir kronometre başlar. Her adımın mantıklı bir tamamlanma süresi vardır (Örn: Tank en fazla 30 saniyede dolmalıdır). Eğer makine aynı adımda 30 saniyeden fazla kalırsa kronometre alarm verir, sistemi güvenli olan "Arıza" adımına (FAULT) geçirir ve operatör gelip sorunu çözene kadar sistemi kilitler.


# 1-Cycle (Tek Çevrim) Reset Taktiği

TON zamanlayıcısını her adım değişiminde sıfırlamak için, adım numarasının değiştiği o milisaniyelik anı yakalayıp Timer'ın IN bacağını sadece 1 PLC çevrimi (Cycle) boyunca FALSE yapıp geri TRUE yapmamız gerekir.

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

// 5. FİZİKSEL ÇIKIŞ ATAMALARI
#q_bFillValve  := (#stat_iStep = #FILL);
#q_bMixer      := (#stat_iStep = #MIX);
#q_bDrainValve := (#stat_iStep = #DRAIN);

#q_bTimeoutAlarm := (#stat_iStep = #FAULT);
#q_iCurrentStep  := #stat_iStep;


# Good-to-know:

1- Bir makine hata verdiğinde kodun çalışmasını (PLC taramasını) öylece durduramazsın. Makineyi, tüm valflerin kapandığı, motorların durduğu ve sadece kırmızı alarmların yandığı özel bir Hata Adımına (Error State) yönlendirmek zorundasın. Eğer sadece #Adim_No'yu olduğu yerde dondurursan, motor ileri gitmeye zorlanmaya devam edebilir.

2- Bazı büyük vanaların (örneğin buhar vanaları) açılması kış aylarında 10 saniye, yaz aylarında 4 saniye sürebilir. Limitleri çok daraltırsan (Örn: 5 saniye), makine kışın sürekli sahte arıza verir. Tolerans sürelerini (PT) sabit yazmak yerine HMI ekranından girilebilir bir değişkene (#i_Vana_Acma_Toleransi) bağlamalısın.

3- Operatörler makinenin neden beklediğini anlamazlar. ton_Adim_Suresi.ET (Geçen Zaman) çıkışını INT/DINT'e çevirip ekrana yansıtırsan, operatör ekranda "Sensör Bekleniyor... 3.2s / 5.0s" şeklinde bir bar veya geri sayım görür. Bu, makinenin kalitesini devasa oranda artırır. 