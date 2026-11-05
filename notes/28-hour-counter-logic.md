# Neden Hazır Timer Kullanmıyoruz?

TON veya TOF gibi standart PLC zamanlayıcılarının maksimum sayma kapasitesi 24 gün (T#24d_20h_31m_23s_647ms) civarındadır (DINT tabanlı oldukları için). Ancak bir pompa yıllarca çalışabilir. Ayrıca motor durduğunda TON kendini sıfırlar, oysa biz kaldığı yerden devam etmesini istiyoruz.


# Saniye-Dakika-Saat Devretme (Carry-Over) Mantığı

Kendi sayıcımızı kurmak için 1 Hertz'lik (Saniyede 1 vuran) bir pals kullanırız.

Motor çalışıyorsa VE 1 Saniyelik pals geldiyse -> `Saniye = Saniye + 1`

Eğer `Saniye >= 60` olursa -> `Dakika = Dakika + 1` ve `Saniye = 0`

Eğer `Dakika >= 60` olursa -> `Saat = Saat + 1` ve `Dakika = 0`

**Kritik Kural**: Bu değerlerin PLC kapanıp açıldığında sıfırlanmaması (Kalıcı olması) gerekir. Bunun için değişkenler DB içinde 

**Retain** (Kalıcı Hafıza) olarak işaretlenmelidir.


# EX: SAAT SAYACI

// 1. SIFIRLAMA (RESET) İŞLEMİ
// Bakım ekibi rulmanı değiştirdiğinde sayacı sıfırlamak ister.
IF #i_bReset THEN
    #stat_iSeconds := 0;
    #stat_iMinutes := 0;
    #stat_diHours  := 0;
END_IF;

// 2. BİR SANİYELİK PALS ÜRETİCİSİ (1Hz Oscillator)
// Motor çalışıyorsa ve henüz Q çıkışı vermediyse 1 saniye say.
// 1 saniye dolduğunda Q=TRUE olur, kendi kendini FALSE yapar ve döngü baştan başlar.
#stat_Timer1Sec(IN := #i_bMotorRun AND NOT #stat_Timer1Sec.Q,
                PT := T#1s);

// 3. SANİYE SAYACI
// Timer 1 saniyeyi doldurup Q=TRUE sinyali verdiği o anı yakala.
IF #stat_Timer1Sec.Q THEN
    #stat_iSeconds := #stat_iSeconds + 1;
END_IF;

// 4. DEVRETME MANTIĞI (CARRY-OVER)
// 60 Saniye -> 1 Dakika
IF #stat_iSeconds >= 60 THEN
    #stat_iMinutes := #stat_iMinutes + 1;
    #stat_iSeconds := 0;
END_IF;

// 60 Dakika -> 1 Saat
IF #stat_iMinutes >= 60 THEN
    #stat_diHours  := #stat_diHours + 1;
    #stat_iMinutes := 0;
END_IF;

// 5. ÇIKIŞLARA AKTARMA (HMI İçin)
#q_iTotalMins   := #stat_iMinutes;
#q_diTotalHours := #stat_diHours;


# Good-to-know:

1- Bazı HMI panellerinde operatörler saati ve dakikayı ayrı görmek istemez. Tek bir ekranda "45.5 Saat Çalıştı (45 Saat 30 Dakika)" görmek isterler. Şu matematiksel formülle, sayaçtan çıkan verileri tek bir ondalıklı değere çevirip SCADA'ya yollayabiliriz:
// Toplam dakikayı (Saat*60 + Dakika) bulup, 60.0'a bölersek ondalıklı (REAL) saati buluruz!
#q_HMI_Ondalikli_Saat := DINT_TO_REAL( (#s_Saat * 60) + INT_TO_DINT(#s_Dakika) ) / 60.0;