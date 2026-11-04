# CPU Clock Memory vs. SCL Blinker

**CPU Clock Memory**: Hızlıdır ama frekansları sabittir (1Hz, 2Hz, 10Hz vb.). İhtiyacınıza göre "1 saniye yansın, 0.5 saniye sönsün" diyemezsiniz. Donanıma özeldir, başka marka PLC'ye kopyalanamaz.

**SCL Blinker (Custom FB)**: İki adet TON (Çekmede Gecikmeli) zamanlayıcının birbirini tetikleyip sıfırlaması mantığıyla çalışır. Açık kalma ve kapalı kalma süreleri (Pulse ve Pause time) ayrı ayrı ayarlanabilir. Modülerdir (Clean Code).


# İki Timer'ın Birbirini Sıfırlaması (Osilatör Mantığı)

**Timer_ON**: Lambanın yanık kalacağı süreyi sayar.

**Timer_OFF**: Lambanın sönük kalacağı süreyi sayar.

Timer_OFF süresini bitirdiği (Q = TRUE) o milisaniyede, Timer_ON'un IN bacağını FALSE yaparak sistemi sıfırlar ve sonsuz bir "Yan-Sön" (Osilatör / Flip-Flop) döngüsü başlatır.


# EX: BLINKER

// 1. AÇIK KALMA (ON) KRONOMETRESİ
// Eğer sistem aktifse (i_bEnable) VE Kapalı kalma süresi henüz dolmadıysa say!
// NOT #stat_TimerOFF.Q kısmı, döngüyü sıfırlayan anahtar noktadır.
#stat_TimerON(IN := #i_bEnable AND NOT #stat_TimerOFF.Q,
              PT := #i_tTimeOn);

// 2. KAPALI KALMA (OFF) KRONOMETRESİ
// Açık kalma süresi dolduğu an (TimerON.Q = TRUE) bu kronometre saymaya başlar.
// Bu süre de dolduğunda (TimerOFF.Q = TRUE), yukarıdaki satır gereği TimerON kendini sıfırlar, 
// TimerON sıfırlanınca TimerOFF da sıfırlanır ve döngü baştan başlar!
#stat_TimerOFF(IN := #stat_TimerON.Q,
               PT := #i_tTimeOff);

// 3. ÇIKIŞIN (LAMBANIN) DURUMU
// Sistem aktifse VE (Henüz ON süresi sayıyorsa) lamba yanar.
// ON süresi dolduğu an lamba söner ve OFF süresi saymaya başlar.
#q_bOut := #i_bEnable AND NOT #stat_TimerON.Q;


# Good-to-know: 

1- PLC'de Timer'lar daima her tarama döngüsünde güncellenmek zorundadır. Onları IF içine hapsedersen, süreleri "donar" ve flaşörün kilitlenir.