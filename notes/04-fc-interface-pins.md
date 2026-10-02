# FC Bacakları Mantığı

**Input (IN)**: Kod içinde inputa değer atanamıyor.

**Output (OUT)**: Denklemin sağ tarafında kullanılamıyor.

**InOut (IN_OUT)**: Mühürleme devreleri için kullanılabilir( %M veya DB'den bir referans değeri alır, hafıza için her zaman buna ihtiyacımız var).

**Temp**: Geçici hafıza (Tek cycle sonunda sıfırlanır).


# EX: Güvenlik Valfi

// 1. GÜVENLİK (INTERLOCK) KONTROLÜ
// Eğer güvenlik şartı sağlanmıyorsa (FALSE ise) valf hiçbir şekilde açılamaz.
IF NOT #i_bInterlock THEN
    #iq_bValveState := FALSE; // Valfi zorla kapat
    #q_bError := TRUE;        // Hata ver
    
ELSE
// Güvenlik sağlandıysa hata çıkışını temizle ve normal çalışmaya izin ver.
    #q_bError := FALSE;
    
// 2. MÜHÜRLEME MANTIĞI
// iq_bValveState bacağı IN_OUT olduğu için hem okuyabiliyor (OR iq_...) hem yazabiliyoruz (iq_... :=)
    #iq_bValveState := (#i_bOpenBtn OR #iq_bValveState) AND NOT #i_bCloseBtn;
    
END_IF;

// 3. FİZİKSEL ÇIKIŞA AKTARMA
// Mühürleme sonucunda elde edilen durumu, Output bacağına aktarıyoruz.
     #q_bValveOut := #iq_bValveState;


# The Code:

IF NOT #i_bInterlock THEN
    #iq_bValveState := FALSE; 
    #q_bError := TRUE;        

ELSE
    #q_bError := FALSE;
    #iq_bValveState := (#i_bOpenBtn OR #iq_bValveState) AND NOT #i_bCloseBtn;
    
END_IF;

    #q_bValveOut := #iq_bValveState;