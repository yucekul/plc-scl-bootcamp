# Sensör İsimlendirme Standartları

Pnömatik silindirlerin arkasında ve önünde manyetik limit switch'ler (sensörler) bulunur.
**Silindir A Geri Sensörü**: LimA0 (A0 da denir)
**Silindir A İleri Sensörü**: LimA1 (A1 de denir)


## State Machine (Durum Makinesi) Geçiş Mantığı

`CASE OF` yapısında her adım bir sayıya (State) atanır. (0, 10, 20, 30 gibi. Arada boşluk bırakırız ki ileride araya adım eklemek istersek 11, 12 diye yerimiz olsun).
Bir State'ten diğerine geçmek için iki şeye ihtiyaç vardır:
**Hareket Emri**: O anki State içinde valfe (bobine) enerji verilir (TRUE).
**Geçiş Şartı (Transition)**: Piston fiziksel olarak hedefe ulaştığında (Sensör = TRUE olduğunda), bir sonraki State numarasına atlanır (`State := 20;`).


# EX: CYLINDER SEQUENCE

// SIRA KONTROLÜ (STATE MACHINE - FSM)
CASE #stat_iState OF
        
    0: // HAZIR VE BEKLEME DURUMU
        // Valflerin enerjisini kesik (güvenli) tutuyoruz.
        #q_bValveA := FALSE;
        #q_bValveB := FALSE;
        
        // Geçiş Şartı: Start'a basıldıysa VE iki piston da başlangıç (home) noktasındaysa başla!
        IF #i_bStart AND #i_bLimA0 AND #i_bLimB0 THEN
            #stat_iState := 10;
        END_IF;
        
    10: // ADIM 1: A+ (A Silindirini İleri İt)
        #q_bValveA := TRUE;
        
        // Geçiş Şartı: A silindiri ileri ulaştı sensörüne (A1) çarptı mı?
        IF #i_bLimA1 THEN
            #stat_iState := 20;
        END_IF;
        
    20: // ADIM 2: B+ (B Silindirini İleri İt)
        // (A valfi enerjili kalmaya devam ediyor, yay geri çekmesin diye ellemeyiz)
        #q_bValveB := TRUE;
        
        // Geçiş Şartı: B silindiri ileri ulaştı sensörüne (B1) çarptı mı?
        IF #i_bLimB1 THEN
            #stat_iState := 30;
        END_IF;
        
    30: // ADIM 3: B- (B Silindirini Geri Çek)
        // B Valfinin enerjisini keseriz, mekanik yay silindiri geri çeker.
        #q_bValveB := FALSE;
        
        // Geçiş Şartı: B silindiri başarıyla geri dönüp A0 sensörüne çarptı mı?
        IF #i_bLimB0 THEN
            #stat_iState := 40;
        END_IF;
        
    40: // ADIM 4: A- (A Silindirini Geri Çek)
        // A Valfinin enerjisini keseriz, mekanik yay silindiri geri çeker.
        #q_bValveA := FALSE;
        
        // Geçiş Şartı: A silindiri başarıyla geri dönüp başlangıç sensörüne çarptı mı?
        IF #i_bLimA0 THEN
            // Çevrim tamamlandı! Makineyi yeniden başa döndür.
            #stat_iState := 0; 
        END_IF;
        
    ELSE
        // Bilinmeyen bir hata olursa güvenli konuma geç
        #stat_iState := 0;
        
END_CASE;


# Good-to-know:

1- Silindir saniyede 1 metre hızla demire çarptığında (strok sonu), mekanik sarsıntıdan dolayı üzerindeki manyetik (Reed) sensör milisaniyeler içinde 1-0-1-0 yapar. Eğer arka arkaya çok hızlı geçen iki adımın varsa, bu titreme sistemin yanlışlıkla iki adım birden atlamasına (Double-step) sebep olabilir. Sensör girişlerini Debounce (TON) filtresinden geçirmeden asla CASE bloğuna sokmuyoruz!