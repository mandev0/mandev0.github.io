---
layout: post
title: ELK Stack ile SIEM Lab Ortamı | Bölüm 2
excerpt: "Elasticsearch, Kibana ve Logstash kurulumu ve ilk logun toplanması"
comments: true
---
Bu yazıda genel olarak kullanılacak servislerin kurulumundan bahsedicem. Burada ben sanallaştırma olarak Proxmox VE kullanıyorum ama tabi bu lab ortamı bundan bağımsız. Siz bu ortamı bazı kısıtlamalarla VMware Workstation ile de biraz daha basitleştirerek kurabilirsiniz. Burada Sophos firewall kurulumuna da değinmeyeceğim çünkü bu yazı için fazla olacaktır. Ayrıca kurulum yapılan ortama bağlı olarak bir firewall cihazı konumlandırmak zor olabilir. Bu nedenle şuan firewall bağımsız bir loglama ve alarm sistemi yapılandırmayı planlıyorum. Zaten bütün trafiğimiz Suricata sistemine mirror edilecek ancak yine de onun için ayrı bir yazı planlıyorum :)  

Öncelikli olarak ELK Stack sunucusu ve ardından Logstash sunucusunu konumlandıracağım. Yazıyı mümkün oldukça basit tutmaya çalışıyorum bu nedenle asıl odak noktam olan loglama ve alarm üretme senaryolarına daha çok vakit ayırabilirim.  

Son olarak ben sunucu işletim sistemi olarak Debian 10 kullanıyorum. Sadece bir tercih zorunluluk değil tabiki. Bu yazılımlar işletim sistemi bağımsız.


## ELK Stack Sunucusunun Kurulması
```bash
# Gerekli paketlerin yüklenmesi
sudo apt-get install -y apt-transport-https gnupg openjdk-11-jre curl unzip

# ELK repo için key çekilir ve eklenir
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Repo kaynak listesine eklenir
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

# Yeni repo ile güncelleme alalım
sudo apt update

# ELK stack kurulumu. Bu adımda logstash servisini aynı zamanda bu sunucuyada kuruyoruz. Yapıya sadık kalmak için siz eklemeyebilirsiniz.
sudo apt install -y elasticsearch logstash kibana

# Servisleri başlangıçta çalışacak şekilde ayarlıyoruz
sudo systemctl enable elasticsearch
sudo systemctl enable logstash
sudo systemctl enable kibana
```

### Gerekli Ayarların Yapılması
#### Elasticsearch Ayarları
```bash
# Konfig dosyasının yedeğini alalım
sudo mv /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.old

# Nano ile yeni dosya oluşturalım ve aşağıdaki #İçeriği yapıştırıp kaydedelim
sudo nano /etc/elasticsearch/elasticsearch.yml

### İçerik
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: {ServerIP} # Burada logstash erişimi için ELK sunucusunun ip adresini girmemiz yeterli
http.port: 9200
discovery.type: single-node
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: "http.p12"
```
#### Kibana Ayarları
```bash
# Konfig dosyasının yedeğini alalım
sudo mv /etc/kibana/kibana.yml /etc/kibana/kibana.yml.old

# Nano ile yeni dosya oluşturalım ve aşağıdaki #İçeriği yapıştırıp kaydedelim
sudo nano /etc/kibana/kibana.yml

### İçerik
server.port: 5601
server.host: {ServerIP} # Bu kısımda sunucunun ip adresini yada 0.0.0.0 girebiliriz. Ancak son seçenek diğer adreslerdende erişime açacaktır
elasticsearch.hosts: ["https://{ServerIP/Domain}:9200"] # Burada elasticsearch'e erişim için sunucu ip adresi kullanılabilir
elasticsearch.username: "kibana_system"
elasticsearch.password: "{PASS}" # Buradaki şifreyi daha sonra ayarlayıp buraya ekleyeceğiz, şimdilik böyle kalabilir
elasticsearch.ssl.verificationMode: {none/full} # Lab ortamı olduğu için ve aynı sunucuda oldukları için bu kısımda "none" seçilebilir. Ancak bu işlem kibananın sertifika doğrulamasını kapatacaktır
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/elasticsearch-ca.pem" ]
xpack.encryptedSavedObjects.encryptionKey: 'fhjskloppd678ehkdfdlliverpoolrgh'
```

### Elasticsearch için TLS Aktif Edilmesi
```bash
# CA oluşturalım
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca
# Enter, Enter (Varsayılan ayarlar ile devam edebiliriz)

# Yukarıda oluşturulan otorite ile bir sertifika oluşturalım
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
# Enter, Enter, Enter (Şifre vermediğimiz için boş bırakarak geçebiliriz)

# Kopyalayıp gerekli izinleri ayarlayalım
sudo cp /usr/share/elasticsearch/elastic-certificates.p12 /etc/elasticsearch/
sudo chown root.elasticsearch /etc/elasticsearch/elastic-certificates.p12
sudo chmod 660 /etc/elasticsearch/elastic-certificates.p12

# Gerekli HTTP sertifikasını oluşturalım
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil http
-Enter
-y # y seçilir
-/usr/share/elasticsearch/elastic-stack-ca.p12
-Enter (No pass)
-Enter (5y)
-Enter (N)
-Enter ( # Burada birden fazla değer girilebiliryor. Değeri girdikten sonra Enter basmak yeterli
  localhost
  elk.home # Localhost harici erişim için (Ev içerisinde kullandığım domain. Firewall boşuna çalışmasın :) )
  *.home (domain with wildcard) # Son değerden sonra iki defa Enter'a bakmak yeterli
)
-Enter (y)
-Enter (
  127.0.0.1
  (ServerIP) #Localhost harici erişim için
)
-Enter (y)
-Enter (N)
-Enter (No pass)
-Enter (zip file)

# Oluşturulan sertifikayı gerekli yerlere kopyalayıp izinlerini ayarlayalım
cd /usr/share/elasticsearch
sudo unzip elasticsearch-ssl-http.zip
sudo cp /usr/share/elasticsearch/elasticsearch/http.p12 /etc/elasticsearch/
sudo chown root.elasticsearch /etc/elasticsearch/http.p12
sudo chmod 660 /etc/elasticsearch/http.p12

# Sertifikayı kibananın kullanması için verelim
sudo cp /usr/share/elasticsearch/kibana/elasticsearch-ca.pem /etc/kibana/
```

### Servislerin Başlatılması
```bash
sudo systemctl restart elasticsearch
sudo systemctl restart logstash  
```

### Servis Şifrelerinin Belirlenmesi
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
# Sırası ile şifreler belirlenir

# Oluşturulan şifre konfig dosyasına girilebilir
sudo nano /etc/kibana/kibana.yml
# elasticsearch.password: {PASS}

# Kibana yeniden başlatılır
sudo systemctl restart kibana
```

## Logstash Sunucusunun Kurulması
```bash
# Gerekli paketlerin yüklenmesi
sudo apt-get install -y apt-transport-https gnupg openjdk-11-jre curl unzip

# ELK repo için key çekilir ve eklenir
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Repo kaynak listesine eklenir
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

# Yeni repo ile güncelleme alalım
sudo apt update

# Logstash kurulumunu başlatalım
sudo apt install -y logstash

# Başlangıçta çalışacak şekilde ayarlayalım
sudo systemctl enable logstash
```

### Gerekli Ayarların Yapılması
```bash
# Konfig dosyası oluşturulur ve #İçerik yapıştırılarak kayıt edilir
sudo nano /etc/logstash/conf.d/filebeat.conf

# İçerik
input {
  beats {
    port => 5044
  }
}
output {
  elasticsearch {
    hosts => ["https://elk.home:9200"] # Yada ELK sunucu ip adresimiz
    index => "auditbeat-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "{PASS}" # Önceki adımlarda oluşturduğumuz şifremiz
    ssl => true
    cacert => "/etc/logstash/elastic.pem" # Kibana için kullandığımız dosyayı burada kopyaladı
    ssl_certificate_verification => false # Lab ortamı olduğu için sertifika doğrulamasını kapattım
  }
}
```

## Linux Sistem için Logları Toplama
Bütün bu işlemlerimizin ardından adres çubuğuna http://{ELKIPAdresi}:5601 yazdığımız zaman bizi Kibana servisinin karşılaması gerekir. Eğer buraya kadar geldiysek bu kısımda ELK sunucumuzu kapatık temiz bir snapshot almanın tam zamanıdır. Bu kısımda sadece tek bir makine için logları toplamaya başlayacağız. Geri kalan sistemler için yapıyı bir sonraki yazıda anlatacağım.  

### İlk Loglar
<div class="mb mt images-sizing" style="text-align:center"><img src="/img/kibana/kibana_1.png" /></div>  

İlk giriş yaptığımızda bizi bu şekilde bir sayfa karşılayacak ve buradan "Add data" butonuna basarak ilk ajanımızın kurulumunu gerçekleştireceğiz. Bu ajanın kurulumu için DMZ bölgesine kurduğum temiz bir Debian 10 makinemi kullanacağım. Bu bizim hedef makinelerimizden biri olacak.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/kibana/kibana_2.png" /></div>  

Ardından "Security" altında "Auditbeat" ajanını seçiyoruz. Bu seçenek bize ne yapmamızı oldukça açıklayıcı bir biçimde sonraki sayfada anlatıyor. Tek fark bizim elasticsearch yerine logstash kullanacak olmamız. Ben debian kullandığım için DEB paketini seçiyorum.  

```bash
# Paketi indirip yükleyelim
curl -L -O https://artifacts.elastic.co/downloads/beats/auditbeat/auditbeat-7.12.0-amd64.deb
sudo dpkg -i auditbeat-7.12.0-amd64.deb

# Konfig dosyasını ayarlayalım
sudo nano /etc/auditbeat/auditbeat.yml

# setup.kibana: altında host değeri için
host: "http://elk.home:5601" # yada sunucu ip adresini kullanabilirsiniz

# output.elasticsearch: satırını başına "#" koyarak yorum satırı haline getirelim
# Ardından output.logstash: başındaki "#" işaretini kaldırıp, altındaki değerleri aşağıdaki gibi değiştirelim
hosts: ["10.10.70.2:5044"] # logstash sunucusuna ait ip adresi (Burada logstash için DMZ bölgesindeki ip adresini kullanıyoruz)
index: auditbeat

# Servisi başlatıyoruz
sudo service auditbeat start

# Template'lerin aktarılması
sudo auditbeat export template > auditbeat.template.json

# Çıktısını aldığımız dosyayı ELK sunucusuna yada erişebileceğimiz bir sunucuya kopyalayalım
# Ardından curl ile template bilgisini yükleyelim
curl --user elastic:{PASS} -XPUT -H 'Content-Type: application/json' https://localhost:9200/_template/auditbeat-7.12.0 -d@auditbeat.template.json --insecure
```  

Bu işlemlerin ardından verileri ELK servisine geliyor olması lazım. Artık tek eksik bir Index Pattern eklemek kalıyor. Bunun için Kibana içerisinde sol menüden "Stack Management" ardından "Index Pattern" seçilir. "Create Index Pattern" butonuna basılır ve aşağıda gösterildiği gibi "auditbeat-*" indexi eklenir.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/kibana/kibana_3.png" /></div>  

İkinci adımda "@timestamp" seçilir ve index oluşturulur.  

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/kibana/kibana_4.png" /></div>  

Bu işlemin ardından artık linux sisteme ait logları sol menüden "Security" -> "Overview" altından görüntüleyebilirsiniz.

<div class="mb mt images-sizing" style="text-align:center"><img src="/img/kibana/kibana_5.png" /></div>  

Bu yazınında sonuna gelmiş bulunuyoruz. Oldukça uzun bir yazı oldu ve artık senoryalarımızı çalıştırmaya biraz daha yaklaşmış bulunuyoruz. Bundan sonraki yazımda Suricata kurulumu ve geri kalan sistemlerin loglarını ELK servisine aktarıyor olacağız. Bir sonraki yazıda görüşmek üzere :)
