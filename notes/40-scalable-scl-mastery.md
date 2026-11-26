# Sihirli Sayılardan (Magic Numbers) Kurtulma: `Constant`

Kodun içinde 5 farklı yerde `TO 6` yazmak yerine, FB arayüzünün `Constant` kısmında tek bir sabit (`c_iZoneCount := 6`) tanımlanır. Yarın hatta 6 yerine 12 bölge eklendiğinde kodun içindeki mantık tek satır değişmeden büyür.


# Multi-Instance Dizisi (`Array of TON_TIME`)

Her bölge için ayrı zamanlayıcı açmak yerine `stat_aTimers : Array[1..6] of TON_TIME` tanımlanır. Tek bir `FOR` döngüsü her PLC taramasında tüm bölgelerin zamanlayıcı, mühürleme, bypass, reset ve HMI paketleme işlerini paralel olarak yürütür.


# First-Out (İlk Arıza) Kilidi

Zincirleme arızalarda kök sebebi kaçırmamak için kullanılan endüstriyel kalıptır:
`IF #stat_iFirstZone = 0 THEN #stat_iFirstZone := #t_i; END_IF;`
İlk arıza kaydedildiği an değer sıfırdan farklı olacağı için sonraki ikincil arızalar bu kaydı ezemez; ancak tüm bölgeler tamamen temizlenip sıfırlandığında (`t_iFaultCount = 0`) tekrar `0` olur.


# EX: KONVEYOR SIKIŞMA SİSTEMİ

// 1. HER TARAMA BAŞINDA GEÇİCİ DEĞERLERİ SIFIRLA
#t_iFaultCount := 0;
#t_wAlarmWord  := 16#0000;

// 2. ÖLÇEKLENEBİLİR ÇOKLU BÖLGE DENETİM DÖNGÜSÜ
FOR #t_i := 1 TO #c_iZoneCount DO
    
    // A. MULTI-INSTANCE ZAMANLAYICI ÇAĞRISI
    // Sıkışma SADECE bant dönüyorsa, sensör doluysa VE o bölge Bypass değilse sayılır.
    #stat_aTimers[#t_i](IN := #i_bConveyorRun AND #i_aSensors[#t_i] AND NOT #i_aZoneBypass[#t_i],
                        PT := #i_tJamLimit);
    
    // B. ARIZA MÜHÜRLEME (LATCH) VE İLK ARIZA (FIRST-OUT) KİLİDİ
    IF #stat_aTimers[#t_i].Q THEN
        #stat_aLatched[#t_i] := TRUE;
        
        // Henüz başka bir bölge arıza kaydetmediyse, olayı başlatan KÖK SEBEP budur!
        IF #stat_iFirstZone = 0 THEN
            #stat_iFirstZone := #t_i;
        END_IF;
    END_IF;
    
    // C. GÜVENLİ RESET KONTROLÜ
    // Kutu fiziksel olarak alındıysa VEYA bakım ekibi o sensörü Bypass ettiyse temiz sayılır.
    #t_bZoneClear := (NOT #i_aSensors[#t_i]) OR #i_aZoneBypass[#t_i];
    
    IF #i_bReset AND #t_bZoneClear THEN
        #stat_aLatched[#t_i] := FALSE;
    END_IF;
    
    // D. ÇIKIŞ AKTARIMI, SAYIM VE HMI ALARM PAKETLEME (GÜN 33: SHL + OR)
    #q_aZoneFault[#t_i] := #stat_aLatched[#t_i];
    
    IF #stat_aLatched[#t_i] THEN
        #t_iFaultCount := #t_iFaultCount + 1;
        // 1. bölge -> Bit 0 (16#0001), 4. bölge -> Bit 3 (16#0008) olarak Word'e yazılır.
        #t_wAlarmWord  := #t_wAlarmWord OR SHL(IN := 16#0001, N := #t_i - 1);
    END_IF;
    
END_FOR;

// 3. GENEL HAT KARARI VE FIRST-OUT SIFIRLAMA
// Tüm bölgeler tamamen temizlendiyse "İlk Arıza" hafızasını da sıfırla.
IF #t_iFaultCount = 0 THEN
    #stat_iFirstZone := 0;
END_IF;

#q_bLineStop   := (#t_iFaultCount > 0);
#q_iFaultCount := #t_iFaultCount;
#q_iFirstZone  := #stat_iFirstZone;
#q_wAlarmWord  := #t_wAlarmWord;


# Good-to-know:

1- `#c_iZoneCount` gibi sistem kapasitesini belirleyen sabitleri blok içi (Local) yerine Global User Constant olarak tanımlamak, hem dışarıdaki veri bloklarını hem de `#stat_aTimers` gibi Multi-Instance dizilerini tek merkezden senkronize büyütmeni sağlar. Ancak bu sabit büyütülüp çalışan makineye yüklendiğinde Instance DB yeniden boyutlanacağı için Re-Initialize (aktif verilerin sıfırlanması) yaşanır; değişiklik mutlaka makine güvenli duruştayken yapılmalıdır.

2- Sabit sayesinde kodun kolayca 20–30 bölgeye ölçeklenebilir, ancak `SHL` ile paketlenen tek bir `#t_wAlarmWord` (16-bit) yalnızca ilk 16 bölgeyi taşır; sonrası taşarak kaybolur. 16 üzeri genişleme ihtimali olan hatlarda `DWord` veya `Array of Word` kullanılmalı, `#q_wAlarmWord` doğrudan Siemens WinCC HMI alarm tablosuna bağlanıyorsa Byte sıralaması için SWAP unutulmamalıdır.

3- Bakım amaçlı `#i_aZoneBypass` girişleri arızalı sensör yüzünden üretimin durmasını engeller; ancak sahada unutulduğunda makineyi korumasız bırakır. En az bir bölgede Bypass aktifken HMI üzerinde sürekli görünür bir genel uyarı biti üretmek (ve mümkünse vardiya/reset sonrası Bypass'ı hatırlatmak) endüstri standardıdır.

4- Döngü bazlı First-Out (`#stat_iFirstZone`) yapılarında tarama her zaman 1'den başlar; bu yüzden tüm bölgeleri aynı anda düşüren genel hat arızaları (enerji, hava veya ana stop) döngüden önce yakalanmazsa sistem haksız yere 1. bölgeyi suçlar. Ayrıca `#i_bReset` gibi sıfırlama girişleri takılı kalma ihtimaline karşı daima yükselen kenar (`R_TRIG`) ile okunmalıdır.

5- Multi-Instance Array (`#stat_aTimers[#t_i]`) kullanan döngülerde TIA Portal'da online izleme açtığında her bölgenin anlık durumunu kod satırında tek tek göremezsin. Sahada belirli bir bölgenin zamanlayıcısını veya `#stat_aLatched` kilidini izlemek için doğrudan Instance DB'yi açıp ilgili dizi indeksini takip etmek gerekir.