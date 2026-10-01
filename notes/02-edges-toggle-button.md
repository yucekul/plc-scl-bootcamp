# Rising Edge ve Falling Edge

**R_TRIG**: Yükselen kenar (False iken True).
// Manuel olarak
#t_bRisingEdge := #i_bButton AND NOT #iq_bButtonPrev;

**F_TRIG**: Düşen kenar (True iken False).
// Manuel olarak
#t_bFallingEdge := NOT #i_bButton AND #iq_bButtonPrev;

// FC içinde kısa hafıza oluşturmak için şart
#iq_bButtonPrev := #i_bButton;


# Toggle Butonu (XOR)

**Toggle (XOR) mantığı**: 
#iq_bState := #iq_bState XOR #t_bRisingEdge;


# Good-to-know:

1- Hafıza Değişkeni (Memory Bit) Türü: Kenar algılamada kullandığın geçmiş durum değişkeni (Button_Previous_State) kesinlikle kalıcı bir hafıza olmalıdır. Fonksiyon Bloğu (FB) kullanıyorsan Static (STAT) bölümünde, global değişken kullanıyorsan Data Block (DB) veya Marker (M) olarak tanımlanmalıdır. Asla Temp (Geçici) değişken olarak tanımlama, yoksa her döngüde sıfırlanacağı için kenar algılama çalışmaz.

2- Satır Sıralaması: Geçmiş durumu güncellediğin satır (Button_Previous_State := Button;), her zaman pulse ürettiğin matematiksel işlemden sonra gelmelidir. Önce hafızayı güncellersen, Button ile Button_Previous_State her zaman aynı değere sahip olur ve asla pulse (tetik) alamazsın.