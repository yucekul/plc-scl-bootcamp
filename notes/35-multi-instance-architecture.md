# Single Instance vs. Multi Instance

**Single Instance**: Bir FB'yi (Örn: FB_Motor) çağırdığınızda ona özel, bağımsız bir Data Block (DB_Motor_1) oluşturulur.

**Multi-Instance**: Bir FB'yi, BAŞKA BİR FB'nin içinde (Static değişkenler kısmında) çağırırsak, TIA Portal yeni bir DB oluşturmaz. Alt FB'lerin tüm hafıza verilerini, Ana FB'nin (Parent) tek bir DB'si içine gömer.


# Neden Multi-Instance Kullanılır?

**Veri Kirliliğini Önler**: 50 motor için 50 DB yaratmak yerine, 1 tane `DB_Tesis` yaratırsınız, tüm motorlar o DB'nin içine gömülür (Encapsulation).

**Kodu Taşıması Kolaydır**: Yazdığınız "Pompa İstasyonu" bloğunu kopyalayıp başka projeye attığınızda, içindeki tüm alt motor blokları ve hafızalarıyla birlikte tek paket halinde gelir.


# EX: MULTI INSTANCE

# // FB_SimpleMotor

#q_bRun := (#i_bStart OR #q_bRun) AND NOT #i_bStop;


# // FB_PumpStation

// 1. BİRİNCİ POMPAYI ÇAĞIRMA (Ana butondan start alır)
#stat_Pump1(i_bStart := #i_bMainStart,
            i_bStop  := #i_bMainStop,
            q_bRun   => #q_bPump1_Run);

// 2. İKİNCİ POMPAYI ÇAĞIRMA (Cascade: 1. Pompa çalıştıysa buna start ver)
#stat_Pump2(i_bStart := #q_bPump1_Run,
            i_bStop  := #i_bMainStop,
            q_bRun   => #q_bPump2_Run);

// 3. ÜÇÜNCÜ POMPAYI ÇAĞIRMA (Cascade: 2. Pompa çalıştıysa buna start ver)
#stat_Pump3(i_bStart := #q_bPump2_Run,
            i_bStop  := #i_bMainStop,
            q_bRun   => #q_bPump3_Run);


# Good-to-know:

1- Bir Matruşka bebeğin içine sonsuz bebek koyamayız. FB_Fabrika içine FB_Hat, onun içine FB_Makine, onun içine FB_Motor, onun içine FB_Valf gömebiliriz ancak Siemens PLC'lerde bu iç içe geçme (Nesting Depth) limiti donanım serisine göre genellikle 8 ile 16 arasındadır. Bu limiti aşarsan derleyici hata verir.

2- Eğer kodu yeni nesil S7-1200/1500 (Optimized Block Access) için değil de eski nesil S7-300'ler için yazıyorsak, bir Global Data Block'un maksimum boyutu 64 KB ile sınırlıdır. Ana bloğun (Master FB) içine devasa string dizileri barındıran 50 tane alt FB gömersek, o Master DB'nin boyutu 64 KB'ı aşar ve PLC'ye yüklenemez. Modern 1500 serisinde bu limit megabaytlar (MB) seviyesindedir, bu yüzden bu modern mimari 1200/1500 sistemlerinde özgürce kullanılır.