# Array (Dizi) Nedir?

Aynı veri tipindeki birden fazla değişkeni tek bir isim altında kutucuklar (indeksler) halinde saklayan veri yapısıdır. `ValveStatus : Array[1..50] of Bool` tanımlaması, size 50 odalı bir apartman verir. `ValveStatus[3]` diyerek 3. vananın bilgisine ulaşırsınız.


# FOR Döngüsü (FOR Loop)

Bir işlemi belirli bir sayıda tekrar etmek için kullanılır. Dizilerle mükemmel bir uyum içinde çalışır. Eğer 50 vanayı kapatmak isterseniz 50 satır kod yazmazsınız; 3 satırlık bir FOR döngüsü yazarsınız.


# EX: FOR ARRAY VALVE MANAGER

// Her döngü başında açık vana sayacını sıfırla.
#q_iOpenValveCount := 0;

// FOR DÖNGÜSÜ BAŞLANGICI (1'den 10'a kadar dön)
FOR #t_iIndex := 1 TO 10 DO
    
    // 1. ACİL STOP KONTROLÜ
    // Eğer acil stop geldiyse, dizideki tüm vanaları sırayla FALSE (Kapalı) yap.
    IF #i_bEmergencyStop THEN
        #iq_bValves[#t_iIndex] := FALSE;
    END_IF;
    
    // 2. AÇIK VANALARI SAYMA
    // Eğer vana açıksa (TRUE), sayacı 1 artır.
    IF #iq_bValves[#t_iIndex] THEN
        #q_iOpenValveCount := #q_iOpenValveCount + 1;
    END_IF;

END_FOR;


# Good-to-know:

 1- PLC'lerde (özellikle OB1 tarama süresi/Cycle Time) çok uzun FOR döngüleri (örn: 10.000 dönüş) yazmak CPU'yu yorar ve Watchdog hatası verdirip PLC'yi durdurabilir (STOP modu). Endüstride döngüler genellikle 100-200 elemanı geçmez.

 2- FOR döngüsünün sayacı olan #i değişkeni geçmişten bir değer taşımaz; her PLC taramasında baştan sayar. Bu yüzden #i değişkenini her zaman bloğun Temp (Geçici) hafızasında ve INT (veya DINT) tipinde tanımlamalısın. Asla Static veya Global DB harcama.


 3- PLC, FOR döngüsünün tamamını tek bir tarama döngüsü (aynı milisaniye) içinde bitirmeden alt satıra geçmez. Eğer döngüyü çok büyük yaparsan (örneğin 1'den 1.000.000'a kadar), PLC işlemi vaktinde bitiremez ve "Cycle Time Exceeded" (Watchdog) hatası vererek tüm fabrikayı durdurur!

 4- Toplamı for döngüsünün üzerine yazmazsan, PLC her milisaniyede eski toplamın üzerine yeni sıcaklıkları eklemeye devam eder ve sayı saniyeler içinde milyonlara ulaşır.