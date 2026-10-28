# TIME (T#) vs DTL (Date and Time Long)

**TIME**: Sadece geçen süreyi (kronometre) ölçer. Timer'larda (TON/TOF) kullanılır (Örn: T#5s).

**DTL**: Tam bir takvim bilgisidir. Yıl, Ay, Gün, Saat, Dakika, Saniye ve Nanosaniye olarak parçalara ayrılmış 12 Byte'lık dev bir yapıdır.


# PLC'den Saati Okuma (RD_SYS_T ve RD_LOC_T)

PLC'nin içinde bir saat pili ve çipi (RTC) vardır.

`RD_SYS_T` (Read System Time): PLC'nin ham sistem saatini (genellikle GMT/UTC 0) okur.

`RD_LOC_T` (Read Local Time): PLC donanım ayarlarından seçtiğiniz yerel saat dilimine (Örn: GMT +1 Berlin) ve yaz/kış saati uygulamasına göre hesaplanmış saati okur. Endüstride her zaman **RD_LOC_T** kullanırız ki operatör ekranındaki saatle kodumuz aynı olsun.


# EX: VARDİYA YÖNETİCİSİ

// 1. PLC'NİN YEREL SAATİNİ OKUMA
// RD_LOC_T komutu, PLC'nin saatini okur ve OUT bacağına bağladığımız DTL değişkenine yazar.
// Komutun çalışıp çalışmadığını gösteren sonuç (Return Value) mecburen bir Temp değişkene atanmalıdır.
#t_iRetVal := RD_LOC_T(#stat_CurrentTime);

// 2. DTL İÇİNDEKİ VERİLERİ PARÇALAMA
// DTL tıpkı bir UDT gibidir. Nokta (.) koyarak içindeki saate ve dakikaya ulaşabiliriz.
#q_iHour   := USINT_TO_INT(#stat_CurrentTime.HOUR);
#q_iMinute := USINT_TO_INT(#stat_CurrentTime.MINUTE);

// 3. VARDİYA (SHIFT) HESAPLAMA MANTIĞI
// Fabrika vardiyaları: 08:00-16:00 (1. Vardiya), 16:00-00:00 (2. Vardiya), 00:00-08:00 (3. Vardiya)

IF #q_iHour >= 8 AND #q_iHour < 16 THEN
    // Saat 08:00 ile 15:59 arasındaysa
    #q_iCurrentShift := 1; // Gündüz Vardiyası

ELSIF #q_iHour >= 16 AND #q_iHour <= 23 THEN
    // Saat 16:00 ile 23:59 arasındaysa
    #q_iCurrentShift := 2; // Akşam Vardiyası

ELSE
    // Geriye kalan tek zaman dilimi: Saat 00:00 ile 07:59 arasıdır
    #q_iCurrentShift := 3; // Gece Vardiyası
    
END_IF;


# Good-to-know:

1- Zaman kontrolleri daima "Eski durum / Yeni durum" (Yükselen Kenar - R_TRIG) ile yapılmalıdır.

2- `RD_LOC_T` fonksiyonunun doğru çalışması için TIA Portal'da Device Configuration -> CPU Properties -> Time of Day sekmesine gidip bulunduğun ülkenin saat dilimini (Timezone) seçmen gerekir. Eğer seçmezsen RD_LOC_T bloğu sana saati yanlış verecektir.

3- Panodaki operatör ekranının (HMI) kendi saati, zamanla PLC'nin saatinden kayar. Genellikle PLC projesine bir "Area Pointer" (Date/Time) eklenir ve ekran kendi saatini düzenli olarak PLC'nin RD_LOC_T verisinden çekip eşitler. Böylece fabrikadaki tüm makineler aynı saniyede çalışır.