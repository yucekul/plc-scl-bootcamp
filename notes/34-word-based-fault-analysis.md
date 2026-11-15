# Genel Arıza (Global Fault) Kontrolü

Elimizde 16 bitlik bir Alarm Word'ü var. Eğer makinede HİÇBİR arıza yoksa, bu Word'ün sayısal değeri tam olarak `16#0000` (Sıfır) olmalıdır. 
Eğer bu Word sıfıra eşit değilse (`<> 16#0000`), bu "Makinede en az 1 tane arıza var" demektir ve sistemi anında FAULT adımına geçirmek için yeterlidir.


# Bitwise AND (VE ile Hedefli Maskeleme)

Bir Word paketinin içinde *sadece* belirli bir alarmın (Örn: 3. Bit olan 'Acil Stop' alarmının) olup olmadığını anlamak için, paketi o bitin Hex karşılığı (`16#0008`) ile `AND` (VE) işlemine sokarız.
`Sonuc := AlarmWord AND 16#0008;`
Eğer `Sonuc` sıfırdan farklıysa, aradığımız o 3. Bit arızası aktif demektir!


# EX: HATA KONTROLÖRÜ

// 1. GENEL ARIZA KONTROLÜ (GLOBAL FAULT)
// Eğer Alarm Paketi sıfır (16#0000) değilse, içindeki 16 bitten en az biri TRUE (1) demektir.
// Bu durumda makinenin enerjisini kes veya State Machine'i FAULT adımına zorla.
IF #i_wAlarmWord <> 16#0000 THEN
    #q_bMachineHalt := TRUE;
END_IF;

// 2. BELİRLİ BİR BİTİ ARAMA (BITWISE AND)
// Varsayalım ki Bit 15 (16#8000) "Acil Stop" gibi makineyi yeniden başlatmayı engelleyen KRİTİK bir hatadır.
// Alarm paketini "16#8000" maskesiyle filtreliyoruz.
// Sadece her ikisinde de 1 olan bitler aşağı düşeceği için, sonuç 16#0000'dan büyükse o bit var demektir!

IF (#i_wAlarmWord AND 16#8000) <> 16#0000 THEN
    #q_bCriticalErr := TRUE;
ELSE
    #q_bCriticalErr := FALSE;
END_IF;

// 3. MAKİNEYİ YENİDEN BAŞLATMA (RESET) MANTIĞI
// Makine kilitlenmişse (Halt=TRUE) ve operatör Reset'e basmışsa:
// Şart: Reset anında içeride hiçbir arızanın kalmamış olması lazım (Word=0).
// Aksi takdirde operatör Reset'e bassa bile makine kilitli kalmaya devam eder!

IF #i_bResetBtn AND (#i_wAlarmWord = 16#0000) THEN
    #q_bMachineHalt := FALSE;
END_IF;


# Good-to-know:

Eğer makinede 16 değil, 320 tane alarm varsa 20 tane farklı Alarm Word oluşturup kodunu çorbaya çevirmiyoruz. Bunun yerine Data Block'ta Array[1..20] of Word oluşturuyoruz. Bu diziyi de HMI'a "Tetikleyici Dizi (Trigger Array)" olarak tek kalemde bağlıyoruz. 320 alarmı sadece 2 saniyede SCADA'ya aktarmış oluruz.