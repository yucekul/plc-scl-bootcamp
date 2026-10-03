# EX: Modüler Motor FC

// 1. ARIZA (FAULT) YÖNETİMİ
// Termik arıza gelirse veya zaten bir arıza mühürlüyse VE reset'e basılmamışsa arıza devam eder.
#iq_bFaultState := (#i_bThermalFault OR #iq_bFaultState) AND NOT #i_bResetBtn;
#q_bFaultLamp := #iq_bFaultState;

// 2. ÇALIŞMA MODLARI VE KONTROL
IF #iq_bFaultState THEN
    // Eğer sistemde arıza varsa, mod ne olursa olsun motor DURUR. (Güvenlik Önceliği)
    #iq_bMotorState := FALSE;
    
ELSE
    // Arıza yoksa modlara bak.
    IF #i_iMode = 0 THEN
        // MOD 0: Bakım / Stop
        #iq_bMotorState := FALSE;
        
    ELSIF #i_iMode = 1 THEN
        // MOD 1: Manuel Kontrol (Start/Stop Mühürleme)
        #iq_bMotorState := (#i_bStartBtn OR #iq_bMotorState) AND NOT #i_bStopBtn;
        
    ELSIF #i_iMode = 2 THEN
        // MOD 2: Otomatik Kontrol (Sensörden / Üst sistemden gelen komut)
        #iq_bMotorState := #i_bSensor;
        
    ELSE
        // HATA: Tanımsız Mod (Güvenliğe al)
        #iq_bMotorState := FALSE;
    END_IF;
    
END_IF;

// 3. FİZİKSEL ÇIKIŞA AKTARMA
#q_bMotorRun := #iq_bMotorState;


# Good-to-know:

1- NC (Normalde Kapalı) Mantığına Alış: Sahadaki Stop butonları, Termikler, Acil Stoplar, Sınır Şalterleri daima Normalde Kapalı (NC) olarak kablolanır (Kablo koparsa sistem kendini hemen durdursun diye). Bu nedenle PLC'ye hiçbir sorun yokken sürekli TRUE sinyali gelir. SCL yazarken kafa karışıklığını önlemek için sinyalin kopması anını IF NOT i_Termik şeklinde kontrol etmeyi refleks haline getirmelisin.

2- iq_Motor_Kontaktörü := (... OR iq_Motor_Kontaktörü) satırına dikkat et. TIA Portal'da bir Output değişkenini kendi içinde okumak mümkündür ancak bazı eski PLC'lerde veya belirli ayarlar aktifken bu hata verebilir. Motor mühürlemesinde kendi değerini okuman gerektiği için bu bacağı InOut yapmak her zaman en profesyonel standarttır.


# The Code:

#iq_bFaultState := (#i_bThermalFault OR #iq_bFaultState) AND NOT #i_bResetBtn;
#q_bFaultLamp := #iq_bFaultState;

IF #iq_bFaultState THEN
    #iq_bMotorState := FALSE;
    
ELSE
    IF #i_iMode = 0 THEN
        #iq_bMotorState := FALSE;
        
    ELSIF #i_iMode = 1 THEN
        #iq_bMotorState := (#i_bStartBtn OR #iq_bMotorState) AND NOT #i_bStopBtn;
        
    ELSIF #i_iMode = 2 THEN
        #iq_bMotorState := #i_bSensor;
        
    ELSE
        #iq_bMotorState := FALSE;
    END_IF;
    
END_IF;

#q_bMotorRun := #iq_bMotorState;