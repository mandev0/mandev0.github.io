---
layout: post
title: ELK Security ile Ev Yapımı Sandbox
excerpt: "Lokalde çalıştırabileceğimiz tamamen sizin kontrolünüzde bir sandbox çözümü"
comments: true
---
Bir süredir aklımda olan ve [Vedat](https://twitter.com/vbora0) ile ara ara konuştuğumuz bir konu vardı oda sandbox sistemleri. Aslında lokalde kullanılabilen bir sandbox sistemi. Neden böyle bir kurulum ihtiyaç duydum derseniz... yani neden olmasın :D

Ancak konuya giriş yapmadan önce belirtmem gerek. Bu tamamen teknik dolu bir yazı olmayacak daha doğrusu adım adım takip edebileceğiniz bir kurulum yazısı olmayacak. Daha çok genel hatları ile biraz bilen birine anlattığım seviyede bir yazı olacak. Belki ileride bu yazıyı parçalara bölüp tam bir anlatım yapabilirim. :) Şimdilik böyle başlayalım.

İlk önce açık kaynak olarak neler var ona baktım. Bu süreçte en popüler isimlerden olan "Cuckoo Sandbox" uygulamasına baktım tekrar ancak bir süredir geliştirilmiyor belki görmüşsünüzdür. E böyle eski bir uygulama ile bir sandbox ortamı kurmak çok sağlıklı gelmedi o nedenle araştırmaya devam ettim. Bu sırada [şöyle](https://www.elastic.co/blog/how-to-build-a-malware-analysis-sandbox-with-elastic-security) bir makaleye denk geldim. Makale uzun zamandır kullandığım ve boşta kaldıkça kurcaladığım ELK stack ile bir sandbox ortamı kurmaktan bahsediyordu. Aslında tam bir ortam anlatımı yoktu ancak ELK nın dinamik analizi yapan ve güzel şablonlar ile süslediği arayüzü kullanarak bir ortam oluşturmaktan bahsediyordu. Ee konu ELK olunca hemen başladım çalışmaya.

> Makale temel olarak ELK nın bu iş için nasıl kullanılabileceğini gösteriyordu. Tam bir sandbox ortamı nasıl olur vs çok detay yoktu

Şimdi zararlı yazılımımızı çalıştırdığımız zaman elimizde logları toplayacak ve güzelce sunacak bir platformumuz vardı ancak bir sandbox ortamımız yoktu. Bu kısımda sadece bir windows ortamı değil aynı zamanda izole bir network ortamına ve sıkı kontrollere ihtiyaç vardı. Öncelikle kolay kısımdan başlayalım.

Network tarafında izole bir ortam oluşturmak en azından elimdeki mevcut ortamda kolayda. Elimdeki sistem ile ilgili bir makale yazıcam ancak kısa bir özet geçmek gerekirse. Eski bir HP workstation makineye Proxmox sanallaştırma yazılımı kurdum. Sonrasında en öne Sophos FW nin Home Edition lisansına sahip bir firewall konumlandırdım. Bu firewallda iki tane fiziksel bacak var. Biri WAN diğer LAN. WAN internet için LAN ise evdeki cihazlar için. 

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/proxmox-interfaces.png" /></div>  

Bu kısma kadar basit olan yapıydı. Geri kalan networkler ise sanal olarak Proxmox üzerinde oluşturuldu. Bunlar arasında "Server", "DMZ" ve en önemlisi "Sandbox" networkü bulunuyor. Mevcut bir ortam olduğu için dediğim gibi kurulum basit oldu. Proxmox üzerinde yeni bir bacak ve sonrasında Sophos FW de yeni bir interface. Şimdi gelelim kurallara.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/sophos-interfaces.png" /></div>  

Kural olarak en önce Sandbox tarafından bütün trafiği engelledim. Sonrasında sadece WAN a çıkış verdim ve son olarak içerideki sandbox a erişim için "Server" networkünden RDP ve SSH izni verdim. Bütün kurulumları tamamlayınca WAN iznini kapadım. Bu şekilde son durumda sadece sandbox makineye RDP ve SSH yapabilen bir kural dizisi mevcut oldu. İkinci olarak logları içeriden almamız gerekliydi. Ancak mevcutta kullandığım SIEM kurulumunu kirletmemek ve tabiki Sandbox ortamının içerideki sistemlere erişimine izin veremeyeceğim için içeri ikinci bir ELK kurulumu gerçekleştirdim. İki ELK ile birlikte HP makine iyice yaşını belli etmeye başladı :D

> Orada docker yazmasının sebebi aslında remote erişim için kullandığım Apache Guacamole sunucusunun docker üzerinde çalışması

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/sophos-rules-1.png" /></div>  
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/sophos-rules-2.png" /></div>  

Bu şekilde temelde network tarafını bitirmiş olduk. Şimdi sıra geldi VM ya da sandbox tarafında. Bu kısım için ilk aklıma gelen Proxmox üzerinde bir Windows makine oluşturup her test öncesi bir snapshot ile yedekli şekilde malware çalıştırmak ve sonrasında ilgili temiz duruma geri dönmekti ancak bu çok zor gelecek olduki başka arayışlara girdim. O sırada karşıma nasılsa daha önce görmediğim Windows Sandbox uygulaması çıktı. Bu microsoft tarafından hazırlanmış ve Windows 10 ya da 11 üzerinde çalışan bir sandbox uygulaması. Bu uygulama (aslında daha derin bir fonksiyonu ve işleyişi var ama basitlik olsun) windows sistemi izole bir şekilde çalıştırıyor ve kapandığın zaman tamamen sıfırlanıyor. Tam bu işler için hazırlanmış.

Şimdi bu uygulama ile sandbox işinide çözdük sayılır ancak bir eksik var. Uygulama her kapandığından nasıl sıfırlanıyorsa her açıldığındada aynı şekilde boş geliyor :D. Bu sorunu geliştirici arkadaşlarda görmüş olacakki onlarda başlangıçta çalıştırılabilecek bazı komutlar ve işlevler verilebilmesi için bir konfig dosyası hazırlama olanağı sunmuşlar. Bu şekilde sandbox her açıldığında önceden hazırladığımız komut ya da scriptler çalıştırılıp sandbox istediğimiz duruma geri getirilebiliyor. Ee o kadar yolu geldik şimdi tek eksiğimiz bir loglama sistemi değil mi :D

Şimdi gelelim ELK tarafına. Burada şöyle bir sorun var, daha doğrusu bu makale için bir sorun. Sadece ELK kurulumunu anlatmam bir kaç makaleye denk gelebilir. Daha önce yazdığım makale serisi mevcut o nedenle o kısmı atlıyorum ve buraya [linki](https://selimakpinar.com/articles/2021-04/elk-stack-ile-siem-cozumu) bırakıyorum. Evet biraz eskimiş bir yazı serisi ancak güzel bir başlangıç yapabilirsiniz.

Kurulumu tamamladığınızda aşağıdaki script için gerekli komutu kendi ortamınızdan alabilirsiniz. Ayrıca ben birde Sophos FW loglarını buraya forward ettim. Bunun için yine Fleet üzerinde gerekli ayarlamaları basitçe arayüzü kullanarak yapabiliyorsunuz ve Sophos parserı içerisinde hazır olarak geliyor. Yıllar içinde kullanımı ve yönetimi çok gelişti ELK nın. Mesela log kaynaklarını yönettiğim ekrandan bir kaç alıntı.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/fleet-1.png" /></div>  
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/fleet-2.png" /></div>  

Şimdi bütün parçaları hazırladık ve birleştirmeye hazırız. Kısa bir özet geçelim tekrar. Bir adet sandbox ortamımız var ve bu ortamı biz network tarafında izole ettik. Burası güzel. Bir malware çalıştırdığımız zaman gerekli logları toplamak ve görselleştirmek içinde ELK kurdum ve ayarladık. Şimdi geriye sandbox makinemizi her açıldığında tekrar ayarlamaya yarayacak konfig dosyamızı yazmaya. Bu kısımda [şu](https://github.com/rcybersec/windowssandbox) arkadaşın hazırladığı konfigden bolca ilham aldım diyebilirim ama bolca :D. Ancak temelde basit tuttum. Arkadaş başka araçlarda yüklüyordu onları iptal ettim vs.

Temelde iki dosyamız mevcut. Biri konfig dosyası diğeri ise scriptimiz. Sırasıyla içerikleri aşağıda mevcut. Yorumları ve açıklamaları yanına yazdım okuması daha kolay olması için  
  
Script dosyamız,
```powershell
### Başlangıçta çalıştırılan script (sandbox-init.cmd)

# Powershell için gerekli gereksiz adımımız
powershell.exe -command "&{Set-ExecutionPolicy RemoteSigned -force}"

# 7-zip yükleme komutumuz
powershell.exe -command "Start-Process -Wait -FilePath "C:\Users\WDAGUtilityAccount\Work\WindowsSandbox\7z2301-x64.exe" -ArgumentList "/S" -PassThru"

# Agent kurulumu için gerekli klasörü kopyalıyoruz
powershell.exe -command "Copy-Item "C:\Users\WDAGUtilityAccount\Work\WindowsSandbox\elastic-agent" -Destination "C:\Users\WDAGUtilityAccount\Desktop\elastic-agent" -recurse -Force"

# Elastic agentı yüklemek için gerekli komutumuz (Bu komut aslında hazır olarak geliyor. ELK ve sonrasında Fleet kurarsanız oradan direkt olarak size veriyor)
powershell.exe -command "C:\Users\WDAGUtilityAccount\Desktop\elastic-agent\elastic-agent.exe install --url=https://10.1.50.11:8220 --enrollment-token=TF93TmQ0NEI4MmFSVnE1S0tnQUg6TmpzTVVaSFFUaWlFc2IyOW1YTU5HUQ== -f --insecure"
```
  
Konfig dosyamız,
```xml
<!-- Konfig dosyamız. Bu dosyaya çift tıklayıp sandbox ortamını başlatıyoruz (sandbox-start.cmd) -->

<Configuration>
	<VGpu>Disable</VGpu>
	<!-- Network erişimini aktif ediyor. Burası kritik!!! -->
	<Networking>Enable</Networking>
	<MappedFolders>
		<MappedFolder>
			<!-- Sandbox aracının kurulu olduğu makinedeki paylaşılan klasör -->
			<HostFolder>C:\Users\mandev\Desktop\Sandbox</HostFolder>
			<SandboxFolder>C:\Users\WDAGUtilityAccount\Work\WindowsSandbox</SandboxFolder>
			<ReadOnly>true</ReadOnly>
		</MappedFolder>
	</MappedFolders>
	<LogonCommand>
		<!-- ve en önemli kısım hazırladığımız script -->
		<Command>C:\Users\WDAGUtilityAccount\Work\WindowsSandbox\sandbox-init.cmd</Command>
	</LogonCommand>
</Configuration>
```

Peki bu kadar komut ve yazıdan sonra elimize ne mi geçti. İşte şu güzel ekran :heart_eyes:

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/siem-1.png" /></div> 

Aynı zamanda bolca alert üretiyor sistem. Tabi eklemeyi unuttum. Bu alertlar boşa oluşmuyor. Sandbox makinede "Locky.exe" i çalıştırdığımda üretiliyor. Bu exeyi githubdan dikkatlice sandboxa atıp orada çalıştırdım.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/siem-2.png" /></div> 

Şimdi geldik sonuca. Bu kadar uğraştık ettik ancak bir şeyler eksik gibi. Büyük harflerle yazıyorum :D CTI. Bir CTI kaynağı eklemediğimiz için alarmlarda ilgili oluşan IOC lere otomatik bir kontrol ekleyemiyoruz. Mesela bir örnekle gidelim. Yukarıdaki alarmlardan birini incelediğimizde çalışan processlerden birine bakalım ve oluşturduğu bir dosyayı kontrol edelim.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/siem-3.png" /></div> 
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/elk-security-sandbox/siem-4.png" /></div> 

Gördüğünüz üzere dosya kaynıyor :D ancak bununla ilgili bir alarm oluşmuyor çünkü oluşan MD5 ya da SHA1 hashlerinin karşılaştırılacağı bir kaynak eklemedik. Maalesef bildiğim kadarıyla ELK ücretsiz olarak içerisinde bir CTI kaynağı ile gelmiyor o nedenle bizim eklememiz gerek ancak bu başka bir makale konusu.

Günün sonunda bolca kan ve göz yaşı döktüğüm bir uğraşı oldu benim için. Bu haliyle özellikle zararlı web sitelerini ziyaret etmek için sık sık kullanabileceğim bir ortamım oldu. Buradaki CTI eksiğini hızlıca kapattığımda oldukça güzel çalışan bir sandbox ortamı kurmuş olacağız. 

Son olarak bu sistemi çalışır hale getirene kadar bulduğum bütün güzel repoları salladığım Vedat'a beni çektiği için bir teşekkür. Birde adama bu sistemi çalıştırdığımda gece 1-2 gibi ekran görüntüsü atmıştım deli gibi :D Neyse lafı uzatmadan bir sonraki yazıda görüşmek üzere :)