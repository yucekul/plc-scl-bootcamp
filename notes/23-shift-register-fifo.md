# Shift Register Nedir?

Bir dizinin (Array) içindeki verileri, her tetikleme (Pulse) geldiğinde bir sonraki adrese kaydırma işlemidir. Bantın başındaki tekerlek (Encoder) döndükçe PLC'ye bir sinyal (Pulse) yollar. Gelen her sinyalde Array'deki veriler 1 adım yana kayar.


# Altın Kural: Sondan Başa Döngü (DOWNTO)

Array kaydırma işlemi yaparken FOR döngüsünü `1'den 10'a` doğru YAZAMAYIZ.
Eğer yazarsak: Array[2] = Array[1] olur, sonra Array[3] = Array[2] olur... Sonuçta dizideki tüm elemanlar 1. elemanın kopyasına dönüşür!
Önce 10. sıraya 9'u kopyala. Sonra 9'a 8'i kopyala... En son 1. sıraya banttan gelen yeni ürünü yaz. Bunun için SCL'de `BY -1` (Geriye doğru say) ifadesi kullanılır.


# EX: BANT TAKİP SİSTEMİ (CONVEYOR TRACKER)

// 1. ENKODER KENAR TETİKLEMESİ (R_TRIG Mantığı)
// Bant kayarken gelen sinyalin sadece yükselen kenarını (FALSE -> TRUE) yakalarız.
#t_bPulseEdge := #i_bEncoderPulse AND NOT #stat_bPrevPulse;
#stat_bPrevPulse := #i_bEncoderPulse;

// 2. SHIFT REGISTER (KAYDIRMA) İŞLEMİ
IF #t_bPulseEdge THEN
    
    // Altın Kural: Döngü SONDAN BAŞA doğru (10'dan 2'ye) dönmeli!
    // SCL dilinde geriye saymak için "BY -1" komutunu kullanırız.
    FOR #t_i := 10 TO 2 BY -1 DO
        #stat_aBeltInfo[#t_i] := #stat_aBeltInfo[#t_i - 1];
    END_FOR;
    
    // Bantın en başı (1. indeks) boşa çıktı. Yeni gelen ürünü oraya koy.
    #stat_aBeltInfo[1] := #i_iNewProduct;
    
END_IF;

// 3. FİZİKSEL ÇIKIŞ (REJECT İSTASYONU)
// Piston 8. adımda duruyor. Eğer 8. kutucuğa gelen ürün kodu '2' (Bozuk) ise pistonu vur.
#q_bRejectPiston := (#stat_aBeltInfo[8] = 2);


# Good-to-know:

1- Eğer kaydırmayı bir mekanik switch veya bant üzerindeki bir fotosel yapıyorsa, sensörün titremesi (anlık 0-1-0 dalgalanması) 1 saniye içinde diziyi 50 adım kaydırabilir. Tetiği mutlaka TON (gecikmeli) filtreden geçirerek net bir pulse (R_TRIG) ile almalısın.

2- Eğer 10.000 elemanlı bir FIFO yapıyorsan, her eleman çekildiğinde 10.000 elemanı 1 adım öne kaydırmak CPU tarama süresini (Watchdog) patlatabilir. Bu gibi durumlarda verileri yerinden oynatmak yerine; sadece "Okuma Noktası (Head Pointer)" ve "Yazma Noktası (Tail Pointer)" adı verilen iki #i değişkeni dairesel olarak kaydırılır. (Buna Circular Buffer / Ring Buffer denir).