# Histerezis Ölü Bant Mantığı (Titremeye karşın)

Bir sistemin "Açılma" (Set) noktası ile "Kapanma" (Reset) noktasının birbirinden farklı olması durumudur. Örneğin, 100 derecede açılacak fan 99'da kapanması yerine, 95'te kapansın (Aradaki 5 derece fark hizterezistir). Böylece sistemin sürekli aç kapa yapıp, arızalanmasının önüne geçer.

// 1. SET ŞARTI: Değer set noktasını aşarsa çıkışı Aktif et.
IF #i_rValue >= #i_rSetpoint THEN
    #iq_bState := TRUE;

// 2. RESET ŞARTI: Değer (Set - Ölü Bant) noktasının altına düşerse Pasif et.
ELSIF #i_rValue < (#i_rSetpoint - #i_rDeadband) THEN
    #iq_bState := FALSE;

// 3. ÖLÜ BANT (DEADBAND) BÖLGESİ
// Dikkat: Burada bilerek "ELSE" bloğu kullanmıyoruz!
// Eğer değer Setpoint ile (Setpoint - Deadband) arasındaysa, sistem yukarıdaki iki şarta da girmez ve iq_bState son halini korur.
END_IF;


# Good-to-know:

1- Eğer %5lik dalgalanmalara bile tahammül yoksa, proje için PID kontrolcüler düşünülmeli (oransal olarak sistemi exact bir derecede çalıştırabilir).

2- Bazen proses, sıcaklığın set değerini asla geçmemesini ister. Bu durumda simetrik (Setpoint + Deadband) yerine, asimetrik histerezis kullanılır. Açılışı (Setpoint - Deadband) noktasında, kapanışı ise tam olarak (Setpoint) noktasında yaparsın.