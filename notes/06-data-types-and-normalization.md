# Sayısal Veri Tipleri ve Tip Dönüşümleri

**INT (Integer)**: 16-bit tam sayıdır (-32768 ile +32767). Siemens PLC'lerde analog modüllerden gelen ham veriler (Örn: 4-20mA = 0-27648) her zaman INT formatındadır. Küsurat tutamaz.

**REAL**: 32-bit kayan noktalı (ondalıklı) sayıdır (Örn: 24.5 °C). Sensör okumaları ve mühendislik birimleri REAL ile hesaplanır.

**(NORM_X)**: (Ham Değer - Min) / (Max - Min)

**TRUNC**: Yuvarlama yapmadan ondalığı atmak istiyorsan kullanabilirsin (2.9'u, 2 yapar).


# EX: NORMX (Normalize)

// 1. GÜVENLİK: Sıfıra Bölme (Division by Zero) Hatasını Engelle
// Eğer Max ve Min değerleri yanlışlıkla aynı girilirse, PLC CPU'su "Fault" verip stopa geçer!
IF #i_iMax = #i_iMin THEN
    #q_rNormOut := 0.0;
    
ELSE
    // 2. TİP DÖNÜŞÜMÜ VE MATEMATİKSEL HESAPLAMA
    // Bölme işlemi yapacağımız için INT değerleri INT_TO_REAL ile ondalıklı sayıya (REAL) çeviriyoruz.
    #t_rResult := (INT_TO_REAL(#i_iRawValue) - INT_TO_REAL(#i_iMin)) / 
                  (INT_TO_REAL(#i_iMax) - INT_TO_REAL(#i_iMin));
                  
    // 3. SINIRLANDIRMA (LIMIT)
    // Sensör kablosu koparsa veya kısa devre olursa değer 27648'i aşabilir (Örn: 32767).
    // Oranın %100'ü (1.0) veya %0'ı (0.0) geçmemesini garanti altına alıyoruz.
    IF #t_rResult > 1.0 THEN
        #q_rNormOut := 1.0;
    ELSIF #t_rResult < 0.0 THEN
        #q_rNormOut := 0.0;
    ELSE
        #q_rNormOut := #t_rResult;
    END_IF;
    
END_IF;


# Good-to-know:

1- SCL'de Bölme İşlemi Tuzağı: İki INT sayıyı bölersen, PLC küsuratı çöpe atar.
Sonuc_REAL := 5 / 2; yazarsan cevap 2.5 çıkmaz, 2.0 çıkar!
Doğru hesap için PLC'ye sayıların REAL olduğunu belirtmelisin: Sonuc_REAL := 5.0 / 2.0;

2- Sabit Sayılarda Nokta Kullanımı: SCL'de bir formül yazarken sayı ondalıklı olmasa bile REAL bir işlem yapıyorsan sonuna .0 koymayı alışkanlık haline getir (Örn: Faktor * 10.0). Bu, PLC'nin "tip uyuşmazlığı" hatası vermesini engeller.

3- NORM_X Sınır Taşması (Overflow Risk): Eğer sensör arızalanır veya kısa devre olursa, PLC'ye gelen değer 27648'i geçebilir (örneğin 30000). Bu durumda NORM_X sana 1.0'dan büyük bir oran (örn 1.08) verir. Bu yüzden analog okumaların altına veya üstüne genellikle bir limit kontrolü (IF Oran > 1.0 THEN Oran := 1.0) eklenir.


# The Code:

IF #i_iMax = #i_iMin THEN
    #q_rNormOut := 0.0;
    
ELSE
    #t_rResult := (INT_TO_REAL(#i_iRawValue) - INT_TO_REAL(#i_iMin)) / 
                  (INT_TO_REAL(#i_iMax) - INT_TO_REAL(#i_iMin));
    IF #t_rResult > 1.0 THEN
        #q_rNormOut := 1.0;
    ELSIF #t_rResult < 0.0 THEN
        #q_rNormOut := 0.0;
    ELSE
        #q_rNormOut := #t_rResult;
    END_IF;
    
END_IF;