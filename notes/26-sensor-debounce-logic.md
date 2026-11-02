# Bouncing (Titreme) Nedir?

Mekanik kontakların kapanırken veya açılırken yaylanması sonucu, PLC girişine çok hızlı "Aç-Kapa" sinyallerinin gitmesidir. Sayıcıların (Counters) veya State Machine'lerin (CASE OF) düşmanıdır.


# Debounce Mantığı

Bir sensörün "Gerçekten geldiğine" (TRUE) veya "Gerçekten gittiğine" (FALSE) inanmadan önce belirli bir süre (Örn: 50 milisaniye) bekleriz.

**On-Delay (TON)**: Sensör TRUE olduysa, hemen sistemi tetikleme. Gerçekten X ms boyunca kesintisiz TRUE kalırsa sinyali kabul et.
**Off-Delay (TOF)**: Sensör FALSE olduysa (Gittiyse), hemen sistemi kesme. Gerçekten X ms boyunca kesintisiz FALSE kalırsa sinyali düşür.

Biz bu ikisini birleştirip, endüstride çok sık kullanılan "Sensör Kararlılık Filtresi" (Debounce FB) yazacağız.


# EX: SENSOR DEBOUNCE

// 1. AÇILMA KARARLILIĞI (TON - Çekmede Gecikme)
// Sensör geldiğinde (TRUE olduğunda), aralıksız i_tDebounceTime kadar bekler.
// Eğer süre dolmadan sensör titreyip FALSE olursa, TON kendini sıfırlar.
#stat_TimerTON(IN := #i_bRawSensor,
               PT := #i_tDebounceTime);

// 2. KAPANMA KARARLILIĞI (TOF - Düşmede Gecikme)
// Sensör gittiğinde (FALSE olduğunda), aralıksız i_tDebounceTime kadar bekler.
// Ancak biz bu TOF'u, TON'un ürettiği temiz çıkış üzerinden besleyeceğiz.
// Böylece sinyal önce TON'dan, sonra TOF'tan geçmiş (Seri bağlanmış) olacak.
#stat_TimerTOF(IN := #stat_TimerTON.Q,
               PT := #i_tDebounceTime);

// 3. FİZİKSEL ÇIKIŞ
// En son filtre olan TOF'un çıkışını nihai "Temiz Sinyal" olarak veriyoruz.
#q_bCleanSignal := #stat_TimerTOF.Q;


# Good-to-know:

1- 50 farklı sensörümüz varsa, 50 farklı TON bloğu için Global DB açmak (Single Instance) projeyi çöplüğe çevirir. 11. Gün FB konusunda öğrendiğimiz gibi, bu debounce TON/TOF bloklarını daima yazacağımız "FB_Sensor_Okuyucu" bloğunun Static hafızasına gömmeli (Multi-Instance) ve tek bir blokla tüm sensörleri filtrelemeliyiz.

2- Güvenlik sinyallerine (Işık bariyeri, Acil Stop butonu, Kapı sivici) asla Debounce veya filtre süresi konmaz! Bu sinyaller 0 olduğu o ilk milisaniyede ana kontaktörü kestirmelidir. Aksi takdirde eklediğin 100 ms'lik gecikme, presin altında birinin elinin ezilmesine sebep olabilir.