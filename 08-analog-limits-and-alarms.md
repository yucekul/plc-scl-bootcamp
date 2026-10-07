# Analog Limit Kontrolleri ve Alarmlar

**High-High (HH)**: Kritik yüksek. Sistem derhal durdurulur (Interlock).

**High (H)**: Yüksek. Sistem çalışmaya devam eder ama operatöre sarı uyarı (Warning) verilir.

**Low (L)**: Düşük. Operatöre uyarı verilir.

**Low-Low (LL)**: Kritik düşük. Sistem güvenliğe alınır (Örn: Pompalar susuz kalmasın diye durdurulur).

// TEMİZ KOD (CLEAN CODE) YAKLAŞIMI İLE LİMİT KONTROLLERİ
// IF kullanmadan doğrudan matematiksel önermeleri çıkışlara atıyoruz.

// 1. Üst Limit Kontrolleri
#q_bAlarmHH  := #i_rValue >= #i_rLimitHH;
#q_bWarningH := #i_rValue >= #i_rLimitH;

// 2. Alt Limit Kontrolleri
#q_bAlarmLL  := #i_rValue <= #i_rLimitLL;
#q_bWarningL := #i_rValue <= #i_rLimitL;


# Good-to-know: 

1- Parazit (Gürültü) Filtreleme - TON (Zamanlayıcı) Kullanımı
Sahada bazen bir sensörün yanından yüksek akımlı bir motor kablosu geçer ve 1 milisaniyeliğine sıcaklığı 500°C okutur. Saniyenin binde biri süren bu "elektriksel gürültü" yüzünden makineyi durdurmamalısın.
Kural: Bir limit aşıldığında alarm vermeden önce mutlaka bir gecikme süresi (Timer - TON) koymalısın (örneğin 2 saniye).