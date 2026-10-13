# TOF (Timer Off Delay - Bırakmada Gecikmeli)

IN sinyali TRUE olduğu an çıkış (Q) beklemeden TRUE olur. Asıl olay IN sinyali FALSE olduğunda başlar. Sinyal kesildikten sonra ayarlanan PT süresi kadar çıkışı TRUE tutmaya devam eder, süre bitince FALSE yapar.

Bir büyük motor durduktan sonra soğutma fanının veya yağlama pompasının 2 dakika daha çalışmaya devam etmesi istenen durumlarda kullanılır.


# TP (Timer Pulse - Darbe Zamanlayıcısı)

IN sinyaline anlık bir tetik (örneğin milisaniyelik bir yükselen kenar) gelse bile, çıkış (Q) ayarlanan PT süresi boyunca kesintisiz TRUE kalır. Süre bitmeden IN sinyali gitse de, kalsa da çıkış süreyi tamamlar.

Bir dozajlama valfini (butona sadece anlık basılsa bile) tam olarak 5 saniye boyunca açık tutmak, sıvı dolum makinelerinde hassas miktar ayarlamak için kullanılır.


# EX: TP & TOF VALVE

// 1. DOLUM VALFİ (TP - Timer Pulse)
// Butona anlık basılsa bile i_tFillTime süresi boyunca çalışacak.
#stat_TimerTP(IN := #i_bStartFill,
              PT := #i_tFillTime);

// TP'nin çıkışı direkt olarak valfi kontrol eder.
#q_bValve := #stat_TimerTP.Q;

// 2. KARIŞTIRICI MOTOR (TOF - Timer Off Delay)
// Valf aktifken (TRUE) karıştırıcı hemen çalışır. 
// Valf kapandığı (FALSE olduğu) an, i_tMixDelay süresi boyunca çalışmaya devam eder.
#stat_TimerTOF(IN := #stat_TimerTP.Q,
               PT := #i_tMixDelay);

// TOF'un çıkışı direkt olarak karıştırıcı motoru kontrol eder.
#q_bMixer := #stat_TimerTOF.Q;


# Good-to-know: 

1- Sensör veya buton bozuksa, çok kısa titreşimler yapıyorsa sistemi nasıl korurum?" -> **TON** kullan.

2- Bir mekanizma durduktan sonra, onu destekleyen başka bir şey biraz daha çalışsın mı?" -> **TOF** kullan. (Soğutma fanı, taşıma bandı boşalana kadar dönen motor).

3- Kullanıcı sadece başlatsın, işlem süresine hiç müdahale edemesin, tam X saniye sürsün" -> **TP** kullan. (Dozaj pompası, otomatik kapı kilidi açma, kaynak robotu ateşleme süresi)