# Akümülatör (Birikimli Toplama) Mantığı

Bir dizi içindeki tüm sayıları toplamak için, her döngüde eski değerin üzerine yeni değeri ekleyen bir "Geçici Toplam" (Akümülatör) değişkeni kullanırız.

Formül: `Toplam = Toplam + Dizi[i]`

**Çok Önemli Kural**: Döngü (FOR) başlamadan önce mutlaka geçici toplam değişkenini `0.0`'a eşitlemeliyiz. Aksi takdirde PLC, bir önceki PLC taramasından (cycle) kalan toplamın üzerine tekrar tekrar ekleme yapar ve sayılar sonsuza doğru patlar.


# Ortalama Hesaplama (Average)

Ortalama = (Dizideki tüm elemanların toplamı) / (Eleman sayısı).
SCL'de bölme işlemi yaparken, eğer sayılarımız ondalıklıysa (REAL), böldüğümüz sayının da `10` yerine `10.0` olarak yazılması veri tipi uyumsuzluğunu önler.


# EX: ARRAY AVERAGE

// 1. DÖNGÜ ÖNCESİ SIFIRLAMA (HAYATİ ADIM)
// Eğer bunu sıfırlamazsak, PLC saniyede 1000 kere döndüğü için toplam değeri sonsuza gider.
#t_rSum := 0.0;

// 2. FOR DÖNGÜSÜ İLE TOPLAMA (SUM)
// Dizinin 1. elemanından 10. elemanına kadar sırayla geziyoruz.
FOR #t_i := 1 TO 10 DO
    // Mevcut toplamın üzerine, dizinin o anki sırasındaki değeri ekle.
    #t_rSum := #t_rSum + #i_aWeights[#t_i];
END_FOR;

// 3. SONUÇLARI ÇIKIŞLARA (HMI'A) AKTARMA
// Toplam değeri doğrudan çıkışa ver.
#q_rTotalSum := #t_rSum;

// Ortalama için Toplamı eleman sayısına böl (Ondalıklı bölme için 10.0 yazıyoruz).
#q_rAverage := #t_rSum / 10.0;


# Good-to-know: 

1- Matematikte bir sayıyı sıfıra bölersen sonuç tanımsızdır; PLC bunu gördüğü an, içine OB121 (Program Error) kalkanı yüklenmemişse anında kendini STOP moduna alır. Bölen sayı değişken ise, daima `IF #Bolen > 0` THEN kalkanı ile korunmalıdır.

2- Döngüler (FOR/WHILE) PLC işlemcisinin düşmanıdır. 10 elemanlı bir döngü mikrosaniyeler sürer, sorun yoktur. Ancak 50.000 elemanlı bir kalite geçmişi dizisini tarayıp ortalama almak istersen, PLC o taramayı bitirene kadar çıkışları (motorları/valfleri) dondurur. Tarama süresi 150 ms'yi (Watchdog limiti) geçerse PLC Kırmızı Işık (STOP) yakar ve fabrikayı durdurur. Büyük dizilerde işlem State Machine ile parçalanır (Örn: 1. taramada ilk 10.000, 2. taramada ikinci 10.000 toplanır).