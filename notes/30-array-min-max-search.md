# "Kral Benim" (King of the Hill) Algoritması

Bir dizi içinde en büyük (Max) değeri bulmak için şu mantık kurulur:

Başlangıçta dizinin **1. elemanı "Kral" (Max)** ilan edilir.
Döngü ile 2. elemandan 10. elemana kadar herkes tek tek Kral ile karşılaştırılır (Savaşır).
Eğer meydan okuyan eleman, Kral'dan (mevcut Max değerinden) **büyükse**, tacı elinden alır ve yeni Kral kendisi olur. Aynı anda kendi indeksini de (kaçıncı sıradan geldiğini) "Kralın İndeksi" olarak kaydeder.
Döngü bittiğinde tahtta kim oturuyorsa en büyük odur.

En küçük (Min) değer için de aynı mantık geçerlidir, sadece şart "<" (küçüktür) olarak değiştirilir.


# EX: ARRAY MIN MAX

// 1. BAŞLANGIÇ DURUMU (1. Elemanı Kral İlan Et)
// Hem Max hem de Min için şimdilik dizinin 1. elemanını varsayılan (Default) seçiyoruz.
#t_rCurrentMax := #i_aWeights[1];
#t_iIdxMax := 1;

#t_rCurrentMin := #i_aWeights[1];
#t_iIdxMin := 1;

// 2. MEYDAN OKUMA DÖNGÜSÜ (2. elemandan başla, 10'a kadar git)
FOR #t_i := 2 TO 10 DO
    
    // EN BÜYÜĞÜ BULMA (MAX)
    // Eğer sıradaki rakip, mevcut Kral'dan büyükse:
    IF #i_aWeights[#t_i] > #t_rCurrentMax THEN
        #t_rCurrentMax := #i_aWeights[#t_i]; // Yeni Kral (Max) o oldu
        #t_iIdxMax := #t_i;                  // Sırasını da kaydet
    END_IF;
    
    // EN KÜÇÜĞÜ BULMA (MIN)
    // Eğer sıradaki rakip, mevcut En Küçük'ten bile daha küçükse:
    IF #i_aWeights[#t_i] < #t_rCurrentMin THEN
        #t_rCurrentMin := #i_aWeights[#t_i]; // Yeni Min o oldu
        #t_iIdxMin := #t_i;                  // Sırasını da kaydet
    END_IF;
    
END_FOR;

// 3. SONUÇLARI DIŞARI AKTARMA (HMI İçin)
#q_rMaxVal   := #t_rCurrentMax;
#q_iMaxIndex := #t_iIdxMax;
#q_rMinVal   := #t_rCurrentMin;
#q_iMinIndex := #t_iIdxMin;


# Good-to-know:

1- Örneğin; konveyördeki 3. kantar tam 50.0 kg ölçtü, biraz sonra gelen 7. kantar da tam 50.0 kg ölçtü. Koddaki büyüktür `>` işareti yüzünden; 7. kantar, 3. kantarın rekorunu ezip tahta oturamaz. En ağır ürünün indeksi HMI ekrana `#q_iMaxIndex = 3` olarak gider. Makinenin çalışma mantığına göre karar vermeliyiz. Eğer her zaman en son geçen ağır ürünü hattan dışarı itmek istiyorsan, kodu `>=` (Büyük Eşittir) yapmalıyız ki tahtı daima güncel (sonradan gelen) eleman devralsın.