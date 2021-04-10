---
layout: post
title: ELK Stack ile SIEM Lab Ortamı | Bölüm 3
excerpt: "Suricata servisinin kurulması ve logların toplanması"
comments: true
---
Bu yazıda Suricata servisini IDS olarak kurulumunu gerçekleştireceğiz. Daha önceki yazıda bahsettiğim gibi Suricata sunucusunun iki network ayağı olacak biri erişim için diğeri ise bütün network trafiğini mirror ettiğimiz bir ayak olacak. Trafiği mirror yapmak için ben TP-link TL-SG108E modelini kullanıyorum. Oldukça hesaplı bir cihaz. Eğer bu sistemi VMware ESXI üzerine kurduysanız kendi içerisinde SPAN(Mirror) portu ayarlayablirsiniz. Kurulum işlemini basit tutmak açısından kaynaktan derlemek yerine paket yöneticisi ile kuracağım. Bu nedenle son sürümü kullanıyor olmayacağız ama bu senaryo için çok mühim değil.

## Suricata Sunucusunun Kurulması
```bash
# Repo ile kurulumu başlatalım
sudo apt-get install suricata suricata-update
# Hyperscan paketinin yüklenmesi ile ilgili bir soru gelirse evet seçelim
```

### Suricata Ayarları
```bash
# Komut ile ayarlı yapacağımız dosyayı açalım
sudo nano /etc/suricata/suricata.yaml

### İçerik
# HOMENET subnetini belirlemek için kullanılır. Birden fazla değer vermek için virgül kullanılabilir
HOME_NET: "[10.10.0.0/16]"

# 'tls-log' altında aşağıdaki değerleri gösterildiği şekilde değiştirelim. Bu şekilde loglarda TLS SNI bilgileri gösterilecek
enabled: yes      # Log TLS connections.
custom: yes       # enabled the custom logging format (defined by customformat)
customformat: # Bu kısım varsayılan ayarları ile kalabilir. Github sisteminde hata çıkardığı için bu kısmı sildim

# Dinleme yapılan ağ arayüzünü değiştirelim
# 'af-packet' altında aşağıdaki değeri belirleyelim.
interface: ens19

# Kurallarımızı güncelleyelim
sudo suricata-update

# Suricata servisini yeniden başlatalım
sudo service suricata restart
```
### Suricata Loglarının ELK Servisine Gönderilmesi
Önceki yazımızda olduğu gibi soldaki menüden "Security" seçildikten sonra sağ üst köşeden tekrar "Add data" butonunu kullanarak gelen ajanlar arasından "Suricata logs" seçeneğini seçelim. Suricata ile ELK aynı ortamda bulundukları için ve kurulum kolaylığı sağlamasından dolayı bu kurulumda direkt olarak logları Elasticsearch servisine yönlendireceğiz.
```bash
# Öncelikli olarak deb paketini Suricata sunucumuza indirelim
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.0-amd64.deb

# Paketi yükleyelim
sudo dpkg -i filebeat-7.12.0-amd64.deb

# Konfig dosyasını ayarlayalım
sudo nano /etc/filebeat/filebeat.yml

### İçerik
# setup.kibana altında
host: "http://elk.home:5601"

# Son olarak aşağıdaki bilgileri girelim
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["https://elk.home:9200"]

  # Protocol - either `http` (default) or `https`.
  protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: "{PASS}"
  ssl.verification_mode: none # Bu sertifika onaylamasını kapatacaktır

# Suricata modülünü aktif edelim
sudo filebeat modules enable suricata

# Kurulum ve servisi yeniden başlatma işlemlerini yapalım
sudo filebeat setup
sudo service filebeat start
```  

Bu komutların ardından "Check Data" butonuna basarak logların geldiğini doğrulayabiliriz. Bu şekilde artık hem Suricata loglarını hemde hedef linux sistemimizin logları ELK servisimize gelmeye başlamış oldu.

## Windows Makine için Loglama kurulumu
Diğer bir hedef makinemiz olan windows kurulumu için öncelikle sisteme sysmon kurulumu gerçekleştireceğiz.
```bash
https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon adresinden sysmon aracını indirin

# Zipli dosyayı açtıktan sonra admin yetkisine sahip bir Powershell ile açtığınız zip dosyasının klasörü içerisine girin ve aşağıdaki komutu çalıştırın
sysmon64 -i # Şimdilik varsayılan config ile yükleyeceğiz
```

Ardından winlogbeat aracını kuracağız. Aynı önceki ajan seçimlerinde olduğu gibi "Add data" butonunu kullanalım ve "Windows Event Log" ajanını seçelim.
```bash
# Aşağıdaki adresden ajanı indirelim
https://www.elastic.co/downloads/beats/winlogbeat

# İndirilen zip dosyasını "C:\Program Files" klasörü altına çıkaralım. Çıkardığımız klasörün adını "Winlogbeat" olarak değiştirelim
# Klasörün içerisinde iken admin yetkisi ile Powershell çalıştıralım ve aşağıdaki komutu başlatalım.
.\install-service-winlogbeat.ps1

# Konfig dosyasını değiştirelim
# "winlogbeat.yml" isimli dosyayı notepad ile açalım
# Bu kısımda da logstash kullanacağız ancak bu sefer farklı bir port kullanıcaz
# output.elasticsearch ve altındaki alanları yorum satırı haline getirelim
# output.logstash için yorum satırını kaldıralım ve host bilgisini aşağıdaki gibi girelim
hosts: ["10.10.70.2:5045"] # yada kendi yapınıza göre ancak port numarası önemli

# Son olarak aşağıdaki komut ile servisi başlatalım
Start-Service winlogbeat
```

### Logstash Eklentisi
Daha önce filebeat için bir konfig dosyası oluşturmuştuk şimdi aynı işlemi winlogbeat için yapacağız. Logstash sunucusuna SSH ile bağlandıktan sonra,
```bash
sudo nano /etc/logstash/conf.d/winlogbeat.conf

### İçerik
input {
  beats {
    port => 5045
  }
}
output {
  elasticsearch {
    hosts => ["https://elk.home:9200"]
    index => "winlogbeat-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "{PASS}"
    ssl => true
    cacert => "/etc/logstash/elastic.pem"
    ssl_certificate_verification => false
  }
}

# ve servisi yeniden başlatalım
sudo service logstash restart

# Template'lerin aktarılması
.\winlogbeat.exe export template > winlogbeat.template.json

# Çıktısını aldığımız dosyayı ELK sunucusuna yada erişebileceğimiz bir sunucuya kopyalayalım
# Ardından curl ile template bilgisini yükleyelim
curl --user elastic:{PASS} -XPUT -H 'Content-Type: application/json' https://localhost:9200/_template/winlogbeat-7.12.0 -d@winlogbeat.template.json --insecure
```
Filebeat ajanında olduğu gibi tekrar "Index Pattarn" eklemek kalıyor. Bunun için Kibana içerisinde sol menüden "Stack Management" ardından "Index Pattern" seçilir. "Create Index Pattern" butonuna basılır "winlogbeat-*" indexi eklenir.  

Bu adımla birlikte artık ELK sunucumuza Suricata, hedef linux ve windows sunucularımızdan loglar gelmeye başladı. Şu aşamada sadece Suricata üzerinde kurallarımız açık. Bir sonraki yazıda ELK üzerindeki bazı "Detection" kurallarını inceleyip aktif edecek ve bir kaç deneme yapacağız.
