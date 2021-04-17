---
layout: post
title: Yeni Bir Teknoloji Yine Bir Yasak
excerpt: "Wireguard websitesinin engellenmesinin ardından yaptığım ufak bir araştırma"
comments: true
---
Her zamanki gibi güne twitter akışıma bakarak başladım. Yakın zamanda kripto paralarla ilgili gelen yasağında etkisiyle akışım bununla ilgili yorumlar ile doluydu. Ardından wireguard ile ilgili bir tweet dikkatimi çekti. İçerik tam olarak aşağıdaki gibi.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/tweet_1.png" /></div>  

Bu haberi alınca direkt olarak nasıl bir engelleme olduğunu görmek için ufak bir test işine girdim. Sonraki kısıma geçmeden önce bu haberi okuduğunda aklıma web sitelerinin engellendiği gelmedi direkt olarak handshake adımında bağlantının engellendiği ihtimali üzerine durdum. Daha önceleri bazı testler yapmıştım wireguard üzerine ancak elimin altında hazır bir sistem yoktu o nedenle bulut üzerinde debian ile ufak bir sunucu kurulumu gerçekleştirdim. Takip ettiğim doküman [linki](https://linuxize.com/post/how-to-set-up-wireguard-vpn-on-debian-10/). Bağlantıyı önce Netspeed internet hizmeti ile denedim ve sorunsuz bağlanıp kullanabildim. Benim bildiğim kadarı ile Turk Telekom altyapısını kullanıyorlar. Bu kısımda bir sorun görmeyinde Turk Telekom internet hizmetini kullanmak için diğer hattıma geçtim ve tekrar sunucuya bağlanmaya çalıştım. Sunucu bağlantısı tekrar sağlandı ancak bu sefer internete çıkamadım. Bağlantım hiç yok gibi değildi, ping attığım zaman dört paketten bazen bir bazen hiç geri dönüş olmuyordu. Bu durum aklıma tam bir engelleme değil ama bir yavaşlatma olabileceğini aklıma getirdi. Bu durumu doğrulamak için bildiğim güvenilir bir teknik yok ancak gözleme dayalı bir kaç şey yazmaya çalıştım. Öncelikle aşağıda iki hat içinde wireguard sunucusuna bağlanırken ve bir adrese giderken aldığım wireshark capture resimleri mevcut(Resimdeki ip adresindeki sunucu artık yok :)).  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/pcap_1.png" /></div>  

Burada dikkatimi çeken paketlerin süresi. Aynı paket sayısı için geçen süre 2 katına çıkmış durumda. Aynı zamanda sunucu tarafında kabaca süreli bir capture testi yaptım Netspeed ile aynı süre içerisinde 5000' den fazla paket yakalanırken Turk Telekom için 200 civarında kaldı. Tabi yaptığım bu testler sağlıklı testler sayılmaz ama genel kullanım(ya da kullanamamak) açısından Turk Telekom için bazı sıkıntılar olduğu açık. TT(Turk Telekom) tarafında genel durumu gördüktan sonra bunu nasıl bypass edebileceğim konusunda araştırma yapmaya başladım. Farklı port denemeleri sonuç vermedi zaten dokümanları okudumunda da wireguard için belirli bir port olmadığını gördüm. Bu öncelikle wireguardın nasıl sınıflandırıldığını bulmak için araştırma yapmaya başladım. İlk adres olarak wireshark kaynak kodlarına bakmak için araştırma yaparken bu [linkde](https://wiki.wireshark.org/WireGuard) wireguard trafiğinin nasıl filtreleneceği gösterilmişti.  

```base
udp[8:4] >= 0x1000000 and udp[8:4] <= 0x4000000
```
Burada UDP payloadı içerisinde belli bir hex araması yapılıyor. Kendi [sayfasında](https://www.wireguard.com/protocol/) farklı mesaj tiplerinde ve yapısından bahsediliyor. Burada dikkat çeken ilk dört byte.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/wireguard_struc_1.png" /></div>  

Aşağıdaki diğer resimler örnek pcap paketlerini görebilirsiniz. Dikkat çeken nokta yine ilk byte'ın mesaj türünü vermesi ve geri kalan üç byte'ın ayrılmış olması. Bu yapı wireguardı sınıflandırmaya yeterli. Buradan yola çıkarak istenilirse ilk handshake mesajında dahi engelleme yapılabilir.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/wireguard_struc_2.png" /></div>
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/wireguard_struc_3.png" /></div>
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/wireguard_block/wireguard_struc_4.png" /></div>  

Elimin altında birde Vodafone Redbox cihazı vardı ve bununlada sunucuya bağlanmaya çalıştım ama bu sefer hiç bağlantı kuramadım. Bu durumla ilgili kesin herhangi bir neden bulamadım açıkçası. Sunucu tarafında tcpdump ile capture yaptığımda bu süre zarfında sunucuya paketlerin ulaştığını ve sunucunun cevap döndüğünü gördüm ancak bu paketler bana ulaşmadı.  

Bu araştırmaları yaparken aynı zamanda wireguard websitesine erişiminde engellendiğini sonunda farkettim :). Burada çok özel bir durum yok aslında klasik DPI cihazı ile SNI bilgisine bakılarak bir engelleme yapılmış. Muhtemelen sizin erişmeye çalıştığınız sunucudan cevap gelmeden önce DPI cihazı size reset paketi göndererek iletişimi sonlandırıyor. Bu tünelleme sisteminin neden hedef alındığını tam olarak anlamış değilim açıkçası. Yeni bir teknoloji ve ülkemizin buna tepkisi artık hiç süpriz dedirtmiyor.
