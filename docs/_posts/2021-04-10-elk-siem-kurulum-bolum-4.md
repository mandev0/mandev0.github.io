---
layout: post
title: ELK Stack ile SIEM Lab Ortamı | Bölüm 4
excerpt: "ELK ile Alarm Üretme"
comments: true
---

Öncelikle Kibana üzerinden "Securiyt" -> "Detections" altında "Load Elastic prebuilt rules and timeline templates" butonu ile kurallarımızı indirelim. Başlangıç olarak linux hedef makinemiz ile başlayalım. Bunun için "Tags" altında "Linux" tagını seçip ona özel kuralları listeleyelim. Bu listeden ilk bakmak istediğim kural "Auditd Max Failed Login Attempts". Bu kural auditbeat tarafından gönderilen loglar arasında belli bir arama yapıyor. Detaylarına aşağıdaki resmi inceleyerek bakalım.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/siem_lab_kurulumu/max_login_rule.png" /></div>

Sol üstteki kırmızı kutudan başlar isek sırasıyla,  
- Alarma ait önem seviyesi ve ona ait skor bilgisini görüyoruz.
- MITRE ATT&CK Framework yapısında alarma ait adımları gözüküyor
- Hangi aralıklar ile ve geriye dönük zaman aralığı
- Yapılan arama
- Hangi index içerisinde yapıldığı  

Burada özel olarak "Custom query" alanından bahsetmek istiyorum. Bu kısım loglarımız içerisinde yapılan aramayı gösteriyor. Eğer bu arama bir sonuç döndürürse bu alarm oluşturuyor. Bu kural için iki durum mevcut.  
Öncelikle "event.module" içerisinde "auditd" aranıyor **ve** "event.action" içerisinde "failed-log-in-too-many-times-to" string değeri aranıyor. Burada önceki cümlede kalın harfler ile belirttiğim "ve" ifadesi önemli. Bu kuralın alarm üretmesi için iki durumunda **doğru** dönmesi lazım.  

Şimdi bu kuralı aktif edelim ve SSH portuna bir bruteforce saldırısı ile alarm üretmeye çalışalım. Ben bu iş için kendi windows makinemde yüklü Kali linux WSL makinesini kullanacağım. Kali içerisindeki Hydra aracı ile aşağıdaki komutu kullanarak bir saldırı başlatalım.
```bash
hydra -l root -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt victimdefine.home -t 4 ssh
```
Bir süre uğraşmama rağmen bir türlü alarm üretemedim. Elle arama yaptığımda "failed-log-in-too-many-times-to" değerini hiç göremedim. Araştırmama devam ettiğimde ise bu alarmın üretilmesi için linux sistemde hesap blocklamanın aktif edilmesi gerektiğini gördüm. Güzel bir uygulama olduğu için bunu [buradaki](https://www.linuxtechi.com/lock-user-account-incorrect-login-attempts-linux/) yönergeleri takip ederek aktif ettim. Sonrasında tekrar komutu çalıştırdığımda yine alarm üretemedim. O sırada aklıma root hesabı için SSH girişinin zaten kapalı olduğu aklıma geldi. Bu durumda aslında olmayan hesaplar için engelleme yapılamayacağından kaynaklı bazı bruteforce saldırılarının bu kural ile yakalanamayacağı ortaya çıktı. Diğer kurallarada bakmama rağmen bu iş için hazır bir kural bulamadım bu nedenle iş başa düştü ve Kibana içerisinde "Discovery" altında log kayıtlarını incelemeye başladım ve belli bir aralıkta ve belli bir sayıda giriş denemesi yapıldığı zaman buna bağlı olarak alarm üretebileceğimi gördüm. Aşağıda kullanacağım alanlara ait bir resim mevcut şimdi bu alanları kullanarak kendimiz bir kural yazalım. Öncelikle "Discovery" sayfasına bakalım.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/siem_lab_kurulumu/discovery_1.png" /></div>

Burada iki kısmı baz aldım, "related.user" ve "event.type". Belli bir kullanıcı için belli bir zaman diliminde kaç defa "authentication_failure" string değerinin geldiğine bakıp çıkan sayıya göre alarm üreteceğiz. Bunun için tekrar "Detections" sekmesi altına geri dönüyoruz ve "Create new rule" ile yeni bir kural oluşturuyoruz. Bu kural için "Rule Type" olarak "Threshold" seçiyoruz ve aşağıdaki gibi ayarlıyoruz.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/siem_lab_kurulumu/new_rule_1.png" /></div>  

Sonrasında kural için bir isim, açıklama ve kritiklik seviyesi belirliyoruz. Son olarak her beş dakikada bir +1 dakika geriye bakacak şekilde ayarlıyoruz. Bu şekilde ilk kuralımızı yazmış ve aktif etmiş olduk. Şimdi kuralın çalışması için bir süre bekleyelim. Geçen sürede herhangi bir alarm üretilmedi çünkü son altı dakika içerisinde herhangi bir bruteforce saldırısı yapmadık. Şimdi yukarıdaki komutu tekrar çalıştıralım ve bakalım bu sefer ne olacak.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/siem_lab_kurulumu/alerts_1.png" /></div>  

Bu kısımda iki kural içinde alarm üretebiliyoruz artık. İkinci kuralımız sayesinde artık olmayan kullanıcılar içinde alarm üretebiliyor hale gelmiş olduk. Bu kısımda yazının son kısmıydı. Bundan sonrası için MITRE ATT&CK ve APT senaryoları üzerinden giderek ne gibi alarmlar üretebileceğimize bakacağız. Bir sonraki yazıda görüşmek üzere :)
