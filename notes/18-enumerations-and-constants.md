# Sihirli Sayı (Magic Number) Nedir?

Kodun içine doğrudan yazılmış, bağlamı olmayan sayılardır (Örn: `IF #Step = 10 THEN`). Bu sayılar kodun okunabilirliğini düşürür ve hata yapma riskini artırır.

# Sabitler (Constants / Enumeration)

TIA Portal'da geleneksel yazılım dillerindeki gibi (C#, Python) katı bir "Enum" komutu yoktur. Bunun yerine **Constants (Sabitler)** sekmesi kullanılır. Sabitler, program çalışırken asla değiştirilemeyen, sadece okunabilirliği artırmak için sayılara giydirilen isim kılıflarıdır.

**Kötü Kod**: `CASE #stat_iStep OF 10:`

**Temiz Kod**: `CASE #stat_iStep OF #FILL:` (Burada TIA Portal arka planda #FILL kelimesinin 10 olduğunu bilir ve öyle derler, PLC hafızasında harf tutulmaz).

Sabitleri (Constants) Global (tüm projede geçerli) veya Local (sadece o FB/FC içinde geçerli) olarak tanımlayabiliriz. Kapsülleme (Encapsulation) kuralı gereği, sadece bu bloğu ilgilendiren sabitleri bloğun arayüzünde (Local) tanımlamak en doğrusudur.


# EX: SIVI KARIŞIM TANKI SIRALAYICISI ENUM

// 1. ZAMANLAYICI (TIMER)
// "Adım 20 ise" demek yerine "Adım Karıştırma (MIX) ise" diyoruz.
#stat_TimerMix(IN := (#stat_iStep = #MIX),
               PT := T#5s);

// 2. STATE MACHINE (DURUM MAKİNESİ)
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
        
    ELSE
        // Bilinmeyen durum koruması
        #stat_iStep := #IDLE;
END_CASE;

// 3. FİZİKSEL ÇIKIŞLAR
// Koda dışarıdan bakan biri bile neyin ne zaman aktif olduğunu anında anlar.
#q_bFillValve  := (#stat_iStep = #FILL);
#q_bMixer      := (#stat_iStep = #MIX);
#q_bDrainValve := (#stat_iStep = #DRAIN);

#q_iCurrentStep := #stat_iStep;


# Good-to-know: 

1- Bir SCADA veya HMI paneli tasarlarken, PLC'deki ENUM text'lerini ekranla senkronize edebilirsin. Böylece SCADA ekranındaki "Makine Durumu" kutusunda sayılar değil, otomatik olarak senin yazdığın ENUM kelimeleri yazar.