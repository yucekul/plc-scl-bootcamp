# IF ELSIF ELSE

Eğer şu olursa bunu yap, o olmazsa buna bak, hiçbiri olmazsa şunu yap" demenin yoludur.

IF #i_iMode = 0 THEN
    // Mod 0: Stop (Sistem durur)
    #q_bMotorRun := FALSE;
    #q_bError := FALSE;

ELSIF #i_iMode = 1 THEN
    // Mod 1: Manuel (Sensörsüz direkt çalışma)
    #q_bMotorRun := TRUE;
    #q_bError := FALSE;

ELSIF #i_iMode = 2 THEN
    // Mod 2: Otomatik (Sensöre bağlı çalışma)
    #q_bMotorRun := #i_bAutoSensor;
    #q_bError := FALSE;

ELSE
    // HATA: Tanımsız mod değeri (0, 1, 2 dışında bir sayı gelirse)
    // Motoru güvenliğe al (Kapat) ve Hata ver.
    #q_bMotorRun := FALSE;
    #q_bError := TRUE; 
END_IF;


# Good-to-know:

1- Her IF bloğu mutlaka END_IF; ile kapatılmalıdır.

2- IF - ELSIF - ELSE bloğu yukarıdan aşağıya doğru okunur. PLC doğru olan ilk koşulu bulduğunda o bloğun içindeki işlemi yapar ve diğer seçeneklerin hiçbirine bakmadan doğrudan END_IF sonrasına atlar.


# The Code:

IF #i_iMode = 0 THEN
    #q_bMotorRun := FALSE;
    #q_bError := FALSE;

ELSIF #i_iMode = 1 THEN
    #q_bMotorRun := TRUE;
    #q_bError := FALSE;

ELSIF #i_iMode = 2 THEN
    #q_bMotorRun := #i_bAutoSensor;
    #q_bError := FALSE;

ELSE
    #q_bMotorRun := FALSE;
    #q_bError := TRUE; 
END_IF;