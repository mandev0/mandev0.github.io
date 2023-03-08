---
layout: post
title: Überonline Modem Zorbalığı
excerpt: "ISP firmalarının gereksiz sınırlamaları ve engelleri"
comments: true
---
Yeni bir adrese taşındık ve bu adreste eski yere göre fiber altyapı mevcuttu. Bu haberi büyük bir sevinçle karşıladım ancak bir problem vardı. Daha önceleri de sorun yaşadığım überonline firmasının altyapısıydı ve pek fazla diğer isp firmaları ile alyapı paylaşımı yapmıyordu. 12 mbps adsl altyapısı bağlatamayacağım için mecbur aradım ve hattı açtırdım.  

```base
Burada firma ismi olarak überonline kullanıyorum malum sebeplerden :)
```

Kurulum için iki adet cihaz veriyorlar biri GPON diğeri ZTE model bir router. Router yeni wifi 6 destekli vs güzel bir cihaza benziyor ama daha önceleri kendi network altyapıma güzel yatırım yaptığım için o router cihazını sökmeyi kafaya koymuştum. Kuruluma geldikleri gün arkadaşlara da sordum cihazı değiştirebilir miyim diye ancak değiştiremezsiniz dediler ben hiç üstelemeden kapattım konuyu.  

Taşınma halleri malum çok vaktim olmadı ilk haftalar ama sonrasında router cihazını değiştirmek için seçtiğim bir cumartesi günü ilk yaptığım iş olarak aboneliğimin kullanıcı adı ve şifresini almaya çalıştım ancak hiç bir şekilde bu bilgiye ulaşamadım. Herhangi bir yolla bu bilgiyi bana vermiyorlar. Benim aldığım, parasını verdiğim hizmet için kullanımımı kısıtlıyorlardı. Daha fazla uğraşmamak adına ZTE router cihazını kullanmayı denedim ancak ince ayar yapılamıyor ve daha önemlisi DNS değişikliği yapmaya izin vermiyordu.  

Bu noktada artık bu kısıtlamalar biraz fazla gelmeye başladığı için ilk önce iptal etmeyi düşündüm ama başka alternatifim şuan için yoktu. Telekom fiber altyapısı için şimdi başvursam gelip gelmeyeceği meçhul. Bu sebeple bir şekilde kullanıcı adı ve şifremi öğrenmem gerektiğine karar verdim ve araştırmaya koyuldum.  

Zaten genel olarak bu altyapılar ile tecrübem olduğu için ilk bağlantıda ZTE router cihazının PPPoE kimlik biglilerini karşı tarafa gönderdiğini biliyordum ancak bunun şifreli olup olmadığına bakmam gerekti ve aslında şifreli gitmediğini öğrendim. Bu büyük bir engeli ortadan kaldırmıştı. İkinci büyük engel aradaki trafiği nasıl dinleyeceğimdi. Bu kısım için şansıma buradaki altyapı kendi kullandıkları tek cihaz modelini desteklemediği için iki ayrı cihaz ile kurulum yapmışlardı ve bu iki cihaz birbirine ethernet ile bağlıydı. Artık tek eksik aradaki trafiği dinlemekti.  

Daha önceleri VLAN kullanımını öğrenmek ve araştırmak için aldığım iki tane tp-link marka yönetilebilir switchlerim vardı. Bu cihazlar aynı zamanda belirli portlardaki trafiği belirlediğiniz bir porta mirror (aynalama) edebiliyorlardı. Bu şekilde önce GPON cihazı ve ZTE modem arasındaki kabloyu araya yönetilebilir switch gelecek şekilde konumlandırdım. GPON dan gelen kablo switch-1 e ZTE routerdan gelen kablo switch-2 ye şeklinde bağlantıyı sağladım. Switch-5 portuna da kendi masaüstü bilgisayarımı bağladım ve bütün trafiği switch-5 e mirror ettim. Diyagram aşağıdaki gibi oldu böylece.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/uberonline_router/ethernet-diagram.png" /></div>  

Sonrasında masaüstünde wireshark ile trafiği dinlemeye başladım ve ZTE router cihazını yeniden başlattım. Yaklaşık iki dakika içerisinde PPPoE şifre bilgileri wireshark ekranında görünür oldular :)  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/uberonline_router/uberonline-pppoe.png" /></div>  

Bu bilgileri kendi router cihazımda kullandım ve ek hiçbir ayar gerekmeden überonline firmasının verdiği ZTE router cihazını aradan çıkarmış oldum. Artık bütün network altyapımı eskisi gibi fazla cihaz olmadan kurabilir hale geldim.  