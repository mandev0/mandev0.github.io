---
layout: post
title: Home Lab Part 1 (Donanım)
excerpt: "Home Lab serimizin ilk bölümüne kullandığım donanımlarla başlıyoruz"
comments: true
---
## Yolculuk
İsterseniz eski bir laptopla başlayın, isterseniz ikinci el bir sunucuyla. Zamanınızı verip üzerine uğraştığınız Home Lab ortamınız hem size çok şey öğretiyor hem de çokça eğleniyorsunuz. Bir dizi yazı serisi şeklinde yapmayı planladığım bu ilk yazıda evimdeki mevcut sistemin donanım tarafını anlatarak başlayacağım. Yazıda genel olarak mevcut sistemden ve yaptığım tercihlerin sebeplerinden bahsedeceğim. İlk önce ana sistem olan Threadripper PC den başlayalım.

Yaklaşık bir yıl önce ikinci el olarak satın almıştım bu sistemi. O zamanlar elimde HP'ye ait SFF formatında bir masaüstü sistem vardı. Bu sistemin özellikleri,

 - Altıncı nesil intel i5 işlemci
 - 32Gb ram
 - 500Gb SSD
  
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/home-lab/hp-elitedesk-800-g5-sff-i5-9500-8g256--4573-b.jpg" /></div>  

## Yeni Sistem
O sistemde Home Assistant'tan ELK ya kadar bir çok yazılım çalıştırdım. Farklı senaryolar test ettim. Ancak artık hem yaşını belli ediyor hem de platformun desteklediği maksimum 32 GB RAM yetersiz geliyordu. Bunlarla beraber bir güncelleme arayışına girdim ve platformlara bakmaya başladım. O zamanlar güncel intel ve amd platformlar 128 GB ramle sınırlıydı ve işlemci çekirdek sayıları ve gücü yeterli olsada ben biraz daha ram istiyordum. Bu arayışım beni birinci nesil Threadripper sistemlere yöneltti. Artık üretimi durduğu için ileride kesin çöp olacak bir sistemdi ama o zaman için nispeten ucuza geliyordu. Karar verdikten sonra zaten kısa zamanda sistemi topladım.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/home-lab/threadripper-pic.jpg" /></div>  

Şu an sistem aşağıdaki gibi,

 - 16/32 işlem çekirdekli 1950X işlemci
 - 160 GB RAM
 - 2 Nic Ethernet Kart
 - Nvidia T400
 - 2x256Gb SSD (Raid OS diskleri)
 - 2TB NVME SSD
 - 2x2TB HDD (ZFS Raid)
 - 500Gb
 
NVME diski OS VM'ler için, ZFS RAID'i yedekleme ve 500GB lık diski kamera kayıtları için kullanıyorum. T400 kartsa kameranın nesne tanıma işlerini çalıştırıyor. Ramde son hedef 256Gb :)

Dediğim gibi, bu sistem benim bütün işlerimi yürütüyor. Daha  önceleri kullandığım Sophos FW de burada çalışıyordu ancak bu makineyi kapadığımda komple internette gittiği için onu iptal ettim. Sonra ev ahalisi üstüme çullanıyordu :D

## Proxmox
Bu yazıda çok detaya girmeyeceğim ancak sanallaştırma olarak Proxmox kullanıyorum. Uzun zamandır kullandığım ve çok memnun olduğum bir yazılım kendisi. ZFS için bir ara TrueNas da kurdum içerisine ancak sonradan çok gereksiz geldiği için onun yerinde Proxmox ın kendi içerisinde ZFS le devam ettim. Yedeklemeler ve NFS paylaşımları Container içerisinde çalışıyor.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/home-lab/proxmox-pic.jpg" /></div>  

## Network
Şimdi network kısmına gelirsek. Burada en başından beri istediğim aslında Unifi ekosistemi ile ilerlemekti. Ancak hem bulunabilirlik hem de fiyat açısından çok cazip gelmedikleri için TP-Link'in Omada ekosistemi ile ilerledim. Şu an sistemde 4 cihaz var. Bir gateway, bir POE switch ve iki AP. Görevleri nispeten basit. AP ve bir hikvision kamera POE switchten güç alıyor. Bütün VLAN lar gateway kontrolünde ve switchte yönetimini sağlıyorum. Ortamda 5 farklı VLAN var. Çok detaya girmeyeceğim ancak temel olarak DMZ, Sunucu, Misafir vb. ortamları ayırmak için kullanıyorum. Gateway tarafında kural yazabiliyorsunuz; ancak güncel firewall çözümlerine alışmış biri olarak belirtmeliyim ki kural yapısı çok kısıtlı. Hatta bazı detaylı kuralları yazamıyorsunuz bile. Ancak sistem olarak sorunsuz çalışıyor. Son olarak, bir VM içerisinde çalışan ve Omada cihazların yöneten bir yazılım var. O aynı zamanda Web ara yüzünü vs sağlıyor.

## Son Olarak
Temel olarak donanım kısmı bu kadar aslında. Tabi akıllı ev sistemlerine vs girersek orada kendim devrelerini ve programlarını hazırladığım ESP IOT cihazları var. Onlara muhtemelen akıllı ev sistemleri kısmında değinirim. Şimdilik bu kadar yeterli gibi. Sonraki yazımda network kısmına detaylı şekilde değineceğim. Şimdilik hoşçakalın.