STM32_Embedded_Journey

STM32 ile yaptığım gömülü yazılım tekniklerini öğrendiğim maker projelerimi paylaşıyorum.

#Repo03 : STM32F407-BUTTON-INTERRUPT

NE YAPTIM : 

STM32F407-DISC1 ile bare metal kodlama yaparak STM32CubeIDE’de buton (user button)’u toggle eden kesme tabanlı bir kontrol akışı gerçekleştirdim.

KAVRAMSAL HARİTA : 

Interrupt vs Polling : “Polling”’de , CPU sürekli bir register’ı , aldığı çıktıya bağlı olmaksızın, bir döngü halinde okuyup kontrol eder. “Interrupt” ‘ta ise donanım ancak bir olay olduğunda CPU’nun halihazırda yaptığı işi kesip saklar (interrupt), ardından ISR’ye geçer ve ilgili olayı gerçekleştirdikten sonra CPU kaldığı yerden devam eder. 

EXTI Nedir ? : EXTI, çip içi elektronik bir devredir. Görevi pinlerdeki değişimi interrupt’a çevirmektir. 0V’dan 3.3V’a geçişi veya 3.3V’dan 0V’a geçişi yakalar. Register değil bir tür peripheral’dir. 

NVIC Nedir ? : Çip içi elektronik bir devredir. EXTI’de olan değişimlerle hemen CPU’nun işlevine müdahale edilemez, CPU o sırada farklı önceliklere sahip başka olayları işliyor olabilir. Bu yüzden interrupt’lar arasında önceliklerin (priority) yönetilmesi gerekir. NVIC, interrupt’lardaki bu priority yönetimini yapar.

Vector Table Nedir ? : Bir tür veri yapısı. Interrupt geldiğinde CPU vector table’a bakarak hangi handler fonksiyonunu işleme alacağına bakar ve bu bilgiye göre o handler’ı işleme alır. 

STM32’de EXTI Nasıl Çalışır ? : EXTI’nın kendisi bir register değil fakat bğlı olduğu register’lar ile interrupt oluşturma sürecini yürütür.

SYSCFG_EXTICR : EXTI hatlarının hangi GPIO portuna bağlanacağını belirleyen multiplexer register. Aynı numaralı pinler aynı numaralı EXTI hattına bağlanır. Örneğin; PA0, EXTI0 hattına bağlanır veya PD15, EXTI15 hattına bağlanır. PB0’ın da bağlanacağı hat EXTI0’dır fakat numarası aynı pinlerin arasından yalnızca tek bir pin EXTI hattına bağlanmalıdır. Bu bağlantıyı SYSCFG_EXTICR registerı yapar.

EXTI_IMR :  İşlemciye “bu hatta bir değişim olursa bana haber  ver” demenin bir yoludur. Eğer bu register 0 yapılırsa herhangi bir değişim olsa da işlemciye haber verilmez. Ancak bu register 1 yapıldığı durumda CPU, gerçekleşen değişimi duyar.

EXTI_RTSR / EXTI_FTSR : EXTI_RTSR register’ı değişimin 0V’dan 3.3V’a olduğu anda tetiklenirken EXTI_FTSR değişimin 3.3V’dan 0V’a doğru gerçekleştiği anda tetiklenir. Kodlama yapılırken ilgili register’ın pull_down veya pull_up oluşuna göre sırasıyla EXTI_RTSR veya EXTI_FTSR set edilir.

EXTI_PR : CPU tarafından işlenmeyi bekleyen interrupt olup olmadığını kontrol eder. İnterrupt doğrultusunda ilgili handler çalıştırıldıktan sonra EXTI_PR register’ının kontrol edilmesi gerekir. Bu register’daki biti maskelemek için o bite 1 yazmak gerekir. Çünkü yazılım dünyasında 0 her zaman “etkisiz eleman/işlem yapma”, 1 ise “eylemi tetikle” anlamına geldiği için; donanıma da 1 yazıldığında reset bacağını tetikleyecek şekilde tasarlamışlardır.

Bounce ve Debounce Nedir ? : Bir butona basılıp iki metal kontağin birleştirildiği sırada aslında iki metal kontak çok kısa bir süre içerisinde olsa da birbirine çarpıp ayrılır, sonra sabitlenir. Bu duruma bounce denir. Debounce ise bu durumu tolere etmeyi, bounce süresi geçene kadar ek geçişleri görmezden gelmeyi sağlar. Bunu gerçekleştirmek için iki yaklaşım var:

1)	Hardware debounce : devreye bir kapasitör/RC filtre ekleyerek sinyali yumuşatmak

2)	Software debounce : kod seviyesinde, ilk tetiklemeden sonra belirli bir süre boyunca gelen ek interrupt’ları yok saymak.

KNOW HOW : 

Kodun sonundaki while (1) döngüsü olmasaydı, işletim sistemi olmayan bir ortamda, (şu anki kodladığımız STM32’de) yani çipe elektrik gelir gelmez çalışmaya başlayan Reset_Handler’ın  main’i çağırdıktan sonraki süreçte, eğer main biterse (sonunda return 0 olsa) CPU rastgele bir yere zıplayabilir bu da çip donması, çökmesi, alakasız komutlar çalıştırması gibi tanımsız davranışlara sebep olabilirdi. Şu anki haliye boş bir döngünün oluşu main’i çerçevesi çizilmiş bir sonsuz döngüye sokarak işlevin sürmesini sağlıyor. Bu boş döngünün içinde CPU boş yere enerji harcıyor, herhangi bir iş yapmıyor ama interrupt geldiğinde de EXTI_IRQHandler fonksiyonuna gidiyor. CPU ‘nun interrupt haricinde, döngü içinde harcadığı enerjiyi azaltmak ve sadece döngü içinde uyku haline sokmak için aşağıdaki gibi bir kullanım tercih edilebilir:

While (1){
	__WFI();
}


Main fonksiyonunun içerisine EXTI0_IRQHandler fonksiyonunu çağıran bir kod yazılmadığı halde, çip üzerinde butona basıldığında EXTI0_IRQHandler fonksiyonu devreye giriyor. Bunun sebebi, halihazırda bu fonksiyonun Vector Table’da kayıtlı olmasıdır. Yani çip, öncesinden bunun için programlanmıştır.

EXTI_IMR varken NVIC_ISER0’a gerek var mı ? Aslında ikisi farklı şeyler. EXTI_IMR, bu pindeki gerçekleşen değişim sinyal üretebilsin mi iznini veriyor. NVIC_ISER0 ise CPU seviyesinde, bu sinyal CPU’ya ulaşsın mı ulaşmasın mı izni veriyor. Bu yüzden ikisine de ihtiyaç var.

NOT: Bu kodda SYSCFG_EXTICR1 register’ını açmasaydık da kodumuz çalışırdı. Çünkü zaten PA0 için register’ı 0000 yapmamız gerekiyordu. Fakat bu çalışma tesadüfen gerçekleşirdi çünkü A pini yerine başka bir pine bağlamak isteydik SYSCFG’yi 0000’dan farklı bir değer kullanmalıydık. Bu durumda da kod compiler edilirdi fakat işlevini yerine getirmez, pini yakmazdı. 


