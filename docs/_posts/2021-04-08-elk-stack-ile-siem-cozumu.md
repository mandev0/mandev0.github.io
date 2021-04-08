---
layout: post
title: ELK Stack ile SIEM Çözümü
excerpt: "ELK Stack, Suricate, Sophos XG ile ücretsiz/açık kaynaklı güvenlik çözümleri"
comments: true
---
Uzun zamandır yapmayı planladığım ama bir türlü başlayamadığım yazı serisine nihayet başlıyorum. Siber güvenlik alanına red team tarafı ile başlamış biri olarak bu alanda çok şey öğrendim. Ancak zaman geçtikçe ve daha çok şey görüp öğrenikçe blue team tarafınada ilgim büyüdü ve odak noktam olan sızma testi yerini savunmaya bıraktı. İlk iş tecrübemin network ve ağ güvenliği olması bir çok bilgi ve tecrübe edinmemi sağladı ancak tecrübelerim genellikle küçük işletmelerle sınırlı kaldı ve bir çok çözümü tecrübe etme şansı elde edemedim. Bu sebeple sektörde elde edemediğim tecrübeler ve araçlar için kendime bir lab ortamı kurmaya başladım.  

Uzun uğraşlar ve bir çok kurup silmeler, optimum çözüm arayışları sonucu aşağıdaki lab ortamını planlayıp kurmaya başladım. Kurulum adımlarına bir sonraki yazımda yer vereceğim. Bu yazıda daha çok tercihlerimi ve yapıyı neden bu şekilde kuruduğumu anlatmak istiyorum böylece varsa hatalarım daha anlaşılır olacaktır :)  

<div style="text-align:center"><img src="/docs/img/SIEM_Lab_Diagram.png" /></div>  

Burada üç network bölümümüz var. Bu alanları Sophos XG Home versiyonu üzerinde oluşturdum. Yapı optimum olmayabilir. İslevsellik olarak DMZ alanı dışarı açık alanı temsim ediyor. ServerZone ile arasındaki fark SZ' nin dışarıya açık olmaması (Bu ortamı kendi evimde kurduğum için aslında hiçbir network dışarı açık değil ama senaryolarımı ona göre şekillendireceğim). Networklerin erişim izinlerini aşağıdaki gibi ayarladım.    

| Zone          | İnternet | DMZ | ServerZone | LAN |
| ------------- | -------- | --- | ---------- | --- |
| DMZ           | +        | *   | -          | -   |
| ServerZone    | +        | +   | *          | -   |
| LAN           | +        | +   | +          | *   |  

Network ile ilgili son olarak Suricate ve Logstash ile ilgili iki durum mevcut. Suricata sunucusunun DMZ alanını dinlemesi gerekli (yada bütün ağı) bunun için ikinci bir bacak oluşturdum ve ev tipi 8 portluk tp-link akıllı switch ile bütün trafiği o network bacağına yönlendirdim. Logstash ile ilgilide benzer bir durum söz konusu. DMZ alanından gelecek logların ELK Stack sunucusuna aktarılması gerekiyor. Bu işlemi direkt olarak ELK sunucusuna gönderecek şekilde yapabiliriz ama ben DMZ alanının direkt olarak ELK sunucusuna erişmesini istemedim. Bu nedenle (ve geleceğe dönük tavsiye edilen yapıda) logları Logstash servisine gönderecek şekilde planladım ve bu servisi ayrı bir sunucuda çalıştırdım. Aslında burada Logstash sunucusu üzerinde tek bir bacak ile işi çözebilirdim ancak daha iyi ayrım oluşturmak açısından bir bacak ServerZone diğer bacak DMZ alanında olacak şekilde iki bacaklı bir sunucu yapısı oluşturdum. Bu anlatıma dair diagram aşağıdaki gibi.  

<div style="text-align:center"><img src="/docs/img/ServerZone_Diagram.png" /></div>  

Şimdi biraz çalışacak servislerden bahsedelim. Öncelikle SIEM(ish) çözümümüz ELK Stack. Açık kaynak kodlu bir araç ve log yönetimi açısından oldukça başarılı ve içerisinde hazır gelen alarm imzaları ile antreman için oldukça yeterli. Bu araç yerince Splunk kurmayıda planladım ve bir kaç denemede yaptım ancak ücretsiz sürümü için kısıtlamalar ve kota(günlük 500MB) olması beni ELK Stack denemeye itti. İleride Splunk ile ELK Stack aracını alarmlar konusunda karşılaştıracağım bir yazı daha planlıyorum onun için Splunk planlarımda var.  

Diğer bir ürünümüz Sophos XG Home sürümü. Uzun zamandır evde firewall olarak pfSense kullanıyordum (merak işte :)). Her ne kadar açık kaynak çözümleri sevsemde uzun süreli bir pfSense kullanıcısı ve destek verdiğim bir ürün olsada kullanım zorluğu ve web filtreleme gibi özelliklerin stabil çalışmaması nedeniyle farklı çözüm arayışlarına başladım. İlk önce Untangle Firewall ürününü denedim ancak ücretsiz sürümü bazı kısıtlamaları ile geliyordu o yüzden arayışlarım devam etti ve sonunda Sophos XG ürününün ücretsiz bir sürümü olduğunu gördüm. Bu sürümde tek kısıtlama donanım kullanımı ile ilgili (gelişmiş özellikler hariç). Cihaz fazladan donanım verseniz dahi performans olarak 4 çekirdek ve 6gb ram kullanımı geçmiyor. Oldukça makul bir kısıtlama, uzun süredir kurumsal cihazlar kullanmak istiyordum ve bu sürüm tam bunun için. Ayrıca bir çok filtreleme ve loglama özelliğide yanında geliyor :)  

Son olarak IDS ürünümüz Suricata. Bu yazılım IPS olarakda konumlandırılabiliyor ancak ben bu lab için daha çok loglama ve alarmlar üzerine yoğunlaşacağım için herhangi IPS konumlandırmak istemedim o yüzden pasif olarak ağı dinleyen bir IDS konumlandırdım. Snort ve Suricata arasında biraz araştırma yaptıktan sonra Suricata aracında bazı güzel özellikler gördüm. TLS handshakelerinden SNI(Server Name Indication, veri şifreli olsa dahi girdiğiniz domain bilgisi burada açık ediliyor. Bunun ile ilgili güzel bir yazı geliyor) bilgisinin çıkarılması, çoklu çekirdek kullanımı ve ELK için hazır gelen loglama aracı bu tercihimde baskın rol oynadı.  

Bütün bu sistemlerin ardından DMZ alanı içerisindeki sunucularımıza ELK Stack aracına ait log toplayıcıları kurup loglarımızı yönlendirmeye başladığımızda kurulumumuz büyük oranda tamamlanmış olacak.  

Peki iyi güzelde bütün bu sistemleri kurduk, logları topladık peki ne olacak. Bundan sonrası için plan APT gruplarına ait kampanyaları/senaryoları araçlar ile canlandırıp bu saldırıları test edip edemediğime bakacağım. Tespit edemediğim saldırı veya saldırıya ait adımlar için özel imzalar yazıp bunları test edeceğim. Bu yazı serisinin oldukça eğlenceli ve dolu dolu olmasını planlıyorum. Burada yaptıklarımın hepsini internette araştırarak oluşturdum ve bu ortam veya senaryolar en iyisi ya da en uygunu olmayabilir ancak deneye yanıla öğreneceğim. Bütün geri bildirimler benim için değerlidir. Şimdiden bir sonraki yazıda görüşmek üzere hoşçakalın :)
