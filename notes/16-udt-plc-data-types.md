# UDT Nedir?

Siemens TIA Portal'da "PLC Data Types" olarak geçer. Kendi özel veri tipimizi (Şablon/Kalıp) oluşturmamızı sağlar. Tıpkı `Int` veya `Bool` gibi, kendi yarattığımız `UDT_Motor` tipinde değişkenler üretebiliriz.


# EX: UDT VALVE CONTROL

// 1. ARIZA KONTROLÜ (Mekanik Sıkışma)
// Valf hem açık hem kapalı sensörünü aynı anda görüyorsa mekanik bir arıza vardır. _st: Struct
IF #iq_stValve.bSensorOpened AND #iq_stValve.bSensorClosed THEN
    #iq_stValve.bError := TRUE;
ELSE
    #iq_stValve.bError := FALSE;
END_IF;

// 2. VALF ÇALIŞMA MANTIĞI
// Hata yoksa ve Açma komutu geldiyse çıkışı ver. 
// Kapatma komutu geldiyse veya valf zaten tam açıldıysa (Limit gördüyse) çıkışı kes.
IF NOT #iq_stValve.bError THEN
    IF #iq_stValve.bCmdOpen AND NOT #iq_stValve.bSensorOpened THEN
        #q_bPhysicalOut := TRUE;
    ELSIF #iq_stValve.bCmdClose THEN
        #q_bPhysicalOut := FALSE;
    END_IF;
ELSE
    // Hata durumunda valf enerjisini keserek güvenliğe al
    #q_bPhysicalOut := FALSE;
END_IF;


# Good-to-know:

1- Ekrana (Siemens HMI veya SCADA) tag eklerken her motor için 5'er tag'den toplam 250 tag tanımlamak zorunda kalırsın. Ancak PLC'de UDT kullanırsan, HMI'da da bu UDT'yi (HMI User Data Type) oluşturup, ekran sayfasına (Faceplate) tek bir tag bağlayarak 50 motoru aynı sayfadan kontrol edebilirsin.