# Array of UDTs Nedir?

Bir UDT (Örn: `UDT_WashRecipe`) bir ürünün tüm reçete parametrelerini (Ad, Sıcaklık, Süre, Devir) tek bir pakette toplar. Bu UDT'den bir dizi (`Array[1..5] of "UDT_WashRecipe"`) oluşturduğunuzda, PLC içinde mini bir **Excel Tablosu (Veritabanı)** kurmuş olursunuz.
SCL'de aynı tipteki iki UDT birbirine **tek satırda** kopyalanır:
`AktifRecete := ReceteTablosu[SecilenNo];`


# Neden `InOut` Kullanmalıyız? (Referansla Aktarım)

Devasa bir `Array of UDTs` yapısını bloğun `Input` bacağına bağlarsak, PLC her taramada (saniyede yüzlerce kez) tüm tabloyu kopyalamak zorunda kalır ve yavaşlar.
`InOut` kullanıldığında veri kopyalanmaz, sadece hafıza adresi (referans) gösterilir. Hem CPU yorulmaz hem de blok içinden tabloya doğrudan yeni reçete **kaydedebilirsiniz (Save)**.


# Hayati Güvenlik: Dizi Sınır Kontrolü (Bounds Checking)

Dizimiz `Array[1..5]` boyutundaysa ve operatör HMI'dan yanlışlıkla `0`, `-1` veya `9` girerse, PLC olmayan bir bellek adresine erişmeye çalışır ve **Area Length Error** (Programlama Hatası) fırlatır.
**Altın Kural**: Bir diziyi dinamik bir indeksle (`Dizi[i]`) okumadan veya yazmadan önce mutlaka `(i >= 1) AND (i <= 5)` şartı doğrulanmalıdır.


# EX: REÇETE SEÇİCİ

// 1- Küçük bir reçete kalıbı (UDT_Recipe) tanımlamak.
// 2- Bundan 3 elemanlı bir dizi (Array[1..3] of "UDT_Recipe") yapmak.
// 3- Operatörün girdiği numara 1 ile 3 arasında mı diye bakıp (Sınır Güvenliği), tüm parametreleri tek satırda çıkışa aktarmak.

// 1. KALIP (UDT): Bir reçetede hangi bilgiler var?
TYPE "UDT_Recipe"
VERSION : 0.1
   STRUCT
      rTemp : Real; // Hedef sıcaklık (°C)
      tTime : Time; // Yıkama süresi
      iRPM  : Int;  // Sıkma devri
   END_STRUCT;
END_TYPE


// 2. REÇETE SEÇİCİ BLOĞU (Tek dosya, ekstra DB yok)
FUNCTION_BLOCK "FB_RecipeSelector"
VERSION : 0.1
   VAR_INPUT
      i_iRecipeNo : Int; // Operatörün seçtiği reçete no (1, 2 veya 3)
   END_VAR

   VAR_OUTPUT
      q_stActive  : "UDT_Recipe"; // Seçilen reçetenin 3 bilgisi tek pakette çıkar
      q_bError    : Bool;         // 1..3 dışında numara girilirse yanar
   END_VAR

   VAR
      // 3 reçeteyi tutan tablomuz (Array of UDTs)
      stat_aRecipes : Array[1..3] of "UDT_Recipe";
   END_VAR

BEGIN
    // A. TABLOYU DOLDUR (3 Örnek Reçete)
    // 1. Reçete: Hızlı
    #stat_aRecipes[1].rTemp := 30.0;
    #stat_aRecipes[1].tTime := T#5s;
    #stat_aRecipes[1].iRPM  := 800;

    // 2. Reçete: Normal
    #stat_aRecipes[2].rTemp := 60.0;
    #stat_aRecipes[2].tTime := T#15s;
    #stat_aRecipes[2].iRPM  := 1200;

    // 3. Reçete: Ağır Kirli
    #stat_aRecipes[3].rTemp := 90.0;
    #stat_aRecipes[3].tTime := T#20s;
    #stat_aRecipes[3].iRPM  := 1400;

    // B. SINIR GÜVENLİĞİ VE TEK SATIRDA KOPYALAMA
    // Numara 1..3 aralığındaysa (güvenliyse) reçeteyi tek satırda çıkışa ver.
    IF (#i_iRecipeNo >= 1) AND (#i_iRecipeNo <= 3) THEN
        #q_stActive := #stat_aRecipes[#i_iRecipeNo];
        #q_bError   := FALSE;
    ELSE
        // 0 veya 9 gibi geçersiz sayı girilirse diziye dokunma (PLC çökmesin), hata ver.
        #q_bError   := TRUE;
    END_IF;

END_FUNCTION_BLOCK


# Good-to-know:

1- Dizi sınır kontrolü ile dizi elemanını okumayı asla riskli bir sırada aynı IF içine sıkıştırmamalıyız. Özellikle dizi elemanını okuyan ifade sınır kontrolünden önce yazılırsa (örneğin önce diziyi okuyup sonra indeksi kontrol etmek), PLC daha koşulu bitiremeden sınır dışı belleğe dokunur ve anında hata üretir. İndeks kontrolünü (`LOWER_BOUND` ile `UPPER_BOUND` arasında mı?) daima dış bir `IF` kalkanı veya erken `RETURN` olarak ayır; diziye erişen satır yalnızca bu kalkan geçildikten sonra çalışsın.

2- TIA Portal'da bir değişkeni sadece `String` olarak tanımlarsan bellekte otomatik olarak 256 Byte yer kaplar. Reçete adı ve açıklaması için 3 tane standart `String` koyup 50 elemanlı bir dizi yaparsak sadece metinler yaklaşık 38 KB yer tutar — oysa birçok S7-1200 CPU'sunun toplam kalıcı (Retain) hafızası 10–14 KB civarındadır ve proje derlenmez. Reçete UDT'lerinde her zaman ihtiyacımız kadar boyut belirt (`String[20]` veya `String[32]`). Büyük reçete arşivlerinde (500+ ürün) tüm diziyi Retain yapmak yerine verileri PLC'nin Load Memory (SD Kart) alanındaki Data Block'ta tutup `READ_DBL` / `WRIT_DBL` komutlarıyla RAM'e çekmek endüstri standardıdır.

3- Müşteri makine teslim edildikten 6 ay sonra mutlaka "Her reçeteye bir de soğutma fanı gecikme süresi ekleyelim" der. O gün `UDT_Recipe` boyutunu değiştirmek zorunda kalmamak için, tasarım aşamasında UDT'nin en altına `rSpare_1`, `rSpare_2`, `iSpare_1`, `bSpare_1` gibi yedek değişkenler koymak kıdemli otomasyoncuların altın kuralıdır. İleride sadece bu yedek alanın adını değiştirdiğimizde DB bellek düzeni bozulmaz ve reçete verileri sıfırlanmadan güncelleme yapabiliriz.

4- Operatör makine 20. adımda (pres aşağıdayken veya servo hareket halindeyken) yeni bir reçeteyi doğrudan `ActiveRecipe` üzerine yüklerse, sistem bir sonraki adımda yeni ürünün hedef pozisyonuna veya basıncına aniden sıçrayarak mekanik çarpışma yaratır. Operatör makine çalışırken `EditRecipe` üzerinde gelecek ürünün ayarlarını rahatça düzenleyip `Storage` deposuna kaydedebilir; ancak `Storage` alanından `ActiveRecipe` alanına aktarım yalnızca makine `IDLE` (veya çevrim sonu güvenli adım) durumundayken serbest bırakılmalıdır.

5- WinCC HMI panellerinin kendi dahili "Recipe" eklentisi vardır ancak sahada sıkça yaşanan bir senaryoda HMI ekranı arızalanır, değiştirilir veya senkronizasyon kopar. Kritik üretim parametrelerinin ana kaynağını (Single Source of Truth) daima PLC tarafındaki `Array of UDTs` yapısında tutup HMI'yı sadece bu bloğu görüntüleyen ve tetikleyen bir arayüz olarak kullanmak, ekran değiştiğinde fabrikanın reçetesiz kalmasını önler.