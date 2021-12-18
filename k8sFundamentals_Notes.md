# KUBERNETES

- Konteyner orchestrate aracidir.

- Modullerden olusuyor
- Declerative metod kullaniyor
- Automation (Current state i surekli gozluyor ve current state i desired state de tutmaya calisiyor)

# Kubernete Yonetim Components

## Control Plane-Master Node

`Controlplane` yani yonetim kismi master node uzerinde calisir. Bu componentlerin hepsi tek bir linux isletim sistemine kurulabilecegi gibi highly availibility icin birden fazla sisteme de kurulabilir. `Etcd` tamamen bu yapidan ayri bir sekilde konuslandirilarak daha yuksek bir erisim saglanabilir iken ama genellikle `etcd` de api-server in kostugu yerde durur. Ornegin 3 sunucu uzerinde 3 `api-server` kurulup diger componentler bu sistemlere dagitilabilir. Bu uzerinde control-plane componentlerinin kostugu sistemlere cluster altinda master node deriz. Master nodelar sadece bu sistemleri barindirmak icin bulunur ve production ortaminda herhangi bir is yukumuzu bu sistemler uzerinde calistirmayiz. Bunlarda sadece cluster in yonetim altyapisini kostururuz. Is yuklerimiz ise worker nodelar uzerinde calistirilir. Bunlar uzerinde bir container run time barindiran ve clustera dahil edilmis sistemlerdir. 

###  1. Kube-apiserver: Resepsiyon ornegi
Kube-apiserver, Kubernetes API'ini ortaya cikaran Kubernetes `control plane`in en oneli bileseni ve giris noktasidir. Tum diger komponent ve node bilesenlerinin direk iletisim kurabildigi tek komponenttir. Kubernetes ile alaki tum komponentlerin ve dis dunyadan kubernetes platformu ile iletisim kuran tum servislerin ortak giris noktasidir. API server tum komponent ve node bilesenlerinin direk iletisim kurabildigi tek komponenttir. Tum iletisim api server uzerinden gerceklestirilir. Kube-apiserver k8s de kaynak olusturma islemlerinde API dogrulamasindan sorumludur. Kullanicilar kubectl komut satiri istemcisi veya rest-api cagrilari araciligiyla api-server ile iletisim kurabilirler. Kisacasi disaridan istek gerceklestirmek icin api-server a ulasir `authantication` ve `authorization` islemlerini gercektlestirir ve kubernetese ulasirsiniz. Diger tum komponenetler de isteklerini api server uzerinden gerceklestirir.

### 2. Etcd: Pano
Tum cluster verisi metadata bilgileri ve k8s de olusturulan tum objelerin bilgilerinin tutuldugu anahtar-deger `key-value` veri deposudur. Clusterin mevcut durumu ile ilgili bilgiyi uzerinde barindirir. Kisacasi k8s in calismasi icin ihtiyac duyulan tum bilgiler etcd uzerinde tutulur. Api-server haric baska hicbir komponent etcd ile dogrudan haberlesemezler. Etcd ile iletisim kurmak istediklerinde bunu api-vserver uzerinden gerceklestirirler. 

### 3. Kube-scheduler: Uretim planlama
Yaratilmak istenen podlarin gereksinimlerini kontrol ederek o podun en uygun calisacagi worker node un hangisi oldugunu tespit eder ve o podun burada olusturulmasini saglar. Podun ozel gereksinimlerini isteklerini, `cpu` gibi cesitli parametreleri elinde bulundurarak degerlendirir ve pod icin en uygun node un hangisi olduguna karar verir. 

### 4. Kube-controller-manager: Vardiya amiri
Tek bir binary olarak bulunsa da icinden birden fazla controler bulundurur. K8s mevcut durumu istenilen durumla mevcut durum arasindaki karsilastirmada uygulama yonetimi saglar. Kube-controller altindaki controller managerlar k8s den istenilen durumla mevcut durum arasinda fark olup olmadigi gozlerler. Kube-api araciligiyla etcd de saklanan cluster durumunu inceler ve eger mevcut durum ile istenilen durum arasinda fark varsa iste bu farki olusturan kaynaklari gerektigi gibi olusturarak, guncelleyerek veya silerek bu durumu esitler. Ornegin siz k8s e uygulamanizin 3 pod olarak calismanizi bildirdiniz. K8s de bunu gerceklestirdi ve uygulamanizin kostugu 3 pod calismaya basladi ama sonra bir tanesi silindi iste controller-manager bu podun yeniden olusturulmasini saglar. 

## Worker Node
***
Her worker node da 3 temel component bulunur. Ilk ve en onemli component containerlarin calismasini saglayacak bir `container run time` dir. Default olarak bu docker dir ancak k8s docker container run time destegini birakarak artik `containerd` ye gecmistir.

### 1. Kubelet: Ustabasi
Her worker node'da bulunur. `Kubelet` api-server araciligiyla etcd yi kontrol eder. Scheduler tarafindan bulundugu node uzerinde calismasi gereken bir pod belirtildi ise kubelet bu podu o sistemde yaratir. Conatinerd ye haber gonderir ve belirlenen ozelliklerde bir container in o sistemde calismasini saglar.

### 2. Kube-proxy: Lojistik elemani
Her worker node da calisir. Olusturulan podlarin tcp, udp ve etcdp trafik akislarini yonetir, ag kurallarini yonetir. Kisaca `network proxy` gorevi gorur.
Bunlarin disinda DNS ya da gui hizmeti saglayan cesitli servisler entegre edilebilen servislerdir fakat bunlar core componentler olarak adlandirilmaz.

# Kubectl Installiation

Kubernetes e komut vermek icin bunu API-server uzerinden yapiyoruz. 

Bunun 3 yontemi var. 
1. Rest cagrilari:
Uygulamalarin ya da scriptlerin icinde kullanilir.
2. Gui istemcileri:
Grafiksel arayuz ile iletisim kurmak. Resmi gui araci dashboard, oktant, land en bilinen araclardir. Gui istemciler en temel araclar degildir. 
3. Kubectl:
Shell uzerinden apiserver a komutlar gonderdigimiz k8s in resmi cli aracidir. k8s cluster olusturulmadan once ilk olarak kubectl yuklenmesi gerekir.

## Kubectl kurulum
Linux icin oncelikle ```brew``` kurulumu
Sonrasinda ```brew install kubectl`` ile kubectl kurulumu tamamlaniyor

## Kubectl config dosyasi
- kubectl araci baglanacagi kubernetes cluster bilgilerine config dosyalari araciligiyla erisir.
- config dosyasinin icerisinde kubernetes cluster baglanti bilgilerini ve oraya baglanirken kullanmak istedigimiz kullanicilari belirtiriz.
- Daha sonra bu baglanti bilgileri ve kullanicilari ve ek olarak namespace bilgilerini de olusturarak contextler yaratiriz.
- kubectl varsayilan olarak `$HOME/.kube/` altindaki config isimli dosyaya bakar ama bunu `KUBECONFIG` environment variable degerini degistirerek guncelleyerebilirsiniz.
-  Context kismi kubeconfig dosyasindaki baglanmask istedigimiz cluster a hangi user ile baglanacagimizi ve hangi namespace e erisecegimizi contextler yaratarak belirleriz. Kullanici ile serverlarin birlestirildigi bir kisimdir context.

## Kubectl komutlari
kubectl config dosyası `$HOME/.kube/` konumunda bulunan `config` isimli dosyadır.

Linux ve Mac `/home/serdem/.kube/config` | Windows `C:\Users\serdem\.kube\config`

Mevcut kubectl client ve Kubernetes cluster versiyonlarını görmek için:
`kubectl version`

Config dosyasında tanımlı context'leri listeleme:
`kubectl config get-contexts`

Seçili durumdaki context'i gösterme.
`kubectl config current-context`

config dosyasında tanımlı contextlerden bir tanesini geçerli context olarak atama. 
`kubectl config use-context "context_ismi"`

Geçerli context'de tanımlı Kubernetes cluster hakkında bilgi edinme.
`kubectl cluster-info`

Varsayılan namespace'de bulunan objeleri listeleme
`kubectl get "obje_tipi"` | `kubectl get pods`

Varsayılan namespace'de bulunan tek bir objenin özelliklerini listeleme
`kubectl get "obje_tipi" "obje_ismi"` | `kubectl get pods testpod`

Varsayılan dışında başka bir namespace'de bulunan objeleri listeleme (_-n "namespace_adi" ekiyle komutların belirtilen namespace üstünde çalıştırılması sağlanmaktadır_)
`kubectl get "obje_tipi" -n "namespace_adi"` | `kubectl get pods -n kube-system`

Tüm namespace'lerde bulunan objeleri listeleme (_--all-namespaces ya da kısaca -A eki komutun tüm namespace'leri kapsamasını sağlar_)
`kubectl get "obje_ismi" --all-namespaces` | `kubectl get pods --all-namespaces`

kubectl komutlarının geri döndürdüğü çıktı kısıtlı bilgiler içerir ve plain-text olarak döner. **-o** opsiyonuyla bu çıktı belirtilen başka bir formatta döndürülebilir. Örneğin **-o yaml** opsiyonu çıktının yaml formatında döndürülmesini sağlar. Kullanılabilecek opsiyonlar:
- `-o yaml` Yaml formatında çıktı döndürür.
- `-o json` Json formatinda çıktı döndürür.
- `-o wide` Plain-text fakat daha çok bilgi içerir.
- `-o jsonpath=<template>` Jsonpath ifadesinde tanımlanan alanları döndürür.
- `-o name` Sadece isimleri döndürür. 
- `-o custom-columns=<spec>` Comma-seperated olarak belirtilen kolonlardan bir table döndürür. 
  
`kubectl get "obje_tipi" "obje_ismi" -o "opsiyon_ismi"` | `kubectl get pods -o wide`

Kubernetes obje tipleriyle ilgili detaylı bilgi edinmek..
`kubectl explain "obje_tipi"` | `kubectl explain pods`

# Kubeadm ile Kubernetes Cluster Kurulumu

## kubeadm kurulum

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

**0:** sanal makine oluşturma
```
$ multipass launch --name master -c 2 -m 2G -d 10G
$ multipass launch --name node1 -c 2 -m 2G -d 10G
```

**1:** Iptables bridged traffic ayarı

```
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

```
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
$ sudo sysctl --system
```

**2:** containerd kurulumu

```
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

```
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

```
$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

```
$ sudo sysctl --system
```

```
$ sudo apt-get update
$ sudo apt-get upgrade -y
$ sudo apt-get install containerd -y
$ sudo mkdir -p /etc/containerd
$ sudo su -
$ containerd config default  /etc/containerd/config.toml
$ exit
$ sudo systemctl restart containerd
```

**3:** kubeadm kurulumu


```
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

**4:** kubernetes cluster kurulumu

```
$ sudo kubeadm config images pull

$ sudo kubeadm init --pod-network-cidr=172.31.1.64/16 --apiserver-advertise-address=<ip> --control-plane-endpoint=172.31.1.64
```

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
$ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
$ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

# Pod

- Kubernetes icinde yaratilan en temel objedir.
- Kubernetesde olusturabileceginiz en kucuk birimdir.
- Podlar bir yada daha fazla container barindirabilir ancak cogu durumda pod tek bir container barindirir.
- Her bir pod un essiz bir id si `uid` bulunur
- Her pod essiz bir ip adresine sahiptir.
- Ayni pod icersindeki containerlar ayni node ustunde calistirilir ve bu containerlar birbirleriyle `localhost` ustunden haberlesebilirler.
- Kubectl ile kube-api ile haberleserek k8s uzerinde pod yaratma islemi gerceklestirilir. Api-server bu poda bizim tanimladigimiz bilgileri atar ve bir pod yaratir ve etcd veri tabanina kaydedilir. `Kube-scheduler` componenti surekli burayi gozler ve herhangi bir worker node uzerinde atamasi yapilmamis pod tanimi yapilmamissa o podun calismasi icin uygun bir worker node secer ve bu bilgiyi pod tanimina ekler. Sonrasinda worker node uzerinde calisan `kubelet` servisi de bu etcd yi surekli gozledigi icin bu pod tanimini gorur ve bu tanimda belirtilen container o worker node uzerinde olusturulur ve boylece pod olusturulma asamalari tamamlanir.

// Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
| `kubectl run nginx --image=nginx --dry-run=client -o yaml`

Imperative yöntemle pod oluşturma.
`kubectl run "pod_ismi" --image="image_ismi" --restart=Never` | `kubectl run firstpod --image=nginx --restart=Never`

Bir objenin ayrıntılı özelliklerini görmek. 
`kubectl describe "obje_tipi" "obje_ismi"` | `kubectl describe pods firstpod`

Bir pod objesinin loglarını görüntüleme. _(-f opsiyonu çıktıya yapışmanızı ve anlık olarak üretilen logları görmenizi sağlar)_
`kubectl logs "pod_ismi"` | `kubectl logs firstpod` | `kubectl logs -f firstpod`

Pod'da komut çalıştırma. _(Eğer pod içerisinde birden fazla container varsa **-c "container_ismi"** opsiyonu ile komutun çalıştırılması istenilen container belirtilebilir)_
`kubectl exec "pod_ismi" -- "komut"` | `kubectl exec firstpod -- printenv` | `kubectl exec firstpod -c container1 --printenv` 

Pod'a shell bağlantısı oluşturma. _(Eğer pod içerisinde birden fazla container varsa **-c "container_ismi"** opsiyonu ile komutun çalıştırılması istenilen container belirtilebilir)_
`kubectl exec -it "pod_ismi" -- "shell_konumu-yada-komutu"` | `kubectl exec -it firstpod -- /bin/sh` | `kubectl exec -it firstpod -c container1 -- /bin/sh` 

Bir Kubernetes objesini silme. 
`kubectl delete "obje_tipi" "obje_ismi"` | `kubectl delete pods firstpod`

Json ya da Yaml formatında hazırlanmış bir konfigurasyon dosyası aracılığıyla yeni obje oluşturma. 
`kubectl apply -f "dosya_yolu/dosya_ismi"` | `kubectl apply -f ./pod1.yaml`

Oluşturulan objeyi dosya kullanarak silme
`kubectl delete -f "dosya_yolu/dosya_ismi"` | `kubectl delete -f ./pod1.yaml`

Label ve port konfigurasyonlarını da ekleyerek bir pod oluşturma. 
`kubectl run "pod_ismi" --image="image_ismi" --port="port_numarası" --labels"anahtar:değer_eşlenikleri" --restart=Never` | `kubectl run secondpod --image=nginx --port=80 --labels="app=front-end,team=developer" --restart=Never`

Cluster'da bulunan bir Kubernetes objesini varsayılan text editörü ile açarak özelliklerini güncelleme. 
`kubectl edit "obje_tipi" "obje_ismi"` | `kubectl edit pods firstpod`

kubectl komutları sonucu oluşan çıktıyı tek seferlik görmek yerine değişiklikleri izleyebilme imkanına **-w** opsiyonu ile kavuşuruz _(linux dünyasındaki **watch** komutunun yaptığı işin bir benzerini **-w** opsiyonu sağlar)_
`kubectl "komut" -w` | `kubectl get pods -w`

kubectl'in çalıştırıldığı bilgisayar üzerinden cluster'da bulunan bir objeye tünel açılarak bağlantı kontrolü yapılması "detaylarına network konusunda geleceğiz"
`kubectl port-forward "obje_tipi"/"obje_ismi" "local_port":"hedef_port"` | `kubectl port-forward pod/multicontainer 8080:80`

```yaml
apiVersion: v1 # obje tipinin hangi API de tanimlandigi (her objenin belli bir API i var ve bunu bilerek buraya yazmak gerekiyor) Nasil bilebiliriz? dokumantasyondan veya explain ile
kind: Pod # obje tipi
metadata: # unique bilgiler tanimlanir
  name: firstpod
  labels: # labellar tanimlaniyor (objenin betimlendigi alan)
    app: frontend
spec: # objenin ozellikleri (her objeye gore farklilasir)
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

## Pod Yasam Dongusu
1. Pending:
Pod yaratmak icin gonderilen yaml dosyasi kube-api tarafindan alinir ve varsayilan ayarlari da ekleyerek poda bir unique id atar ve tum bunlari etcd ye kaydeder. Bu asamadan sonra pod olusturulmustur ve bu asamadan sonra pending asamasina gecer. 
Eger bir podun statusu pending durumunda ise bunun anlami sudur: birisi bir pod olusturdu podla ilgili tanimlar yapildi ve veri tabanina kaydedildi ama pod herhangi bir node uzerinde olusturulmadi. 

2. Creating:
Kube-scheduler api-server araciligiyla surekli etcd yi gozler. Eger burada yeni yaratilmis ve herhangi bir node atamasi yapilmamis bir pod gorurse calismaya baslar. Algoritmasi ve secme kriterlerine gore podun calismasinin en uygun olacagi node u secer ve veri tabanindaki pod objesine node bilgisini ekler. Bu noktadan itibaren pod yasam dongusunde creating asamasina gecer. _Eger bir pod creating asamasina gecemiyorsa bu kube-scheduler in uygun bir node bulamadigi_ anlamina gelir. Bunun pek cok nedeni olabilir: cluster da bu podun gereksinimlerini karsilayabilecek uygun bir node bulunmuyor olabilir veya nodelar uzerinde `cpu` ve `memory` gibi kaynaklar tukenmis olabilir. 

3. ImagePullBackOff:
Clusterdaki tum nodelar uzerinde kubelet adli bir servis calisir. Bu servis ayni kube-scheduler gibi surekli etcd veri tabanini gozler ve bulundugu node a atanmis podlari tespit eder ve hemen islemlere baslar. Ilk olarak pod taniminda olusturulacak containerlara bakar ve bu containerlarda olusturulacak image lari sisteme indirmeye baslar. Eger bir sekilde image indirilemezse oncelikle pod erimagepull ve ardindan da imagepullbackoff asamasina gecer. Eger podun statusunda `imagepullbackoff` gorurseniz bu node un image i repository den cekemedigi ve bunu tekrar tekrar denemeye devam ettigi anlamina gelir. Bunun birkac nedeni olabilir: en sik karsilasilan neden pod tanimlanmasinda image isminin yanlis yazilmasidir. Image ismi yanlis yazildigi icin image cekilemeyecektir ya da image i cekmek icin sisteme kaydolmak gerektigi ve bu authentication islemlerinde hata yapilmasidir. Bu nedenle image cekilemeyecektir. Bu gibi bir hata olmamasi durumda ise diger asamaya gecer.

4. Runnig:
Islemlerin sorunsuz bir sekilde ilerlemesi halinde kubelet node da bulunan container engine ile haberlesir ve ilgili containerlarin olusturulmasini saglar ve containerler running durumuna gecer. Bu noktadan itibaren artik pod olusturulmus olur.

**- Containerlarla ilgili temel kural:** Container icindeki uygulama calistigi surece container calisir. Uygulama calismayi birakirsa container durdurulur. Uygulamanin da durdurulmasi 3 sekilde olur: 
   1. isi bittigi icin hata vermeden otomatik olarak kapanir. 
   2. Kullanicinin istegi uzerine komut gonderilir ve hata vermeden kapanir. 
   3. Hata verir, sikinti cikar, hata kodu olusturarak kapanir. Ornegin nginx image uzerinden bir container yarattigimizi dusunelim. Bu uygulama bir deamon bir servis yani siz onu durdurana ya da hata cikip cokene kadar calismaya devam eder. O calistigi surece de container calismaya devam eder. Fakat diyelim ki siz bir image yarattiniz ve bu image in varsayilan uygulamasi da bir script ya da basit bir komut. Container baslayinca bu script calisiyor isini yapiyor ve isi bitince kapaniyor yani container da kapaniyor yani kapanmasi icin illa ki hata vermeden kapanmasina gerek yok.
Bu gibi sorunlar nedeni ile containerlar icin restart policy tanimi yapilir. 

Restart policy 3 adet deger alabilir:
   1. **Always:** Default degerdir. Pod un icerindeki container her durumda durdurulursa durdurulsun her sartta yeniden baslat anlamina gelir. 
   2. **On-failure:** Sadece hata alip kapanirsa yeniden baslatilir.
   3. **Never:** Pod hicbir zaman yeniden baslatilmaz.


5. Succeed:
Podun altindaki containerlar calismaya devam ettikce status running olarak devam eder. Eger containerlarin hepsi hata vermeden dogal olarak kapanirsa ve restart policy never veya onfailure olarak set edildi ise podun statusu succeed statusune gecer ve pod yasam dongusunu tamamlar. Yani bir diger degisle completed, basarili olarak pod un yasam dongusu tamamlanir. 

6. Failed
`Never` ya da `Onfailure` olarak policy tanimlandi ve containerlardan biri hata verip kapandi ise bu sefer pod un statusu failed olarak isaretlenir ve yasam dongusunu boyle tamamlar. 

7. CrashLoopBackOff
Fakat restart policy `always` olarak set edildi ise pod hicbir zaman succeed ya da failed durumuna gecmez. Bunun yerine podun icerisindeki container yeniden baslatilir ve running state de devam eder. Fakat kubernetes bu yeniden baslatma islemini belirli bir siklikla yapiyorsa bazi seylerin ters gittigine kanaat getirir ve pod u `CrashLoopBackOff` adini verdigimiz bir state e sokar. Bunun anlami sen bir pod olusturdun ama bu pod un icerisindeki container ikide bir kapaniyor buna bir bak. Bu pod da sunlar olur: k8s container in icerisinde birseylerin ters gittigini anlar ve pod un statusunu `crashloopbackoff` a cevirir. Surekli restart etme yerine; restart eder 10 sn bekler eger 10 sn icinde yeniden cokerse 20 sn bekler yeniden cokerse 40 sn bekler sonra 80 sn bekler ve bu durum 5dk lik araliga cikana kadar boyle devam eder ve ondan sonra her 5dk da bir tekrar eder. Bu arada container cokmeyi birakir ve 10dk sure ile sorunsuz bir sekilde calismaya devam ederse. Kubelet container i crashloopbackoff dan cikarir ve running e dondurur. Eger bu olmaz ve siz mudahale etmezseniz bu durum sonsuza kadar boyle devam eder. _Ozetle crashloopbackoff mudahaleyi gerektirir ve bakilmasi gerekmektedir._

## Multicontainer pod (sidecar container)
Bir frontend ve bir de backend den olusan two tier yani cift katmanli bir uygulama yazdik. Cok populer olan ornegin wordpress uygulamasi. WordPress uygulamasini deploy etmek istedigimizi varsayalim. Bildigimiz uzere wordpress php tabanli bir frontend ve bu uygulamanin verilerinin tutuldugu bir mysql veri tabanina sahip. Bu uygualamyi container haline getirmek istiyorum. Bu uygulamayi icinde mysql ve wordpress halinde calisan tek bir container haline getirebilir miyim? Teknik olarak evet. Fakat bunu yapmiyoruz. Her iki uygulama icin de ayri iki container kullaniyoruz. Bunun iki nedeni var 
  1. container in temel mantigi izolasyon bu iki uygulamayi da ayni container icine koymak bu izolasyondan mahrum olmak demek. 
  2. Iki uygulamayi scale etmek istemeniz durumunda yatay buyume yapamiyorsunuz. 

Bunu soyle dusunun ben 2 sunucudan olusan bir ortam kurdum. Hem wordpress hem de mysql uygulmasini ayni container icine kurdum ve tek bir uygulama olarak calistirmaya basladim. Sayfama yogun bir istek oldugunda wordpress in frontend katmani bu isteklere cevap verememeye basladi ben de yeniden bir container daha olusturarak load balancer arkasina almak ve kaynaklarimi cogaltmak istedim ki bu problemi cozebileyim. Ama bunu yaptigim zaman ortamda 2 frontend 2 tane de mysql veri tabani olacak. Ancak benim veri tabanimda veri sikintisi yoktu. Ikisi de ayni container da oldugu icin bunu coklamam gerektiginde ikisini birden cokladim. Hatta 2. containeri deploy ettikten sonra mysql baglanti ayarlarini degistirdim ve 1. container icindeki mysql e baglanmasi icin ayar yaptim cunku simdiye kadar olusturdugum her sey o veri tabaninda. Bunun 100 container a kadar scale edilen bir ortam oldugunu dusunun. Sirf 2 uygulamayi da ayni container icine gomdugum icin sikintilar yasadim. Bunun yerine bir mysql ve bir de wordpress olmak uzere 2 ayri image yaratsa idim bu sefer 1 tane mysql container yaratilirdi benim de istedigim kadar wordpress scale edebilme imkanim olusurdu. Bu nedenlerden dolayi en onemli best practice bir container icine 1 tane uygulama koymak. 
Kubernetes icinde ise wordpress icin bir pod mysql icin bir pod olusturulmali. Fakat istersem bir pod icinde birden fazla container da koyabilirim ama bu da bir container icine iki uygulama koymak ile ayni sey cunku k8s de scale ettigimiz sey pod. Her container da tek bir uyguama her pod da bir container. 

*Peki neden kubernetes ayni pod icinde birden fazla container calismasina izin veriyor?* 
Diyelim ki ben wordpress php uygulamasinin uygulama ici performans degerlerinin merkezi bir yerde toplayarak analiz etmek istiyorum. Bunu yapabilecek bir uygulamayi deploy etmek istiyoruz. Bunu k8s de nasil yapabiliriz? Oncelikle mysql podumuzu ayaga kaldirdik sonrasinda wordpress uygulamamiz agaya kalkti son olarak da yeni uygulamanin ayaga kalkacagi podu yarattim. Son uygulamanin tek bir amaci var wordpress e baglanacak ve uygulamayi analiz edecek veriyi toplayacak yani bu son uygulamam wordpress e bagimli. Bu uygulama wordpress hangi worker node da olusturuluyorsa orda olustrulmali. Wordpress calismaya basladigi zaman calismali kapandigi zaman kapanmali yani tamamen ona bagimli. Yani ben 2. bir wordpres uygulamayi yaratirken 2. bir uygulama daha deploy etmem gerekecek. 3. bir wordpress uygulamasi icin 3. log kodunu olusturuacgim. Wordpress i silerken bu uygulamadan bilgi toplatan podu da silmem gerekecek. Bu cok zaman alan bir islem surekli iki is yapmam gerekiyor. Diger bir sikinti ise diyelim ki benim metric toplayan uygulamam word press uygulamasi ile storage seviyesinde haberlesmesi ortak dosyalari yazmasi gerekiyor ancak 2 podun ayni local volume e baglanabilmesi icin 2 pod un ayni worker node uzerinde calismasi gerekiyor. Diyelim ki ben wordpress podunun olusturdugumda kube-scheduler bunu o an en uygun olan olan node1 da olusturdu. Sonrasinda analiz verisi toplayan uygulama olusturmak istedim ve kube-scheduler bunu node 1 uzerinde yeterli kaynagim yok diye node2 uzerinde olusturdu. Bu uygulamalar stroge seviyesinde birbirlerine ulasamayacaklar. Bu da ayri bir sorun. K8s bu sorunlari ortadan kaldirmak icin ana uygulamaya bagimli, onunla network seviyesinde izolasyon olmadan ve gerektigi durumda ayni izolasyon altyapisini kullanabilecek ayni pod icerisinde 2 container olarak calistirma imkani sagliyor. Birlikte scale edilmesi gereken, birbirleriyle network ve storage sebiyesinde haberlesmesi gereken uygulamalari ayni pod icerisinde ayri ayri containerlar olarak calistirabiliyoruz. Buna terminolojide `sidecar` container denmektedir. 

Bir pod icerisinde birden fazla container calistirdigimiz zaman:
1. Ayni pod icerisinde tnaimlanmis tum containerlar ayni worker node uzerinde olsturuluyorlar.
2. Containerlar ayri birer unitedir fakat pod un lifecycle i icinde yonetilir. Yani pod olusturulunca iki container birden olusturulur ve silirse de iki container birden silinir.
3. Bu containerlar arasinda network izolasyonu bulunmaz yani ayni pod icerisinde bulunan a ve b containerlari birbirlerine localhost uzerinden ulasabilirler. Bunlar network bakimindan sanki ayni makinada calisan processlerdir.
4. Tek bir volume yaratilarak her iki containera da mount edilebilir boylelikle ayni dosyalar uzerinde calisabilirsiniz.

# InitContainers
`initContainers` da bir pod icerisinde birde fazla pod yaratilmasina imkan verir. Fakat app containerlardan farkli olarak `initContainers`lar pod un yasam dongusu boyunca calismaz. Siz bir pod tanimina `initContainers` tanimi koydugunuz zaman pod olusturuldugu zaman ilk olarak bu `initContainers` calistirilir. `initContainers` calisir icindeki uygulama ne yapacaksa onu yapar ve ardindan kapanir bu `initContainers`lar islerini tamamlayip kapanana kadar da app containerlar calismaya baslamaz. 

*Peki neden boyle bir seye ihtiyac duyariz ve kullanim alani nedir?* 
`initContainers`lar esas uygulamamiz calismadan once tamamlamamiz gereken seyler var ve bunlari tamamlamadan esas uygulamayi baslatmak mantikli degilse kullanmak gerekir. Mesela uygulamamizin bagimli oldugu baska bir uygulama ya da servis var. Eger bu ayakta ve hazir degilken uygulamayi baslatirsak uygulamada sikinti cikiyor. Bu durumda pod tanimina `initContainers` tanimi ekler ve bu `initContainers`in bu servisi gozlemesi icin ayar yapariz. `initContainers` icinde bir uygulama calistirilir ve bu servisten okey alana kadar calisacak okey aldiktan sonra da kapanacak sekilde ayar yapariz. Bu sayede pod olusturuldugu zaman ilk olarak `initContainers` calisir. Diger servisi beklemeye baslar. Diger servis hazir oldugu anda uygulama kapanir ve `initContainers` da kapatilir. `initContainers` kapatildigi anda da esas uygulamamizin calistigi container ayaga kalkar ve boylece servisimiz hazir olana kadar uygulama beklemis olur. 

Soyle bir senaryo dusunun ana uygulamamizin ihtiyaci olan bazi config dosyalarinin guncel halinin ana uygulama baslamadan sisteme cekilmesi gerekiyor, iste bu cekme islemini de `initContainers` ile halleder ve ana uygulama baslamadan bunlari sisteme indirebiliriz. 
Multicontainerdan farkli olarak `initContainers`lar yaml dosyalarinda tanimlanirken `initContainers` adi ile ayri bir alanda tanimlanir. Containerin isi bittikten sonra kapanir ve ardindan ana container calismaya baslar.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

# Label and selector
Label yani tagler k8s de her turlu objeye atayabildiginiz anahtar deger eslenikleridir. Etiketler sayesinde olusturdugumuz objelere bizlerin anlayacagi ve gruplara yaparken kullanabilecegimiz bilgiler eklemis oluruz. Bu sayede k8s tarafindan bizlere core ozellik olarak sunulmayan objelere belirli bir aidiyet atanmasi islemini gerceklestirebiliriz. 
Ornegin k8s de 5 ekip tarafindan olusturulan 100 den fazla pod calistiriyoruz. Hangi pod un hangi ekibe ait oldugunuz belirtgmek istersek k8s in bize sunmus oldugu bir hiyerarsi mekanizmasi yok ama labels sayesinda bunu saglayabiliyoruz. Ornegin her ekip kendi olusturdugu objeye team:teamname seklinde etiket atarsa bizler bu etiketler sayesinde hangi ekibin hangi objelere sahip oldugunu listeleme imkanina kavusuruz. Ya da uygulamamizin frontend katmanini olusturan objelere tier:front-end backend katmanini olusturan objelere ise tier:backend etiketini atayarak frontend ve backend katmanlarini ayri ayri listeleme imkanina kavusuruz. 
Etiketler obje olusturulurken atanacagi gibi objeler olustuktan sonra da atanip silinebilir. Etiketler anahtar veri degerleri seklinde olusturulur. Etiketler bir DNS subdomanin seklinde atanan opsiyonel bir on ek kismi bulundurabilir example.com/tier:frontened gibi. Bu kisim zorunlu degildir. Bu kisimdan sonra bir slash etklenir ve ana kisim yazilir bu ana kisim anahtar veri kismidir ve bu iki deger arasinda iki nokta ust uste bulunur. Bu anahtar ve deger kismi en fazla 63 karekter olabilir. Alfanumerik bir karakterle baslamali ve bitmelidir. Tire alt cizgi noktalar ve arasinda numerik degerler icerebilir ancak bunlarla baslayamazlar.
Etiketlerin bir onemli gorevi de k8s de objeler arasindaki baglanti da labels lar araciligiyla kurulur. Servisler ve deployment objeleri hangi podlar ile iliski kuracagini label lar sayesinde belirlerler. Ornegin bir servis yaratir ve ona git tier:frontend label li podu bul ve servisi bu poda yonlendir.
Yaml dosyalarinda selector araciligiyla deploymenta senin yonetecegin podlar app:frontend label ina sahip olacaklar. Bunu gorursen anla ki bu podlar senin podlarin. yaml dosyasi icindeki template alindaki labels kismina da bu app:frontend i ekliyorum ki deployment hangi pod u sececegini anlayabilsin. 

Labels ve selector kismi 3 acidan onemli
- Ilk olarak bu bir zorunlu alan. Yani deployment da en az bir selector tanimi olmali ve ayni label lar template kisminda da bulunmali.
- Egerki birden fazla deployment objesi olusturacaksaniz ki production ortaminda bu olacak, siz her deployment objesinde farkli label ve selector kullanilmalisiniz. Ayni labellar kullanilmasi sikinti cikarir.
- Ayni label lari kullandiginiz baska objeleri yaratirsaniz misal singleton bir pod yaratirsaniz bu da sikinti cikarir. 

// belirli bir labeldaki podlari goster
`kubectl get pods -l "<label ismi>"` | 

// podlarin labelslarini goster
`kubectl get pods -l --show-labels`

// equality based kullanim
`kubectl get pods -l "app=firstpod" --show-labels`

// set based kullanimi (bu kullanimin equality den farki )
`kubectl get pods -l 'app in (firstapp)' --show-labels`

//notin esit degildir anlamina gelir
//app anahtari degeri firstapp olmayan tum podlari getir
`kubectl get pod -l 'app notin (firstapp)' --show-labels`

//app anahtarini sorgula ve bunlarin icerisinden degeri firstapp olmayanlari getir
`kubectl get pod -l 'app,app notin (firstapp)' --show-labels`

// anahtarina sahip olmayanlari listele
`kubectl get pods -l '!app'`

// label ekle
`kubectl label pods pod9 app=thirdapp`

// label sil
`kubectl label pods pod9 app-`

//label guncelle
`kubectl label --overwrite pods pod9 team=team3`

//Tum podlara (namespacedeki) label ekle
`kubectl label pods --all foo=bar`

**nodeSelector** ile podun calismasini istedigimiz node u belirleyebiliyoruz. Bunu da labellar araciligiyla yapiyoruz.Bu sayede pod ile node u eslestirebiliyoruz. Mesela bazi worker nodelarda ssd var iken bazilarinda ise yavas bir disk var. Bazi uygulamalarimin hizli ssd de calismasini istiyor isem node lara bunu belirten labellar atarim ve podlarda da nodeselector ile bunu belirtir isem bu amacima ulasabilirim.

# Annotation
Aynen labellar gibi anahtar veri opsiyonu ekleyebilecegimiz 2. secenek ise annotations. Labellar k8s icin cok onemli oldugu icin label eklemek veya cikarmak clusterda birseyleri tetikleyebilir. Dolayisiyla her bilgiyi label olarak metadataya ekleyemeyebiliriz. Ancak bu bilgiler gerekli olabilir. Mesela podun ne zaman ve kim tarafindan olusturuldugu bilgisini eklemek istiyorum. Bu bilginin label olarak eklenmesi dogru degil, yani bu bilgiyi label olarak secme istegim yok veya bu bilgiyi herhangi bir secme istegi olarak kullanmak da istemiyorum. Dolayisiyla bunu annotationa eklemek daha dogru olacaktir. Ayrica annotation k8s in ana componenti olmayan fakat k8s ile baglantisi olan yazilimlar tarafindan ihtiyac duyulan bilgilerin de yazildigi yerdir. Ornegin firmamizin destek ekipleri tarafindan kullanilan bir cagri kabul uygulamasi var. Bu yazilimi k8s den bilgi cekebilecek hale getirdik ve soyle bir sistem kurmak istiyoruz: Herhangi bir k8s objesinde bir sikinti ciktigi zaman bu cagri yazilimi bunu tespit etsin ve o objenin destek sorumlusu olarak belirlenen insana mail gondersin. Bu durumda o mail adresi bilgisini objenin metadatasina annotation olarak ekleyebiliriz ve destek yazilimi bu bilgiyi cekebilir. Bu sayede de bu insana mail gonderebilir. Annotation bu ve benzeri durumlar icin kullanilir. Olusturulma kurallari label larla hemen hemen aynidir. DNS subdomain olarak eklenebilecek ve zorunlu olmayan bir prefix ile baslayabilen, sonrasinda da label lar gibi 63 karakteri gecmeyecek; nokta tire ve alt tire icerebilen ama bunlarla baslayamayan alfanumerik degerler icerebilir. Value kisminda ise bu kurallar gecerli degildir. Alfanumerik olmayan karakterler de alabilir. 

// annotation ekle
`kubectl annotate pods <pod adi> <foo=bar>`

// annotation sil
`kubectl annotate pods <pod adi> <foo>-`

# Namespace
Soyle bir senaryo kurun bi firmada calisiyoruz ve tum calisanlarin ortak calistiklari dosyalari barindirabilecekleri bir yapi tasarliyoruz. 10 ayri ekibimiz var ve bu 10 ayi ekibin de erisip dosya barindirabilecegi bir altyapi kurmak istiyoruz. Bu sistemi nasil kurariz: bir file server yaratirim ve bunun altinda tum ekibin erisebilecegi bir paylasim alani yaratirim. Sonra gider tek tek tum kullanicilarin bilgisayarlarinin buraya erismesini saglayabilirim. Baslangicta sorunsuz olabilir ancak birden fazla ekibin olmasi durumunda soru olabilir. Bu dosya altinda 100lerce dosyanin olmasi durumunda bu belli bir zaman sonra yonetilemez hal alabilir. Bir diger sikinti da su olabilir diyelim ki ben a.txt adinda bir dosya olusturdum. Baska bir arkadasim da a.txt adinda bir dosya olusturup ayni yere atamaz. Bir baska sikinti da su olabilir. Diyelim ki sadece HR calisanlarinin gormesi gereken dosyalari da burada tutuyorum. Bu dosyalarin sadece HT tarafindan gorulmesini saglamak icin dosya bazinda ayarlamalar yapmam gerekir. Her dosya upload edildigide de bu islemleri tekrarlamam gerekir. Son olarak da su sikintiyi yasariz. Herkes tek bir alana dosya kaydettigi icin ekip bazli kota ayarlamasi yapmam zor olur. Yani hr kismina 20gb alan ayirayim IT kismina 100gb alan ayirayim gibi ayarlamalar yapamam. Cozum belirli bir zaman sonra yeterli olmamaya basladi.
Her ekip icin klasor altinda ayri bir alan yaratirsak tum bu problemleri ortadan kaldiririz. Her ekibe ozel bir klasor olusturulur. Her ekip kendi dosyalarini kendi klasorlerinde tutabilirler. Ayrica it ekibi gibi ekiplere ozel proje bazli klasorler de yaratiriz. Ornegin yeni bir urun gelistirmesi yaratiyoruz ve bu urune ait gelistirme dosyalarini bu klasorde tutariz bu sayede tum guvenlik ayarlarini klasor bazda halledebiliriz. HR klasorune sadece hr calisanlari erissin it kalsorune sadece it calisanlari erissin, proje klasorunde proje yoneticisinin yazma izni olsun ama stajyer sadece dosyalari okuyabilsin diyebiliriz. Boylece guvenlik ve gruplama ile pek cok sorunun ustesinden geldik. Isimin cakismasini engelledik. Tek tek dosya bazinda kim erisebilir ayari yapmaya gerek kalmadi bunu klasor bazinda yaptik. Son olarak da klasor bazinda kota ayarlamasi yaparak kaynak kisitlamasi saglabiliyoruz. Namespace tam anlami ile bu ise yarar. 
Kubernetes cluster i bu ornekteki fileserver olarak dusunursek namespace ler de burada yarattigimiz klasorlerdir. K8s de objeleri sanal klasorler altinda olusturabilir ve bu sanal klasorler altinda olusturabilir ve sonrasinda bu sanal klasor bazinda erisim izni ve kota ayarlamasi yapabiliriz.
Her k8s kurulumunda varsayilan olarak 4 namespace olusturulur.
- `kube-sysem:` k8s tarafindan olusturulan objelerin tutuldugu namespace dir. 
- `kube-public:` kimligi dogulanmamis olanlar da dahil tum kullanicilar tarafindan erisilmesine ihtiyac duyulan objelerin olusturulabilecegi yerdir. 
- `kube-node-lease:` Node hard disk islemleri icin olusturulan ozel bir namespacedir.
- `default:` Kisacasi kube ile baslayan tum namespaceler kubernetes tarafindan olusturulur ve clusterin isleyisi ile alakali objelerin tutuldugu yerlerdir. Bunlarin disinda default namespace adinda baska bir namespace daha olusturulur ve adindan da anlasilacagi uzere bizler aksini belirtmedigimiz surece objeler burada olusturulur.
Bizler herhangi bir namespace tanimi yapmadan baslar ve objelerimizi bir tek namespace altinda toplamayiz. Tek bir ekip tarafindan yonetilen kucuk bir kubernetes clusterimiz var ise boyle de devam ederiz. Fakat ne zaman buyur ve birden fazla ekip tarafindan yonetilen ve birden fazla ekip tarafindan deploy edildigi bir k8s cluster a evrilirsek iste o zaman namespaceler bize kota ayarlamasi ve namespace bazinda kullanici erisimi verebilme gibi ozellikleri sayesinde yardimci olacaktir. 

// belirli bir namespacedeki podlari goster
`kubectl get pods --namespace <namespace ismi>`

// tum namespacelerdeki podlari goster
`kubectl get pods --all-namespaces`

// namespace yarat
`kubectl create namespace <namespace ismi>`

// namespace leri goster
`kubectl get namespaces`

// belirli bir namespacedeki podlari goster
`kubectl get pods -n <namespace ismi>`

// varsayilan namespace i degistir
`kubectl config set-context --current --namespace=development`

// Bir pod baska bir namespacedeki pod a baglanmasi icin
`mysql.connect("<service name>.<namespace>.<service>.<domain>")`
`mysql.connect("db-service.dev.svc.cluster.local")`

# Deployment
En temel obje pod dur ama biz genelde tekil yonetilmeyen podlar yaratmayiz. podlari yoneten ust seviye objeler yaratiriz ve podlar bu objeler tarafindan yaratilir ve yonetilir. 

***Neden tekil podlar yaratmiyoruz?***
Deployment buna nasil cozum bulur?
Bir uygulamamizi production ortaminda deploy ettigimizi varsayalim. Uygulamamizi container image i haline getirdik ve bunu kubernetes de deploy etmek icin bir pod tanimi yarattik kubectl apply ile bunu api sever a gonderdik. 3 worker node lu bir yapimiz var ve kube-scheduler uygun bir worker node secerek podu o node uzerinde olusturdu. Fakat bir sure sonra bu podun olusturuldugu worker node bozuldu ve erisilemiyor. Dolayisiyla pod a da erisemiyoruz. Bu durumda pod terminating durumuna gecer ve oylece kalir ve tabii ki de pod calismadigi icin uygulamamiz da calismamaya baslar. Siz bir pod yarattiginiz zaman kube-scheduler bu poda uygun bir node bulur ve onun uzerinde bu pod u calistirir. Eger container seviyesinde bir sikinti cikarsa ve restart policy onfailure ya da always olarak secildi ise container restart edilir ve sorun cozulur. Ama pod un schedule edildigi node da bir sikinti cikar ya da kaynak sikintisi nedeni ile o pod durdurulursa kube-scheduler devreye girip aaa su pod calismiyormus deyip baska bir node uzerinde onu schedule edeyim demez. Bu nedenle worker node umuz gittigi icin de pod umuz terminate state e duser ve oylece kalir. Buna cozum olarak da 3 ayri podun 3 ayri node uzerinde calismasini node selector tanimlari ile sagladik. Kisacasi her bir worker node da bir tane olmak uzere 3 tane pod yarattik, onlerine de bir load balancer koyduk. Boylece bir tanesine bir sey olursa diger 2 tanesine erisilebilmesini sagladik. Ancak diyelim ki ben bu uygulamayi gelistirmeye devam ettim, yeni bir versiyon cikardim, yeniden bir container image i yarattim. Simdi bu 3 podu bu yeni image la yeniden olusturmak istiyorum. Bunu nasil yapmam gerekir tek tek butun yaml dosyalarindaki image kismini degistirmem gerekiyor. Tekrar kubectl apply yaptim ve 3 podu tekrar olusturum isler boyle zahmetli oluyor. Eskisine donmek istedigimde veya label olusturmak istedigimde isler kontrolden cikiyor.
Deloyment bir veya birden fazla pod u bizim belirledigimiz desire state e gore olusturan ve sonrasinda bu desire state i yani istenilen durumu mevcut durumla surekli karsilastirip gerekli duzeltmeleri yapan bir obje turudur. Bizler bir deployment objesi olusturmak icin tanim yapar ve bu tanim icinde olusturmak istedigimiz pod un hangi ozellliklere sahip olacagini ve kac adet olusturmak istedigimizi belirtiriz. Bu deployment objesi olusturuldugu zaman bu tanim ve adette pod olusturulur. Ornegin nginx image indan 3 tane pod olusturan bir deployment yaratiriz. Deployment bunu desire state olarak alir ve bu 3 pod olusturulur ve deployment controller devreye girer. Ornegin bu podlardan birini sildim. Deployment kontroller desire state ile current state i karsilastirir ve sonrasinda esitler. Bunun yaninda deployment objeleri bize kurallar sayesinde pod larda guncelleme yapmamiza imkan tanir. Onceki ornektei gibi uygulamanin yeni versiyonunu yazip bundan yeni bir image olusturmustuk. Ama bunu deploy etmek istedigimiz zaman tek tek pod taminlarinda bu isleri manuel yapmistik. Deployment da ise sadece desire state tanimimizla image kismini guncelleyerek bu isi halledebiliriz. Deployment yeni desired state tanimini alir ve tek tek buna gore podlari olusturmaya baslar. Hatta bunu kontrollu bir sekilde yapmasi icin ek parametreler belirleme sansini da verir. Yani yeni image bu podlari guncelle ama birden butun podlari silip yenilerini olusturmaya calisma. Bir tane sil sonra yenisini olustur 30 sn bekle ikincisini sil seklinde rollout u kontrollu bir sekilde yapmamizi saglayabilir. Boylelikle uygulamanin yeni versiyonu deploy ederken kesintisiz gecis imkani saglar. Bunun yaninda sikinti cikan durumlarda eskiye donmeyi de kolay bir sekilde yapabilmemize imkan saglar. Yani gunceller ama bir seylerin yanlis gittigini de gorursek eski haline dondurmeyi tek bir komutla halledebiliriz. Bizler aslinda k8s uzerinde her ne kadar pod olustursak da bu podlari yalnizca pod olarak olusturmayiz. Olusturulmak istenen podlari daha ust seviyede objeler ile olustururuz ve bu sayede uzun vadede yapacagimiz islemleri otomatize etmis oluruz. Deployment bunlarin icinde en sik kullanilandir hatta is yuklerimizin tamami deployment objeleri halinde deploy edilir. 
Best practice olarak yek bir pod bile yaratacak olsaniz deployment ile yaratmak gerekir.

// Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

// Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

// In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.
`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

// deployment yarat
`kubectl create deployment firstdeployment --image=nginx:latest --replicas=2`

// deployment guncelle
`kubectl set image deployment/firstdeployment nginx=httpd`

// deployment delete
`kubectl delete deployment firstdeployment`

Deployment yaml dosyalarinda template kisimlari deployment in ozelliklerinin yazildigi alan. Bu kisim olusturulmasi istenen pod un metedata kismi ile ayni o kisimdan yalnizca name kismini cikarip yapistiriyoruz. 
Selector kismi ise deployment objesininn yakalayacagi (sahip oldugu) podlarin labellerini deployment a bildirmek icin kullanilir. Template kisminda labels in altinda bunu tanimliyorum ki secme isi tamamlansin. 

# Replicaset
Replication Controller replicaset in eski bir teknolojisi ve artik kulanilmiyor.
Bir replicasetin amaci herhangi bir zamanda calisan kararli bir replica pod setini surdurmektir. Bu nedenle, genellikle belirli sayida ozdes pod un kullanilabilirligini garanti etmek icin kullanilir. Deployment bizim istedigimiz ozelliklerde podu kendi olusturmaz. Replicaset bizim istedigimiz ozelliklerde replicaset objesi olusturur ve podlar bu replicaset objesi tarafindan olsturulur. 
K8s ilk ciktiginda replication-controller adinda bir objesi vardi halen var ancak kullanilmiyor. Replication controller birden fazla ayni tipte pod olusturmak icin kullaniliyordu fakat deploy ettigi podlarla ilgili degisiklik yapmak istedigimiz zaman bazi sikintilar cikariyorudu. Bu sikintilari cozmek icin de soyle bir yola gidildi. Bu replication controller in sagladigi ozellikler deployment ve replicaset adinda 2 objeye bolundu. Replicaset objesinin temel gorevi su oldu: belirledigimiz ozelliklere gore belirledigimiz sayida pod olusturmak ve bunun desired state de kalmasini saglamak. Deployment ise bunun bir ust seviye objesi olarak dizayn edildi ve pod taniminda bir guncelleme yaparsak bu guncellemenin belirledigimiz kurallara ve sirayla uygulanmasini saglamak oldu. Ozetle biz bir deployment objesi olusturdugumuz zaman bu kendi yonettigi bir replicaset objesi olusturur ve bu replicaset objesi de podlari yaratir ve yonetir. Bir deployment taniminda bir degisiklik yaparsak ornegin kullanilan image i guncellersek deployment bu yeni tanimla yeni bir replicaset objesi daha yaratir. Ilk yaratilan replicaset objesi yavas yavas kendi olusturdugu podlari silmeye baslar ve yeni replicaset de yeni podlari yaratir. Burada silme ve yaratma isleminin neye gore olacagini bizler belirleyebiliriz. Bu bize herhangi bir kesinti olmadan uygulama guncelleme ve yeni versiyon gecisi yapma imkani saglar.
Replicaset ile deployment yaml ayni arasindaki tek farf kind kisminda birisinde replicaset yazarken digerinde deployment yazar. Geri kalan tum satirlar ayni kalmaktadir. Replicaset yerine deployment olusturmak rollout ve rollback yapmamizi sagliyor. Bunu replicasetler ile yapamiyoruz. Replicaset ler ile bir kere olusturulan pod un imagei bir daha degistirilemiyor.
Replicaset ile deployment yaml dosyalari arasinda bir fark yok sadece isim kismi deployment yerine replicaset yazmak yeterli.

`kubectl create -f replicaset-definition.yaml`
`kubectl get replicaset`
`kubectl delete replicaset myapp-replicaset`
`kubectl replace -f replicaset-definition.yaml`
`kubectl scale -replicas=6 -f replicaset-definition.yaml` 

# Rollout and Rollback
Deployment yaml dosyalarinda spec altinda strategy anahtari ile bizler bu deploymenti guncelledigimiz zaman rollout islemlerinin nasil yapilacagini belirleriz. 
Kullanabilecegimiz 2 tip rollout stratejisi mevcuttur. 
1. *Recreate:* Bu deploymentta bir degisiklik yaparsam oncelikle bu deploymet daki tum podlari sil ve bu islem tamamlandiktan sonra yeni podlari olustur. Bu stratejiyi genelde hardcore migration yaptimizda kullaniriz. Ornegin uygulamamizin yeni versiyonu ile eski versiyonunun kisa bir sure icin bile olsa bir arada calismasi sakincali olacaksa, major degisikliklerde `recreate` stratejisi secilerek oncelikle tum eski verisyonlarin sistemden kaldirilmasi ve sonrasinda yeni podlarin yaratilmasi istenebilir. *Her zaman kullanilan bir secenek degildir.*
2. *Rollingupdate:* Default stratejidir. Herhangi strateji belirtmez iseniz bu strateji uygulanir. Recreate in tam tersidir. Degisikligi asamali olarak yapar. Bu islemin de 2 farkli opsiyonu vardir. Bunlar `maxUnavailable` ve `maxSurge` dir. 
  1. `maxUnavailable`: Ben deploymentda bir degisiklik yaptigim zaman en fazla burada belirledigim sayidaki kadar podu sil. `maxUnavailable: 2` su demek: Yani ornegin 10 tane pod dan olusturulan bir deployment da bir guncelleme yapar isem bu guncellemeye baslandigi anda en fazla 2 tanesini siliyor. Sonra yeni podlari olusturuyor onlar olusturulduktan sonra devam ediyor yani bir nevi gecis sirasinda en fazla kac pod un silinebilecegi bilgisini giriyorum. Burada sayi yerine yuzde de girilebilir. 
  2. `maxSurge` Gecis sirasinda toplam pod sayisinin en fazla kac pod olabilecegini belirler. Soyle bir sey dusunun 10 podlu bir deployment i guncelledigim zaman ne olacak kubernetes sirayla 10 tane eski podu silecek ve 10 tane de yeni pod ayaga kaldiracak ve bunu asama asama yapacak yani ayni anda sistemde hem yeni hem de eski podlar olacak. Iste bu durumda toplamda sistemde en fazla kac pod olabilir sayisini `maxSurge` ile belirliyoruz. Bu ornekte 2 secilmis bu su demek oluyor bizim desired state imiz 10 pod ama gecis sirasinda 12 poda kadar calisabilir. Yani olacak olan su ben bu deployment i olusturdum, sonra update ettim, k8s oncelikle yeni bir `replicaset` olusturacak ve yeni tanimda 2 pod ayaga kalkacak. Dolayisiyla eski 10 pod + 2 yeni pod toplam 12 pod ayakta olacak. Sonrasinda eski replicaset 2 tane podu silecek. Toplam pod sayisi tekrar 10 a dusecek. Yeni replicaset yeni 2 pod olusturacak toplam sayi 12 ye cikacak sonra eski replicaseti tekrar silecek sonra yeni replicaset tekrar olusturacak tek tek toplam sayi 12 yi gecmeden ve 8 in altina da dusmeden islemler devam edecek ve sonunda eski replicaset 0 a yeni replicaset de 10 a cikacak bu sayede sistemde hicbir kesinti yapmadan gecis yapmis olacagiz.
`maxUnavailable` ve `maxSurge` un default degeri yuzde 25 dir.

// record komutu ile deploymentdaki yapilan islemleri daha sonra rollout yapabilmek icin kaydediyor
`kubectl set image deployment <deployment ismi> <yapilacak degisiklik> --record=true`

// rollout ile Deployment yapılan değişikliklerin listelenmesi.
`kubectl rollout history deployment <deployment ismi>`

// belirli bir degiskligin detayli listelenmesi
`kubectl rollout history deployment <deployment ismi> --revision=<sayi>`

// istenilen bir revizyona rollout yapmak
`kubetcl rollout undo deployment <deployment ismi> --to-revision=<historydeki revizyon sayisi>`

*`Rollback`* ise deployment da yapilan degisiklerin geri alinmasidir.
Rollout komutunda 3 opsiyon mevcuttur:
  1. *Rollout status:* Bu opsiyonu ile deployment imizda yapilan degisiklikleri canli olarak izleme imkani elde ederiz. 
  2. *Rollout pause:* Bir rollout un ortasinda bu komut ile mevcut komut durduruluyor.
  3. *Rollout resume:* Durdurmus oldugum deployment i yeniden devam ettirmek istersem resume komutu ile devam ettiriyorum. 

//Deployment yapılan değişikliklerin izlenmesi.
`kubectl rollout status deployment <deployment_ismi>` | `kubectl rollout status deployment rolldeployment` 

// Deployment üstünde yapılan değişikliklerin durdurulması.
`kubectl rollout pause deployment <deployment_ismi>`
| `kubectl rollout pause deployment rolldeployment`

// Durdurulan rollout'un devam ettirilmesi. 
`kubectl rollout resume deployment <deployment_ismi>` | `kubectl rollout resume deployment rolldeployment`

# Network
Network temel kurallari
- Kubernetes kurulumunda podlara ip dagitilmasi icin bir ip adres araligi ya da kubernetes terminolojisinda bilinen adiyla `--pod-network-cidr` belirlenir.
- Kubernetes'de her pod bu cidr blogundan atanacak bir unique ip adresine sahip olur.
- Ayni cluster icerisindeki tum podlar varsayilan olarak birbirleriyle herhangi bir kisitlama olmadan ve `NAT` yani network address translation olmadan haberlesebilirler.

Diyelim ki elimde 4 tane sunucum var ve bunlardan kubernetes cluster olusturmak istiyorum. Evdeki internet baglantimi saglayan bir routerim var ve bu routerin da ic networku 192.168.1.0/24 networku. Router in ic bacagi da 192.168.1.1 ip adresine sahip. Buraya bir switch bagladim ve router ve makinalari da bu switch e bagladim. Makinalara 192.168.1 networkundan ip adresi atadim ve hepsinin default gateway leri router i goruyor. Bu makinalar hepsi ayni network uzerinde ve dolayisiyla birbirleriyle sikintisiz bir sekilde haberlesebiliyorlar. Default gateway olarak da router olarak ayarli oldugu icin de dis dunyaya gidecek tum trafigi bu router a yonlendiriyorlar ve onun ustunden gitmek istedikleri yere gidebiliyorlar. Bu makinalarin hepsine linux isletim sistemlerini de yukledim. Simdi sirada k8s kurulumu var. Tum makinalarda k8s kurulumu icin gerekli on hazirliklari yaptim ve master node a gecerek k8s kurulumunu baslatacagim. Oncelikle podlar icin bir ip adres araligi belirlemem gerekiyor yani --pod-network-cidr. Ben de bu network disindan bir ip adres araligi belirledim gerekli ve 10.10.0.0/16 networkunu sectim. k8s kurulumuna --pod-network-cidr=10.10.0.0/16 opsiyonunu girerek basladim, master node ayaga kalkti. Sonra gittim tek tek worker node lari da cluster a dahil ettim ve k8s kurulumum tamamlandi. Bu kurulumu tamamladigimizda worker node larda su islemler olur: ilk olarak her bir worker node uzerinde bir adet virtual bridge yaratilir. Bu virtual bridge bize baslangicta belirledigimiz pod-cidr araliginda iletisim kuracak seklinde ayarlanir. Sonrasinda yeni bir pod yarattigimiz zaman o pod icin linux kernelinda ayri bir network namespace olusturulur ve bu pod un sanal ethernet sifir interface ini bu pod cidr adres blogundan bir ip adresi atanarak bulundugu host ustundeki bridge e balganir. 6 adet pod yarattim ve bu podlarin hepsi cidr blogundan ip adresi almis durumda. 

Bu asamada network trafiginin nasil isledigini dusunelim bununla ilgili 3 ayri senorya yapilacak:
1. Senaryoda podlar dis dunyadaki bir adrese gitmek istediginde network akisi nasil oluyor bunu pod a uzerinden inceleyecegiz. Diyelim ki bu senaryodaki pod-a dis yunyadaki 1.1.1.1 ip adresli bir sunucu  ile iletisim kurmak istedi. Bu durumda pod-a iletisim paketlerini bridge e teslim edecek paket burdan worker node un ethernet sifir internet face ine ulasacak paket burada nat lanacak sonrasinda paket worker node un default gateway i olan rooter a gidecek. Rooter da yeniden nat lacak oradan da 1.1.1.1 ip adresli sunucuya ulasacak. Bu sunucudan gelecek paket de yeniden natlacak ve pod-a ya ulasacak. Dolayisiyla pod-a uzerinde calistigi worker node un ulasabildigi her destination a ulasabilecek. 
2. Senaryoda yine pod-a yi ele alalim ve bu sefer pod-a nin ayni worker node da bulunan pod-b ile iletisim kurmak istedigini varsayalim. Gordugunuz gibi her iki pod da ayni worker node uzerinde bulunan ayni bridge e bagli ve ayni ip adres blogundan adrese sahipler. Dolayisiyla pod-a paketi bridge e teslim edecek bridge den de pod-b ye gidecek. pod-b nin cevabi da bu yolun aynisi olacak. Ayni network deki 2 makina gibi sorunsuz ve natsiz birbirleriyle haberlesebilecekler. 
3. Bu senaryoda pod-a baska bir worker node da deploy edilmis bir pod ile iletisim kurmak istiyor. Ornegin worker2 node uzerindeki pod-c ile. Kurallarda ne demistik podlar birbirleriyle nat (network adress translation) a ihtiyac duymadan birbirleriyle haberlesebilmeli. Peki kural buysa pod-a pod-c ile nat olmadan nasil haberlesecek? Pod-a pod-c ye ulasmak istiyor paketi bridge e teslim etti ancak bridge de pod-c nin ip adresi tanimli degil. Mecburen trafigi ethernet sifir a gonderecek. Ethernet sifir bakacak destination 10.10.20.2 bilmedigi bir adres. Kendi ip adresi blogu disinda, routing tanimli degil. Napacak mecburen natlayip router a teslim edecek. Orada da paket kaybi olacak. Cunku router da paketi nereye gonderecegi ile ilgili bir bilgi yok. Bu senaryoda ayri worker nodelarda kosan podlar birbirleriyle haberlesemiyorlar. Bunu cozecek birkac teknoloji var. Mesele vxlan teknolojisini kullanarak 10.10.0.0/16 network unu node larin bagli oldugu 192.168.1.0 network u uzerinde kosacak bir overlay network u olarak tanimlayarak nat a ihtiyac duymadan trafik akisi saglayabiliriz. Ya da podlara 198.162.1.0 networkundan ip verecek sekilde tanim yapip worker node un ethernet sifirlarina bu ip adreslerini de tanimlayip trafigi nat siz route edebiliriz. Bu ve benzeri bizim network altyapimiza uygun olacak bir cozum bulabiliriz. Ancak tum bu cozumleri k8s altinda saglamak neredeyse imkansiz. Bu nedenle k8s kendi icinde bir network cozumu ile gelmez bunun yerine container run timelar veya kubernetes gibi orchestratorlar ortaya ciktiklarinda hepsi bu ve benzeri network sorunlarini adreslemek icin cesitli yontemler denediler ve uyguladilar fakat bir sure sonra bu ayri ayri yontemler belirli bir sure sonra karmasa yaratti. Bu karmasayi cozmek adina Cloud Native Computing Foundation (CNCF) tarafindan tum bu linux container alt yapisinda ip dagitimi bridge tanimlari vxlan tanimlari ve diger ayarlarin standartlarini belirleyebilecek container Container Netork Interface (CNI) denilen bir proje baslatildi. CNI bu container network altyapisinin nasil olmasi gerektigi ile ilgili standartlar yayinladi ve developerlarin bu standarda uygun networking cozumleri olusturabilmesine imkan sagladi. Bu sayede bu standartlara uygun bir sekilde network plugin leri yazilma imkani dogdu. K8s de bu projenin bir tarafi olarak sunu dedi bakin ben k8s de bu pod networkunu kendim cozmeyecegim CNI standardini benimsedim. Siz kubernetes clusterinizda network islerinizi bu standarda uygun olarak yazilmis ve sizin network cozumunuze uygun olacak bir plugin secin. Yani k8s  bu isleri kendi uzerinden atti ve sizin k8s i deploy ettiginiz ortama uygun CNI network plugini secerek tum bu altyapiyi bunun tanimlamasina izin verdi.
  
K8s kurulumunda ortamimiza uygun bir CNI plugin i secmemiz sarttir. Kullanabilecegimiz pek cok open source plugin vardir ve hepsinin degisik ozellikleri vardir. Mesela Azure uzerinden yonetilen Aks kullansa idik Azure kendi yazdigi aks plugin ini kullanarak her pod un vnetten direk bir ip adresi almasini saglar ve bu iletisimi bu sekilde cozer. Ya da cisco router lardan olusan bir network altyapisi kullaniyor olsa idik cisco nun aci plugin ini kullanip bunun bu isi halletmesini saglayabilirdik. Tum bu cni plugin dunyasi ortama uygun olarak secilmelidir. Bunlarin 2 tanesi on plana cikti. Manuel kurulan k8s clusterlarda genelde bu iki plugin kuruluyor. Birincisi `flaner` ve network policy gibi destek sunmasi ile de one cikan `calico` dur. k8s kurulumunu tamamladiktan sonra cni plugin ini de yukleyerek k8s in network yonetimini bu plug ine devretmesini saglayabiliyoruz. Ornegin biz bu ortamda `calico` cni plugin yuklersek podlarin ip adresinin atanmasi, ip table kurallarinin duzenlenmesi ve node lar arasinda bu cidr blogunun nasil calismasi icin overlay tanimlarinin yapilarak vxlan interfacelarinin olusturulmasi dahil tum gorevler calico tarafindan hallediliyor. k8s de bir pod olusturdugunuz zaman scheduler bunun calisacagi node u secer ve kubelet burada devreye girerek podu olusturur. Pod icin atanacak ip adresinin belirlenmesi, bunun poda atanmasi ve altyapida ayarlanmasi gereken tum islemlerin halledilmesi ise calico tarafindan saglanir. Boylece pod lar farkli node larda olsalar bile birbirleriyle nat olmadan haberlesebilirler. k8s de podlarin birbirleri ile ve dis dunya ile haberlesmeleri yani ingress trafigi bu sekilde gerceklesiyor.


// ip link eth0 in mac adresini goster
`ip link show <eth0>`

// ip adresinden node cni yi tespit et
`ip a | grep -i <10.59.250.12>`

// ip range show
`ip addr show <weave>`

// node mac adres
`arp <node01>`

// ping google from the master node
`ip route show default`

// network plugin path
`/opt/cni/bin`

// network ip range
`apt update`
`apt install ipcalc`
`ip a | grep eth0`

// ip range for pod
`kubectl logs <weave-pod-name> weave -n kube-system`

// IP Range configured for the services within the cluster
`cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range`

# Service
Bir dizi pod uzerinde calisan bir uygulamayi ag hizmeti olarak gostermenin soyut bir yolu.
Bir web sitesi olusturacagimizi hayal edelim. Insanlar bu web sitesine baglanacak ve resim yukleyecek. Yukledikleri resimlere bir filtre ekleyecegiz ve geri dondurecegiz. Biz bu uygulamayi 2 ayri servis olarak tasarladik. Birinci servis frontend adinda olacak ve kullanicilar buraya baglanacak. Bu kismi javascript ile yazdik. 2. kisim ise bu frontend in baglanarak resimleri gonderdigi ve bu resimlere filtre ekleyen yazilim olacak. Bunu da .net de yazdik. Her iki kismi da container image lari haline getirdik, artik k8s uzerinde bunu deploy edebiliriz. 2 tane deployment yaml dosyasi olusturduk ve bu frontend ve backend katmanlarini 3'er pod olacak seklinde k8s uzerinde deploy ettik, senaryomuz bu. Simdi bu senaryonun erisim kismini inceleyelim. Ilk olarak bu 3 frontend poduna benim bir sekilde dis dunyadan baglanilmasini saglamam gerekiyor. Cozmemiz gereken ilk sorun bu. Benim bu podlara dis dunyadan bir sekilde erisim saglamam lazim. Bu sorunu  bir sekilde cozduk diyelim. Kullanicilar bu podlardan birine baglandi ve resmi yukledi. Benim bu frontend uygulamamin resmi islemesi icin backend podlardan birisi ile iletisim kurmasi ve resmi ona gondermesi gerekiyor. Bu kisim sanki daha kolay gibi. Sonucta k8s uzerinden her podun essiz bir ip adresi var ve her pod birbiri ile sikintisiz haberlesebiliyor. Fakat simdi soyle bir sikinti var. 3 frontend pod u deploy edecek sekilde bir deployment objesi olusturdum ve o da 3 pod olusturdu. Ayni sekilde 3 backend podu olusturacak deployment objesi de olusturdum. Benim frontend podlarim backend podlarim ile network uzerinden iletisim kurmalari gerekiyor. Frontend e bir resim yuklendigi zaman bu frontend pod herhangi bir backend pod ile iletisim kurarak ona resmi yollayacak o da isleyip geri yollayacak. Bunun olmasi icin frontend lerin backend podlara nasil ulasabilecegini bilmeleri gerekir. Kisacasi onlarin ip adreslerini bilmeleri lazim. Ben bu sorunu cozmek adina oncelikle tum backend pod larinin ip adreslerini listeledim sonra frontend podlarima tek tek baglandim ve iletisim kumalari gereken ip adreslerini yazilima ekledim ve bu sayade frontendler backendlere erisebildi sorunu simdilik cozdum. Ama simdi soyle bir sikinti var. Bildiginiz gibi k8s cesitli durumlarda bu podlari bu sistemlerden kaldirip yeniden olusturabilir. Ornegin node lardan biri bir sikinti cikardi ve node devre disi kaldi. Backend deployment objesi bu durumu tespit etti ve devre disi kalan node lar uzerinde kalan podlari saglikli node lar uzerinde yeniden olusturdu. Yeniden olusan podlar da yeni ip adresleri aldi. Simdi benim tekrardan frontend lere baglanarak yazilim icerisinde bu ip adreslerini guncellemem gerekecek ve isin kotu tarafi su ki bu durum k8s uzerinde surekli olabilir. Surekli pod silinerek yeniden yaratilabilir. Ya da diyelim ben bu backend deploymenti scale ettim. Artik 4 pod calisiyor bunu da eklemem gerekebilir. Goruldugu gibi benim bu senaryoda cok islem yapmam gerekiyor. Neyse ki bu mekanizmayi cozebilecek bir mekanizma k8s de mevcut.

K8s servisler bu ve benzeri sikintilari ortadan kaldirmaya yarayan objelerdir. 4 tip servis objesi yaratabiliriz. 
1. `Cluster IP:` Bizler bir cluster ip tipi servis objesi yaratip, bunu label selector ler araciligiyla podlarla iliskilendirebiliriz. Bu objeyi yarattigimiz zaman bu obje cluster icerisindeki tum podlarin ve kaynaklarin cozebilecegi unique bir dns hizmetine sahip olur. Bunun yaninda bizler her k8s hizmeti kurulumunda servis cluster range ip anahtari ile bir sanal ip blogu belirleriz. Ornegin 10.10.0.0/16. Siz bir cluster ip servis objesi yarattiginiz zaman bu objeye bu ip blogundan bir sanal ip adresi atanir ve ismi ile bu ip adresi cluster in dns mekanizmasinda kaydedilir. Bu ip adresi virtual bir ip adresidir herhangi bir interface e map edilmez. Kube-proxy servisi tarafindan tum nodelardaki ip table lara eklenir ve buraya ulasan trafik servisin iliskilendirdigi podlara randomic bir siralama ile yonlendirilir. Yani cluster icerisinden bu ip adresine gonderilen paketler bu servisle iliskilendirilmis podlardan bir tanesine yonlendirilir. Bu bizim iki sikintimizi cozer. Az onceki ornekteki gibi ben bu backend podlarla iliski kuracak bir backend isimli servis yaratir ve frontend lere de backend podlara ulasmak istediginde goturmek gereken adres backend isimli adrestir dersem su olur. Cluster icerisindeki herhangi bir pod backend ismini cozmek istediginde ona cevap olarak bu backend servisinin ip adresi doner. Pod o ip adresine ulasmak istediginde de trafik backend podlarindan bir tanesine yonlendirilir dolayisiyla ben tek tek frontendlere baglanarak her seferinde yeni eklenen podlarin ip adresi bu, eski podlarin ip adresleri bu diye guncelleme yapmam gerekmez. Ona sadece gitmen gereken yer backend derim o trafigi bu ip adresine yollar. Gerisini cluster halleder. Yeni eklenen podlar otomatik olarak bu servise eklenir. Sistemden cikarilan podlar da otomatik olarak bu servisten cikarilir. Yani cluster ip servisleri k8s altinda basit de olsa service discovery ve load balancing hizmetleri kuran obje tipidir. Simdi cluster ip sayesinde forntend backend baglanti sorununu cozduk. Olusturulan servise atanan bir virtual ip adresidir ve yalnizca cluster icinde gecerlidir. Cluster icinde herhangi bir kaynak bu ip adresinin 5000 portuna istek gonderdigi anda bu istek bu servisin selector ile sectigi podlardan birinin ip adresine ayni port uzerinden yonlendirilecek. Ayni cluster icindeki podlar birbirleriyle isimleri uzerinden haberlesebilirler ancak namespace farklilastigi zaman acik ismini belirtmek gerekir. Podun acik ismi de su sekilde yazilmaktadir: `isim.namespace_ismi.svc.cluster_domain`. bir poddan digerine istek yapildiginda her seferinde olmasa da bu yanit farki podlardan gelir cunku load balancing yapilmaktadir. Burada round robin algoritmasi kullanilarak istekler farkli podlara dagitilmaktadir.

2. `Nodeport:` Ilk sorunumuz neydi. Podlarimiz web sitemizi barindiriyor dolayisiyla bunlarin cluster disindan erisilebiliyor olmasin lazim. Bu sorun da nodeport tipi servis objeleri ile cozeriz. Aynen cluster ip gibi node port da bir servis obje tipidir. Siz bir nodeport servisi yaratip bunu label selectorler araciligiyla podlarla eslestirebilirsiniz. Bu obje yaratildigi zaman su olur 30000-32767 araliginda bir port secilir ve tum worker node larda bu port sizin servisinizde proxy lenir. Kisacasi worker node un ip adresininin bu portuna gelen tum paketler iliskilendirdiginiz podlarin publish ettiginiz podlarina yonlendirilir. Sizler de bu worker node larin onune bir worker load balancer ya da reverse proxy sunucu koyar ve dis dunyadan erisilmesini saglarsaniz buraya ulasacak paketler worker node lara, worker node lara ulasan paketler de podlarin birine ulasarak kullanicilarinizin uygulamaniza erismesini saglarsiniz. 
Olusturdugum nodeport service cluster IP den cok farkli degil. Cluster icindeki haberlesmeye imkan taniyor ancak bunun yaninda ek bir sey daha yapiyor: tum worker node larin dis bacaginda 30447 portunu dinlemeye basliyor. Biz herhangi bir worker node un bu portuna ulasir isek trafik frontend portlarindan birine yonlenecek. 

3. `Lod balancer:` Bu tip objeleri sadece azure k8s servis gibi ya da google k8s engine gibi cluster saglayicilarin manage k8s servislerinde kullanabiliriz. Bu obje olusturuldugunda cloud servis saglayici dis dunyadan erisilebilecek public bir ip adresine sahip bir load balancer olusturur. Sonrasinda bu load balancer i sizin service objenizle iliskilendirir. Dolayisiyla bu public adresine ulasan paketler iceriden podlara yonlendirilir. Nodeport icin dedim ya worker node larin onune bir load balancer ya da reverse proxy koyar oradan kullanicilara eristirirsiniz. Iste cloud servis saglayicilar bunu otomatik olarak yapiyorlar. Buna da load balancer tipi servis objesi diyoruz. 
Service dosyasi olustururken name kismi onemli cunku bu isim artik clusterdan bizim ulasacagimiz kisim olacak. Service olustururkenki yaml dosyalarinda spec kisminin altindaki `selector` arkasina alacagi yani yakalayacgi podlarin labelleri yazilmali. Ports tanimlamasinda ise `port` bu servisin dinleyecegi port `targetport` ise podlarin expose ettigi port olacak.

Bu servisi localde kurmak bir ise yaramayacaktir. Bu servis gcp, eks, aks gibi cloud saglayicilarin k8s ortamlarinda ise yarayip sonuc veren bir uygulamadir.

Kubernetes icerisinde `core dns` tabanli bir `dns` hizmeti bulunur ve tum podlar sorgulari buraya gonderir. Bir servis olusturdugunuz zaman onun isim ve IP adresi bu DNS de kaydedilir ve dolayisiyla tum podlar bu isme ait IP adresini ogrenebilirler. Biz backend yazarak cozduk ancak bu servisin adi aslinda `backend.default.svc.cluster.local` yani `isim.namespace_ismi.svc.cluster_domain` bu ayni namespace icinde calisan uygulamalar icin sorun degil baska bir namespacedeki uygulamalar icin tam isminin yazilmasi gerekecekti.

Deployment konusunda gectigi uzere deployment lar replicasetleri replicasetler de podlari olusturur. Benzer bir durum service icin de gecerli biz bir service olusturdugumuz zaman bu service `endpoint` adinda bir obje olusturur ve `endpoint` objesi bizim belirledigimiz selector tarafindan secilen podlarin ip adresini uzerinde barindirir service trafigini nereye yonlendirecegini bu listeye gore duzenler. Yaratilan service arka planda sectigi podlarin ip lerini adresler yani describe komutu ile bu ip adreslerini liste halinde gorebiliriz. Bu listeyi dimanik olarak tutmaktadir. Bir pod terminate oldugunda veya yeni bir pod yaratildiginda bu liste otomatik olarak guncelleniyor. Ekleniyor ya da siliniyor. 

// Bir deployment'in expose edilmesi "imperative olarak service objesinin oluşturulması"
`kubectl expose deployment "deployment_ismi" --type="service_tipi" --name="servis_ismi"` | `kubectl expose deployment backend --type=ClusterIP --name=backend`

```yaml
spec:
  type: NodePort
  ports:
   - targetPort: 80 # pod un portu
     port: 80 # service portu
     NodePort: 30008 # nodeport un port
  selector:
     app: myapp
     type: front-end
```

Deployment olustururken bu obje bir replicaset olusturuyordu ve bu replicasetler podlari olusturuyordu. Burada da servis objesi olusturulurken servis objesi endpoint adinda bir obje olusturur ve bu endpoint objesi bizim belirledigimiz selector tarafindan secilen podlarin IP adreslerini uzerinde barindirir ve servis trafigini nereye yonlendirecegini bu listeye gore duzenler. Podlar silinip yeniden baslatildiginda bu liste otomatik olarak guncelleniyor.

# Network Policies
Herhangir network policy yaratmadi iseniz
1. Her pod diger her pod ile konusabilir
2. Her pod diger dis dunya ile konusabilir.
3. Herkes disardan ingress ile ya da loadbalancer ile bu podlara erisebilir.

Bu davranisi kisitlamak icin `network policy` lere ihtiyac duyariz
Biz bir network policy yaratiriz ve daha sonra bu network policy nin hangi podlar uzerinde etkili olacagini pod selector ile seceriz. Yani ilk once tanimlamamiz gereken `podSelector` tanimidir.
`policyTypes` da tipini belirliyoruz. `Ingress` iceriye dogru demek yani selector ile secilen podlara gelen trafikle ilgili ayarlar. `Egress` ise secilen podlardan dis dunyaya giden trafigin ayarlarini gosterir. `from` nerenin ingresinden yapailacak olan islemleri gosterir. `ipBlock` hangi ip bloklarindan trafige izin verilmesi gerektigini belirten alana verilen ad. Yani secilen podlara belirlenen ip adresi uzerinden gelecek isteklere `ports` da belirtilen port uzerinden izin ver. `expect` ise haric tutulan `io` blogunu belirtmek icin kullanilir. `namespaceSelector` bu secilen podlara bu namespacedeki labeslerini belirttigim tum podlar asagidaki port numarasindan erisebilsin. `podselector` ile direk pod secerek o podlara network policy eklenebilir. 

`egress` de ise dis dunyaya erirsim izinleri tanimlaniyor.
Ayni poda farkli network policyler atanabilir. Bu durumda kural su: bu policylerdeki kurallarin birlesimi seklinde asign edilir.

# Livenessprobe
Kubelet, bir containerin ne zaman yeniden baslatilacagi bilmek icin `livenessprobe` kullanir. Ornegin, liveless probe, bir uygulamanin calistigi ancak ilerleme saglayamadigi bir kilitlenmeyi yakalayabilir. Boyle bir durumda bir container'i yeniden baslatmak, hatalara ragmen uygulamayi daha kullanilabilir hale getirmeye yardimci olabilir.

*`Livenessprobe` ile container icinde bir komut calistirarak ya da bir http endpoint e request gondererek ya da bir porta tcp connection acarak uygulamamizin dogru calisip calismadigini kontrol etmeye calisiyoruz.* Bu sorgulamanin sonucunda bekledigimiz sonucu alirsak container in saglikli oldugunu, eger bekledigimiz sonucu alamazsak containerda sikinti oldugunu tespit edebiliyoruz. 
Bazi durumlarda pod icinde uygulama calisiyor olarak gorunse de islevini yerine getirmiyor olabilir. Ornegin bir web sayfasi yayinlayan bir podumuz var. Httpd temelli bir container calisiyor. Bu baslayinca container icindeki httpd daemon ayaga kalkiyor ve bizim web sitemizi sunmaya basliyor. Fakat bir zaman sonra bu servis takilmaya basliyor. Bir hatadan dolayi sayfalari sunmaya devam edemiyor. Ama httpd uygulamasi yani service ayakta service cokmedi ya da kapanmadi ama esas yapmasi gereken is olan web sitesini yayinlamak, sunma isini yapamiyor. Bu durumda ortamda soyle bir sorun olusuyor. Eger uygulama calismiyor olsa ya da uygulama kapansa kubelet bunu tespit ederek containeri yeniden olusturarak sorunu cozuyor. Fakat uygulama ayakta olmasina ragmen yapmasi gereken isi yapmiyorsa kubelet bunu tespit edemiyor dolayisiyla container i yeniden baslatarak sorunu cozme mekanizmasi calismiyor. Bunun cozumu ise liveness probes dur. 

Livenessprobe da bir endpointe httpGet ile istek gonderiyoruz oradan 200 donerse basarili 400 donerse basarisiz olarak gorunuyor. Eger sizin uygulamanizin bir healthcheck endpointi varsa onu denersiniz yoksa direk localhosta gidebilirsiniz. Ya da saglik kontrolunu yapmanin en saglikli oldugu endpoint neresi ise onu denersiniz. Bunu uygulamaya ozel kurgulamaniz gerekir. 

Livenessprobe yaml dosyasi icerisindeki `initialDelaySeconds` kismi pod olustuktan sonra uygulama hemen ayaga kalkmayabilir on hazirlik yapmasi bir yerlerden dosya cekmesi gerekebilir. Eger livenessprobe hemen baslatilirsa uygulama hazir olmadigindan hata alacak ve uygulama yeniden baslatilacaktir. Bunu onlemek adina `initialDelaySeconds` parametresi ile container basladiktan sonra bekledikten sonra healtcheck e baslamasi gerektigini soyleriz. `periodSeconds` degeri de bu isin kac saniyede bir yapilacagini gosteriyor. Yani container baslatildi delay seconds 3 saniye, 3 saniye bekledi, httpget ile liveness yapmaya basladi; birinci sorgusunu yapti cevap olumlu; 3 sn bekleyecek ve sonra bir sorgu daha iste bu sorgular arasinda kacar saniye bekleyecegi bu `periodSeconds` degeridir.

Eger bizim uygulamamiz http tabanli bir uygulama degilse bir console uygulamasi ise bunun icin bir script yazar ve bu script ile uygulamamiza uygun sorgu ne ise misal bir dosyanin olup olmadigina bakabiliriz. Iste liveness check alinda httpget disinda ikinci kullanabildigimiz prob olan `exec` bize bu imkani verir. `exec` ile shellde komut ya da uygulama calistiririz. Bu uygulama bize 0 kodu donerse saglikli bunun disinda bir kod donerse sikintili oldugunu anlar ve container i restart eder. 

`tcpSocket:` probu ise bazen http ya da `exec` probu birseylerin saglikli olup olmadigina yardimci olmayabilir. Mysql veri tabani container i calistiriyoruz. Mysql process i 3306 portundan erisilebilir. Iste bizler tcp socket ile bu porta istek gonderip eger cevap alirsak uygulamanin duzgun calistigini varsayabiliriz. Bu tarz http get ve exec ile sonuc alamayacagimiz durumlar icin tcp portuna sorgulama yontemini izlemek istersek tcp socket portunu kullanabiliriz.
Uygulamaniza yonelik uygun bir prob tanimi yapmalisiniz.

```yaml
containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server #container in calistiracagi uygulama
    livenessProbe:
      httpGet:
        path: /healthz #endpoint ine httpGet istegi gonderiyorum eger uygulama calisiyorsa http 200 geri donecek yanlisa sorgunun sonucu 400 donecek ve liveness fail edecek
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

# Readiness
Kubelet, bir containerin ne zaman trafigi kabul etmeye hazir oldugunu bilmek icin `readiness` probelari kullanir. Bir pod tum containerlar hazir oldugunda hazir kabul edilir. Readiness probe sayesinde bir pod hazir olana kadar service arkasina eklenmez.

3 frontend podumu olusturacak bir deploymentimiz bunlari da dis dunyadan erisilmesini saglayacak load balancer tipi bir service imiz oldugunu varsayalim. Bunlari yaml dosyalari araciligiyla olusturduk. Bu deploymet da bir guncelleme yapmak istersek diyelim ki web sitemizin yeni bir image ini olusturduk ve deploymentimizi guncelledik ve `rollingupdate` stratejisi secildigi icin de podlar birer birer devreden cikartilip yeni podlar devreye alindi. Ben bu deployment da yeni bir image atadigim zaman k8s hemen bu guncel image ile yeni bir pod olusturacak ve bunu load balancer servisine dahil edecek yani olusturup baslatildigi anda dis dunyaya bunun uzerine trafik akmaya baslayacak. Ya bu benim uygulamam ilk baslatildiginda bir yerlere baglanip bazi dosyalar cekiyor ve sonra web sitesini servis etmeye basliyor ise ya da baska bir nedenden dolayi container baslatilir baslatilmaz bu servis hizmet veremiyorsa arada belirli bir sure gecip bazi islemler yapmasi gerekiyorsa eger durum buysa ben bu podu olusturdugum anda bu podlar load balancer servisinden trafik almaya basladigi icin sikinti olacak. Yani bu pod ayakta, icindeki uygulama calisiyor. Ama daha henuz web sitemi sunmaya hazir degil. Dolayisiyla bu pod henuz load balancer servisine eklenmek icin hazir degil. Peki ben bunun load balancer hizmetinden trafik almaya hazir oldugunu nasil anlayacagim? Readiness ile.

Readiness probe lar aynen liveness probe lar gibi olusturulur. `http`, `exec` ya da `tcpsocket` olarak kurgulanabilir. Siz bir readiness check tanimlarsiniz eger bu readiness check olumlu sonuc donuyorsa pod servise eklenir ve trafik alir. Eger bu readiness check fail ederse de bu pod servisten cikarilir.

Ornege donersek image i guncelledim ve yeni pod olusturuldu pod calismaya basladi ama bu noktada hemen loadbalancer a eklenmedi. Pod icerisinde `readiness check` calismaya basladi, `initial delay` suresi kadar bekledi sonra tanimladigimiz prob ile kontrolunu yapti. Olumlu cevap aldigi anda bu pod servise eklenir ve trafik akmaya baslar. `Readiness check` period seconda belirlenen saniye kadar surede bir bu islemi yapmaya devam eder. `Fail` edene kadar bu pod servise eklenir. Eger `fail` ederse de pod servisin endpoint listesinden cikarilir. Readiness chek budur. Pod un servis sunmaya hazir halde oldugunu belirtir. Pod olusturuldu ama pod da readiness tanimi var. Dolayisiyla henuz servise eklenmedi. Initial delay second kadar bekledi ardindan ilk kontrolu yapti. Olumlu cevap aldi ve servise eklenerek trafik almaya basladi. Tam bu noktada eski pod terminate edilmeye baslanir. Bu pod terminate edildiginde dahi halen bu poda trafik gelebilir. Kullanicilar buna baglanmaya devam ederler ve tam islem yaparken pod kapanmaya baslamis olursa bu kullanicilara hata donmek zorunda kaliriz. Bunu engellemek adina bir pod terminate edilecegi zaman bu surede poda yeni trafik gelmemesi icin pod servis arkasindan alinir ve servis ile baglantisi kesilir boylece yeni trafik akmaz fakat k8s yine de bu podu hemen kapatmaz cunku yeni trafik gelmese bile halen bu pod hala eski istekleri islemeye devam ediyor ve bu islemlerin bir sure bitmesinin beklenmesi istenebilir.

Pod tanimlarinda `terminationGracePeriodSeconds` adinda bir anahtar bulunur ve varsayilan degeri `30sn` dir. Siz ya da kubelet bir podu terminate etmek istediginiz zaman bu podun icinde calisan `pid1` process e `skill` yani yaptigin ne varsa hemen birak ve kapan komutu gondermez bunun yerine `sitterm` yani su anda yaptigin islem varsa onlari duzgun sekilde tamamla ve bitirince de kendini kapat sinyali gosterir. Process bu sinyali aldiginda eger ortasinda oldugu bir is varsa onu yapmayi birakmaz. O islemleri tamamlar ve bitince tum baglantilari duzgun sekilde kapatir ve sonrasinda process de kapatilir. Iste `terminattionGracePeriodSeconds` degeri container a duzgun bir sekilde kendini kapatabilmesi icin atadigimiz suredir. Eger container bu sure icerisinde duzgun bir sekilde kapanirsa sikinti olmaz fakat kapanmazsa `skill` gonderilir ve zorla kapatilir. Dolayisiyla sizin uygulamanizin ne kadar sure ile duzgun bir sekilde kapanabilecegini bilmeniz ve ondan daha uzun sure surecek bir `terminationGracePeriodSeconds` degeri atamaniz gerekmektedir. Ama 30 sn hemen hemen tum islemler icin yeterlidir. 
Yaml dosyalari icinde `liveness` tanimlari ile `readiness` tanimlamalari ayni seklide yapilmaktadir. Yuzde 99 ikisi ayni anda kullanilmaktadir.

```yaml
containers:
  - name: frontend
    image: ozgurozturknet/k8s:blue
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 20
      periodSeconds: 3
```

# Resource limits
Siz aksini belirtmediginiz surece containerlar uzerinde calistirdiginiz sistemin kaynaklarina sinirsiz bir sekilde erisirler. Eger bir pod tum kaynaklari tuketiyorsa diger podlari etkileyebilir bu nedenle podlarin kullanacagi kaynaklari kisitlamak gerekir. Pod tanimlarinda kullanabildigimiz `request` ve `limit` tanimlari bize bu imkani verir.
- ***cpu:*** cpu kisitlamalari su sekilde uygulanir her uygulamanin belirli cpu kaynaklari vardir. Bu kaynaklar cloud uzerinde ya da sanal olarak calisan sistemlerde `vcp` olarak bare metal sunucularda da `hyperthread` olarak hesaplanir. Kisacasi core saysindan bahsediyorum. Ben bir pod tanimina cpu: "1" tanimi eklersem. O podun calistigi node uzerindeki corelardan sadece bir tanesini kullanabilecegi anlamina gelir. Bunun bir baska yazim sekli daha vardir. Her pod 1000 ms yani 1000 cpu luk bir guc demektir. Ben eger cpu 100m tanimi yaparsam pod calistigi node uzerindeki bir core un 10 da 1 lik gucune erisebilir anlamina gelecektir. Yani cpu 1 ile cpu 1000m ayni keza 0.1 cpu ile 100m de ayni.
- ***memory:*** Clusterda mevcut durumda kullanilabilir bellek varsa, bir container belirlenen bellek istegini asabilir. Ancak bir container bellek sinirindan fazlasini kullanmasina izin verilmez. Bir container, sinirindan daha fazla bellek ayirirsa, container sonlandirma icin aday olur. Container, limitinin otesinde bellek tuketmeye devam ederse, container sonlandirilir. Sonlandirilmis bir container yeniden baslatilabiliyorsa diger herhangi bir runtime hatasi turunde oldugu gibi kubelet onu yeniden baslatir.
Container in kullanacagi `max memory` i byte cinsinden tanimlayabilirsiniz. Ornegin memory 64m derseniz bu container in en fazla 64mbit memory kullanmaniza izin verdiginiz anlamina gelir. `m` megabayt, `g` gigabayt, `k` kilobaytin kisaltilmasi anlamina gelmektedir. Bunlarin yaninda 2 nin katlarini kullanan gosterim sekli olan  kibibayt `KIB`, mebibayt `MIB`, gibibayt `GIB` tanimlari da desteklenir. 

Yaml dosyasi icinde resource kisitlamalarini 2 ayri bolumde tanimlayabiliyoruz. 
1. `Request`: k8s bu podu olusturmaya basladigi zaman scheduler bu podun olusturulacagi bir node sececek bu secme asamasinda tanimlanan degerlerin bulundugu bir node uzerinde schedule et yani bu pod un olusturulabilmesi icin node da minimum ne kadar kaynagin olmasi gerektigini belirtiyor, scheduler bu parametreleri dikkate alarak hangi node da scale edilecegini belirliyor eger burada kaynaklarin saglandigi bir node yoksa bu pod olusturulamayacak. 
2. `Limits`: Bu container in en fazla kullanacagi sistem kaynagi. cpu da belirlenen limitden fazlasina erisilemez ancak memory degerinde tam olarak oyle degil. memory de belirlenen sayi container in kullanabilecegi max ram degeri fakat container bu degere geldigi zaman daha fazlasini kullanamayacak diye bir durum soz konusu degil. `Memory allocation` cpu gibi calismiyor. Bu, sistemden daha fazla ram talebinde bulunabiliyor ve sistem varsayilan olarak bunu engelleme sansina sahip degil. Container daha fazla memory istegine cevap alamadiginda ise bu durumda `OOMKilled` durumuna gelip `restart` edilecek.

```yaml
containers:
  - name: requestlimit
    image: ozgurozturknet/stress
    resources:
      requests: # bu pod un olusturulmasi icin ne kadar bos kaynagin (node) olmasi gerektigini kubescedule a bildirdigimiz alan
        memory: "64M"
        cpu: "250m"
      limits: # container node uzerinde kullanacagi kaynaklarin limitleri
        memory: "256M"
        cpu: "0.5"
```

// Kubernetes pod'lar üstünde cpu ve memory kullanımının kontrol edilmesi.
`kubectl top node` | `kubectl top node "node_ismi"` ile tekil bakılabilir

# Envrionment variable
Ornegin bir web uygulamasi yaziyoruz ve bu web uygulamasi bir veri tabanina baglanarak uygulamalarini burada sakliyor. Bu uygulamanin baglandigi veri tabaninin sunucu adresini ve kullanici adi, sifre bilgilerini de bu uygulamanin icine gomduk ve bu uygulamayi container haline getirdik ve istedigimiz platformda calistirmaya haziriz. Fakat bu senaryoda 2 sikintimiz var. Biz bu veritabani baglanti bilgilerini bu container imagee icine hard-code olarak yazdik yani bir sekilde container image i ele gecerse bi bilgiler expose olabilir. Diger bir sorun ise ben bu image dan container olusturmaya calistigim zaman her defasinda ayni veri tabanina ayni kullanici adi ve sifre ile baglanmaya calisacak. Benim veri tabani adim test ortamimda ayri prod ortamimda ayri olabilir ben kullanici adi bilgilerini zaman icerisinde guncellemis olabilirim. Ya her ortam icin yeni bilgilerle image olusturacagim ya da container olusturduktan sonra bu bilgileri guncelleyecegim. Her seferinde bunu manuel olarak yapmak mumkun degil. Bunun verine environment variable lar tanimlariz.

K8s de disaridan bir poda direk olarak erisim saglayamayiz ya load balancer ya da nodeport araciligiyla olur fakat k8s soyle bir imkan sagliyor. kubetcl araciligiyla poda deployment a service e kendi bilgisayarimdan tunel acarak o objenin portuna trafigimi yonlendirebiliyorum. Iste testlerde hizli bir sekilde objelere baglanmamizi saglayan bu ozellige `portforward` diyoruz.

// Kubectl port-forward pod/envpod 8080:80 komutu ile local hostumu envpod unun 80 portuna yonlendirdim.
`kubectl port-forward <pod ismi> <source:target>` | `kubectl port-forward pod/envpod 8080:80`

```yaml
containers:
  - name: envpod
    image: ozgurozturknet/env:latest
    ports:
    - containerPort: 80
    env:
      - name: USER
        value: "Ozgur"
      - name: database
        value: "testdb.example.com"
```

# Volume

*`Stateless:`* Uzerine yazilan bilginin silinmesi durumunda sorun olmayan uygulama. Mesela onune gelen istege islem yapip gonderen uzerinde veri barindirmayan bir backend uygulamasi. Bu tur uygulamalar container silinip yenisi olustugunda islemeye devam eder. 
Ornegin bir frontend ve bir web sitemiz oldugunu dusunelim. Kullanicilar bu uygulamaya baglaniyor ve bu uygulamada baska bir yerden resimleri cekerek kullanicilara gosteriyor. Uygulama resim bir defa cektikten sonra bunu bir folder icinde sakliyor ve bir daha resmi cekmesine gerek kalmiyor yani bir nevi kendi uzerinde cacheliyor. Bunu k8s de deploy edecegiz. Bir deployment objesi olusturduk ve bu uygulama bir replica olacak sekilde deploy edildi. Podumuz olusturuldu ve calismaya basladi. Buraya erisebilecek bir service objesi de olusturduk. Boylece bu uygulamaya kullanicilar baglanabildi. Hemen backend uygulamamizi ve service i de olusturduk. Kullanicilar web sitemizi kullanmaya basladilar ve diyelim ki cesitli resimleri sectiler. Bizim uygulamamiz da yapmasi gereken islemlere basladi. Diger servise baglandi bu resimleri cekti isledi ve bagli kullanicilara bunu servis etti ama ayni zamanda bu resimleri container icindeki cache isimli folder a kaydetti. Bir baska kullanici daha ayni resmi istedigi zaman tekrar diger uygulamaya baglanmasina gerek kalmadan cache folder dan cekiyor. Diyelim ki bu pod taniminda `livenessprob` da var bir sikinti cikti ve `livenessprob` fail etti. Bu durumda ne oluyordu `kubelet` devreye giriyordu ve containeri restart ediyordu ama aslinda containerlarda restart diye bir kavram yoktur. k8s bize bunu restart olarak gosterse de olay aslinda olan sey su: kubelet calisan containeri siler ve yeniden bir container olusturur. Ancak container silindigi zaman icindeki veriler de siliniyor. Dolayisiyla yeni container icinde cache folderi bos. Cok buyuk bir sikinti degil sonucta bunlar cache dosyalari tekrar diger servise baglanilarak bunu halledebilirim ancak keske bu cache dosyalari yeni container olusturuldugunda silinmese. Keske ben podun icerisindeki tum containerlarin erisebilecegi ve podun yasam suresi boyunca ayakta kalabilecek bir mekanizmaya sahip olsam da dosyalari buraya yazsam. Boyle bir mekanizma olsa bu cache folderini buraya baglarim ve icerisine yazilan tum dosyalar buraya yazilir. Yeni bir container olustugunda da bu dosyalara erismeye devam ederim. Boylece cache den mahrum kalmam. Bu duruma cozum getiren volume ise `ephemeral` yani gecici volume. 

- *`Ephemeral volume:`* Verileri container disinda tutmamiza yarayan volume turleri. 2 tip ephemeral volume vardir
  
  1. `EmpyDir`: `emptyDir` volume ilk olarak bir Pod bir node'a atandiginda olusturulur ve bu Pod o node'unda calistigi surece var olur. Adindan da anlasilacagi gibi emptyDir birimi baslangicta bostur. Pod icindeki tum containerlar, `emptyDir` volumedeki ayni dosyalari okuyabilir ve yazabilir, ancak bu birim her kapsayicida ayni veya farkli yollara mount edilebilir. Bir pod herhangi bir nedenle bir node'dan silindiginde, emptyDir icindeki veriler de kalici olarak silinir. EmptyDir olusturulmasi ile ilgili gerekli tanimlamalari yaptiginiz zaman k8s pod uzerinde bos bir klasor yaratir daha sonra siz bu klasoru konteyner icerisindeki herhangi bir path e mount edebilirsiniz. Ornegin bizim senaryomuzdaki /cache klasorune. Bu containerin bu mount ettiginiz path e yazdiginiz her turlu dosya fiziksel olarak node uzerinde olusturulmus olan bu bos klasore yazilir. Dolayisiyla container silinse bile bu dosyalar silinmez. Yeni container olusturuldugu zaman tekrar bu volume container da ayni path e mount edilir. Bu sayede yeni olusturulmus container da bu dosyalara erismeye devam eder fakat podun yasam suresi sonuna kadar erisilebilir durumdadir. Yani *pod silinirse bu volume de silinir*. Bu nedenle bu volume lere gecici volume ler denir. Podun yasam suresince containerdan bagimsiz olan volumelerdir.
  Ephemeral volumeler ayni podun icerisindeki tum containelerlara ayni anda baglanabilir.
  
  2. `HostPath`: Bir `hostPath` volume, worker node dosya sisteminden Pod'unuza bir dosya veya dizini baglayabilme imkani verir. Bu cogu podun ihtiyac duyacagi bir sey degildir, ancak bazi uygulamalar icin guclu bir kacis kapisi sunar. Temelde emptydir ile mantik olarak aynidir. Yine siz bir volume yaratilmasi icin bir pod taniminizda gerekli ayarlamalari yaparsiniz *fakat bu sefer rastgele bos klasor yaratilmasini soylemek yerine podun olusturulacagi worker node uzerindeki spesifik bir klasoru veya dosyayi belirtirsiniz*. Ornegin worker node uzerindeki /tmp/cache dosyasinin volume olarak kullanmasini soyleyebilirsiniz. Daha sonra container icerisindeki bir path e mounth edebilirsiniz. Bu genelde podlarin calistigi worker node lar uzerinde bulunan spesifik folder ya da dosyalara erismesi gerektigi durumlarda isimize yarar. Misal ozel bir path de konuslanmis bir unique soketine container in baglanmasi gerekirse burayi hostpath ile container a mount edebiliriz.
  3 type `hostPath` volume vardir:
    1. `Directory`: path ini vermis oldugum folder worker node uzerinde zaten var. Zaten var olan bu folder i  volume olarak tanimliyorum. 
    2. `DirectoryOrCache`: Bu folder varsa bunu kullan yoksa bunu olustur.
    3. `FileOrCreate`: Dosya varsa onu kullan yoksa yarat. Ornegin `/cache/config.json` pathini mount edebilirsiniz. 
Volume yaratirken ilk olarak spec altinda volume tanimi yapmak gerekir. Sonrasinda bu olusturdugumuz volume u containerlarda belirledigimiz path lere mount ederiz.

# Secret
Secretlar, parolalar, Auth token ve ssh anahtarlari gibi hassas bilgileri depolamaniza ve yonetmenize olanak tanir. Gizli bilgileri bir `secret` icinde saklamak onu bir pod tanimina veya bir container imajina koymaktan daha guvenli ve esnektir.
  1. env variable lari yaml dosyalarinda tanimladigimizda bu dosyalara erisebilen herkes bu bilgilere de kolaylikla erisebiliyor.
  2. Verilerin degismesi sifrenin guncellenmesi gerektigi zaman yaml dosyasi icerisinde degisikliklere gitmem gerekiyor. Bu hassas bilgileri bu tanimlamalardan ayirmak gerekiyor bunu da secretlar ile saglayabiliyoruz.
Verileri secret icerisinde saklar sonrasinda pod a ekleriz. Secretlar da diger objeler gibi yaratabiliriz.

Secret ile pod ayni namespace icinde olmali.
8 degisik tipte secret yaratabiliriz. `Opaque` bunlardan varsayilan olan turdur. Ama nerdeyse hicbir zaman `Opaque` disinda bir secret tipini kullanmayiz. 

Basit bir secret.yaml dosyasi
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData: #stringdata olarak yazarsaniz bu bilgileri ekranda oldugu gibi yazarsiniz data olarak yazarsaniz bunlarin data64 encoded halini yazmaniz gerekir
  db_server: db.example.com
  db_username: admin
  db_password: P@ssw0rd!
```

Bu secretlari pod a nasil ekleriz? 2 senecek var: 
  1. volume araciligiyla 
  2. env variable araciligiyla ekleriz.

1. Yontem in yaml hali ornek olarak
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secretpodvolume
spec:
  containers:
  - name: secretcontainer
    image: ozgurozturknet/k8s:blue
    volumeMounts:
    - name: secret-vol
      mountPath: /secret # volume u mount ettim boylelikle k8s secret icerisindeki tum anahttarlari birer dosya olarak bu path e koyacak. Bu dosya icerisinde de anahtarla atadigim degerler olacak dolayisiyla ben artik sunu dileybilecegim kullanici adina mi almak istiyorsun /secret altinda db_user_name dosyasina bakarak ogrenebilirsin.
  volumes:
  - name: secret-vol
    secret: # volume u secrettan olusturdum
      secretName: mysecret3
---
apiVersion: v1
kind: Pod
metadata:
  name: secretpodenv #env variable ile secret lari pod a eklemek
spec:
  containers:
  - name: secretcontainer
    image: ozgurozturknet/k8s:blue
    env: # env anahtari ile environment variable tanimladim fakat degerleri yaml dosyasina yazmak yerine secrettan okutuyorum.
      - name: username
        valueFrom:
          secretKeyRef:
            name: mysecret3
            key: db_username
      - name: password
        valueFrom:
          secretKeyRef:
            name: mysecret3
            key: db_password
      - name: server
        valueFrom:
          secretKeyRef:
            name: mysecret3
            key: db_server
---
apiVersion: v1
kind: Pod
metadata:
  name: secretpodenvall #secretlarin icindeki env variable lardan pod a secret aktarmak
spec:
  containers:
  - name: secretcontainer
    image: ozgurozturknet/k8s:blue
    envFrom: # yine env var olarak tanimliyorum ancak bu sefer her birini tek tek tanimlamak yerine git su scripte bak ve oradaki tum env degerlerini ata
    - secretRef:
        name: mysecret3
```

Bu secretlar etcd de `base64` olarak tutuluyor. Kendi yonettigimiz clusterlarda manuel olarak bu secretlari encrypt etmemiz gerekiyor. Cloud saglayicilar bunu bizim yerimize otomatik olarak yapiyor. Kendi yonettigimiz clusterlarda bunu manuel olarak devreye almamiz gerekiyor. Siz bir secret olusturdugunuzda diger objelerde oldugu gibi diger kullanicilar da bu secretlara varsayilan olarak erisim hakkina sahip bunu engellemek icin de `rbac` kullaniyoruz.

// Imperative olarak Secret objelerinin oluşturulması (Opaque de olsa imperative yazarken generic olarak yazmak gerekiyor)
`kubectl create secret generic "secret_ismi" --from-literal="anahtar"="değer" --from-file="anahtar"="değerin_okunacagi_dosya" --from-file="değerin_okunacagi_dosya"` | `kubectl create secret generic mysecret --from-literal=db_server=db.example.com --from-file=db_server=server.txt --from-file=config.json`

// Secret objelerinin listelenmesi
`kubectl get secret`

// Secret objelerinin silinmesi
`kubectl delete secret "secret_ismi"` |  `kubectl delete secret my-secret`


// Degerlerin dosyadan okunarak secret olusturulmasi
`kubectl create secret generic mysecret --from-file=db_server=server.txt --from-file=db_username=username.txt --from-file=db_password=password.txt`

// Imperative olarak ConfigMap objelerinin oluşturulması
`kubectl create configmap "configmap_ismi" --from-literal="anahtar"="değer" --from-file="anahtar"="değerin_okunacagi_dosya" --from-file="değerin_okunacagi_dosya"` | `kubectl create configmap myconfigmap--from-literal=db_server=db.example.com --from-file=db_server=server.txt --from-file=config.json`

//ConfigMap objelerinin listelenmesi
`kubectl get configmap`

// ConfigMap objelerinin silinmesi
`kubectl delete configmap <configmap_ismi>` |`kubectl delete configmap my-configmap`

# Configmap
Gizli olmayan verileri anahtar/deger eslenikleri olarak depolamak icin kullanilan bir API nesnesidir. Podlar, ConfigMap'i environment variable, komut satiri argumanlari veya bir volume olarak baglanan yapilandirma dosyalari olarak kullanabilir.

Secretlar ile ayni gorevi gorurler. Olusturulan configmap dosyalarindaki key value ler ile verileri tutup bunlari env var olarak ya da volume olarak podlara aktarabiliriz. Olusturma adimlari da nerede ise birebir aynidir. Sadece `secret` yazilan yerlere `configmap` yazilir. 

- *Peki birebir ayni ise neden iki farkli obje tipi vardir?* Secret, hassas verileri saklamak icin kullanilan objelerdir. Olusturulan secretler base64 encode edilerek `etcd` de saklanir ve ayar yaparsak `etcd` uzerinde encryptsiz sekilde durabilir. `ConfigMap` de ise veriler base64 edilmez ve encrypt etmemize de gerek yoktur. Cunku bizler config map icerisinde gizli olmayan fakat yine de pod tanimimizdan ayirmamiz gereken configurasyon tarzi bilgileri tutariz yani uygulama disinda tutmaniz gereken veri, sifre, ssh anahtari gibi gizli bir veri secret degilse `configMap` uygun olacaktir. Bu ikisinin `etcd` de encoded olarak tutulmasi disinda birebir aynidir. Ilerde secretlarin guncellenmesinin otomatize edilmesi ya da ozel secret store larda ayri tutulmasi gibi ek ozellikler getirilebilir. Bu nedenle en bastan iki farkli obje tipi olusturmus k8s.

Yaml dosyasi secret gibi. Tek farki data kismina girmek istedigim bilgileri key=value seklinde girmek ya da settings altinda oldugu gibi birden fazla degeri alt alta girebiliyorum. Poda aktarma kismi secret ile ayni. Bir de secretdaki gibi tip tanimi belirtmeye gerek yok.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  db_server: "db.example.com"
  database: "mydatabase"
  site.settings: |
    color=blue # burada secerts dan farkli olarak daha fazle veri girebiliyorum
    padding:25px
---
apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  containers:
  - name: configmapcontainer
    image: ozgurozturknet/k8s:blue
    env:
      - name: DB_SERVER
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: db_server
      - name: DATABASE
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: database
    volumeMounts:
      - name: config-vol
        mountPath: "/config"
        readOnly: true
  volumes:
    - name: config-vol
      configMap:
        name: myconfigmap
```

// Imperative olarak ConfigMap objelerinin oluşturulması
`kubectl create configmap "configmap_ismi" --from-literal="anahtar"="değer" --from-file="anahtar"="değerin_okunacagi_dosya" --from-file="değerin_okunacagi_dosya"` | `kubectl create configmap myconfigmap--from-literal=db_server=db.example.com --from-file=db_server=server.txt --from-file=config.json`

// ConfigMap objelerinin listelenmesi
`kubectl get configmap`

// ConfigMap objelerinin silinmesi
`kubectl delete configmap "configmap_ismi"` | `kubectl delete configmap my-configmap`

# Node affinity(yakinlik benzesme)
`Node affinity` kavramsal olarak nodeSelector'a benzer ve nodelara atanan etiketlere gore podunuzun hangi node ustunde schedule edilmeye uygun oldugunu kisitlamaniza olanak tanir.
Podlarimizin uygun worker nodelarda olusturulmasini saglayan k8s ozelligidir. Node selectora cok benzer. Bizler pod umuzu ekler ve podumuzun schedule edilecegi node uzerinde ekledigimiz label in olmasini bekleriz. Bunun yaninda bunun olmasini sart kosar ya da tercih ettigimizi belirtiriz.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodeaffinitypod1
spec:
  containers:
  - name: nodeaffinity1
    image: ozgurozturknet/k8s
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: app
            operator: In #In, NotIn, Exists, DoesNotExist
            values:
            - blue
---
apiVersion: v1
kind: Pod
metadata:
  name: nodeaffinitypod2
spec:
  containers:
  - name: nodeaffinity2
    image: ozgurozturknet/k8s
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: app
            operator: In
            values:
            - blue
      - weight: 2
        preference:
          matchExpressions:
          - key: app
            operator: In
            values:
            - red
---
apiVersion: v1
kind: Pod
metadata:
  name: nodeaffinitypod3
spec:
  containers:
  - name: nodeaffinity3
    image: ozgurozturknet/k8s
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: app
            operator: Exists #In, NotIn, Exists, DoesNotExist
```

Yaml dosyasinda `nodeAffinity` tanimi `affinity` altinda yapiliyor burada iki secenek var bunlardan ilki `requiredDuringSchedulingIgnoredDuringExecution` tanimi su demek: Bu podu olustururken mutlaka altta yapmis oldugum eslesmeye uygun bir node bul ve bu podu orada olustur. Uygun node bulamazsan olusturma. Bu yaml dosyasini k8s api ye gonderirsem nodelarda app=blue olan bir labelli node arayacak buna uygun bir node bulursa podumu orada schedule edecek. Tanim required oldugu icin eger buna uygun bir worker node bulamazsa da schedule etmeyecek pod `pending` olacak. Kisacasi bu anahtar bizim gerekli bir eslesme varsa onu belirttigimiz bir anahtar. Buradaki bir kac secenek `nodeselector`den ayiriyor. Yaml dosyasinda operator kisminda `in` yerine `not in` dese idim bu sefer pod app=blue tanimlanmamis bir node uzerinde olusturulacakti. Buna da `antiaffinity` diyoruz ya da operator olarak `Exist` degerini girsem bu podu calistirmak icin uzerinde app anahtari olusturulmus herhangi bir worker node bul demek olacakti. Yani degeri onemli degil, app olsun da blue da olur. `DoesNotExist` ise uzerinde app anahtari olmayan bir worker node bul ve orada calistir demek olacakti. `DoesNotExist` ise uzerinde app anahtari olmayan bir node da calistir. Bu secenekte buradaki sartlar gerceklesmez ise pod calismayacaktir ancak cogu zaman bunu istemem.
Mesela podlarimin ssd li worker nodelarda calismasini isteyebilirim. Nodelara label olarak ssd atamasi yaparim ve required alanin da ssd olarak belirlerim ancak burada da sorun su ki bu worker nodelarin kapasitesi doldugu zaman ne yapacaksiniz? O zaman gene de olusturulsun derseniz `prefer` dir. 
`preferredDuringSchedulingIgnoredDuringExecution` Prefer secenegi su anlama gelir bak burada tanim yapiyorum buna bak bu podu bu tanima uygun bir node uzerinde olustur. Fakat bu tanima uygun bir node bulamazsan da pending de bekleme baska bir node bul. Yine orada olustur prefer taniminda gordugunuz uzere `weight` de girebiliyorum. 1 ile 100 arasinda bir deger girilebiliyor hangisinin oncelikli olarak degerlendirilebicegi belirleniyor. Ornekte 1 ve 2 var sadece. Oncelikli olarak weight i yuksek olan yani 2 olan secenek uygulanacak yoksa 1 inci secenek uygulanacak o da yerine getirilemiyorsa uygun olan herhangi bir worker nodeda olusturulur. Ornegin pod calismaya basladikten sonra ben buradaki label i worker node dan sildim. `Ignoredduringexecution` ise label silinse de bu pod orada calismaya devam etsin anlamina gelmektedir. Bu secenegin baska bir alternatifi yok. Yani `preferedduringexecution` diye bir secenek yok.

# Pod affinity
Pod affinity podunuzun hangi node uzerinde olusturulmaya uygun oldugunu nodelardaki etiketlere gore degil halihazirda nodeda calismakta olan podlardaki etiketlere gore sinirlamaniza olanak tanir.
Podun uzerinde calistiracagimiz worker node da calisan baska podlarin durumuna gore podun nerede calisip calismamasini isteyecegimiz durumlarda kullaniriz. 
Ornegin bir azure uzerinde kosan 4 farkli worker node uzerinde 2 farkli AZ uzerinde kosan bir cluserimiz var. Ornegin frontend cache ve veri tabanindan olusan bir uygulamamiz var. Ilk olarak veri tabanimizi deploy edecek pod tanimimizi hazirladik ve k8s api uzerine gonderdik. K8s scheduler isini yapti ve bu podun calisacagi uygun bir node buldu ve varsayalim ki node1 node2 sonra frontend i deploy ettik onu da node3 de schedule etti. Son olarak da cache podumuzu schedule ettik onu da node4 de schedule etti. Buradaki sorun ise frontend ve backend podum ayri AZ uzerinde schedule edildiler. Benim frontend sunumum veri tabani ile surekli gorusuyor ve hemen hemen tum cloud providerlarda 2 AZ arasindaki veri transferi ucrete tabi. Yani ben bu iki podu da ayni AZ uzerinde bulunan worker nodelarda kostursa idim veri transferine para odemeyecektim. Bunun icin `nodeaffinity` yazarak podlari ayni yere toplayabilirim. Ancak bu sefer her seyi manuel ayarlamam gerekir. Eger ben front end tanimina podu olustururken DB poduna bak ve o pod hangi AZ de olustu ise onu da orda olustur der ve sorunu cozerdim.
 
Diger bir ornek de cache ile frontend podlari keske ayni makinada olsa cunku bu iki pod arasinda ne kadar az latency olursa performans artar. Eger ben cache pod tanimima frontend poduna bak eger o hangi node da olusturudu ise o node da olustur diye yazsa idim sorun cozulurdu. 
Nasil node affinity de bir podun o node da olusturulup olusturulmamasi label lara gore secebiliyorsak bunu da o node uzerinde calisan pod olup olmadigina gore secebiliyoruz.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontendpod
  labels:
    app: frontend
    deployment: test
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: cachepod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: kubernetes.io/hostname
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: color
              operator: In
              values:
              - blue
          topologyKey: kubernetes.io/hostname
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: deployment
              operator: In
              values:
              - prod
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: cachecontainer
    image: redis:6-alpine
```

Nodelar uzerinde onceden tanimlanmis pek cok label bulunmaktadir. Her node uzerinde `kubernetes.io/arch`, `kubernetes.io/hostname`, `kubernetes.io/os` isimli labeller bulunmaktadir. `kubernetes.io/arch` node un x64 ya da arch tabanli bir isletim sistemine sahip oldugunu belirtir. Anahtarinin degeri amd64 olabilir isletim sisteminin degerini gosterir. `kubernetes.io/hostname` nodeun hostname degerini alir minikube gibi. `kubernetes.io/os` ise isletim sistemini belirtir, linux degerini alabilir. Bunun yaninda cloud providerlarda `topology.kubernetes.io/region=northeurope`, `topology.kubernetes.io/zone=northeurope-1` nodelarin hangi regionlada ve AZ lerde bulundugu bu labellarla belirtilir. 

Podun tanimlandigi yaml dosyasina bakildiginda. Node affinity tanimlarindaki gibi hard yani `required`, soft yani `prefered` olmak uzere iki tanimlama yapilmaktadir. Tek fark burada `topologykey` diye bir deger girilir. Topologykey kisminda `kubernetes.io/hostname` secersem bu podlari ayni hostname e sahip nodelar uzerinde calistir demek. Zone secseydim ayni worker node uzerinde olmasina gerek olmayacakti. O zaman ayni AZ deki herhangi bir node uzerinde calismasi gerektigi belirtilir. `Prefer` ise `nodeaffinity` ile ayni yani olsa iyi olur ancak bu sart durum degil. Pod affinity nin bir diger farki ise `antiaffinity` taniminin ayri olarak yapilmasidir. `Nodeantiaffinity` yi not in ile yapabliyorken `pod antianffinity` yi ozel olarak tanimlamamaiz gerekiyor. Bu da affinity nin tam tersi. 
Bu tanimlamalar buyuk clusterlarda sikca kullanilan bir ozelliktir. Kucuk cluster icin cok da gerekmemektedir.

# Taint ve toleration
Taint(leke) anlamina gelir. Siz bir node a anahtar veri eslenigi seklinde bir `taint` ve bu `taint`la birlikte `noschedule`, `prefernoschedule` ya da `noexecute` olmak uzere bir emir eklersiniz. Bu asamadan sonra bu node uzerinde sadece bu taint i yani bozuklugu tolere edebilecek podlar calisabilir. 

Worker1 blue olarak label landi. Worker2 ise green olarak label landi. Worker3 ise label lanmadi. Blue uygulamamizin blue label lanan node da green uygulamamizin green label lanan node da schedule edilmesini istiyoruz. Buna gore gerekli `pod affinity` ve `node affinity` tanimlamalarini yaptik ve uygulamalari deploy ettik. Baska bir ekip yeni bir pod schedule etti ve 3 pod tum nodelarda schedule edilmeye baslandi. 

Mesela 10 tane nodeunuz var bunlardan 3 tanesi cok iyi makinalar ve bu 3 makina musteriye sunulan nodelar olsun istiyorsunuz kalan 7 node ise test ortami olarak kullanacaginiz uygulamalar olsun istiyorsunuz. Yani sadece belirli tipte podlar belirli nodelar uzerinde schedule edilebilsin. 

Ornegin ben `platform=production:NoSchedule` isimli bi taint eklersem bu etiket eklenmemis hicbir pod bu `taint` uzerindeki nodelarda schedule edilemez. Ayni sekilde pod a bu labeli eklersem ve `prefer noschedule` yazarsam bu etiketi tolere edemeyen hicbir pod bu worker node uzerinde schedule edilemez. Ama bunun icin uygun baska bir node bulunamazsa da son secenek olarak bu worker node uzerinde pod calistirilabilir.

`Noexecute` secenegi ise bu tainti tolere edemeyen hicbir pod bu node uzerinde schedule edilemeyecegi gibi mevcut durumda bu worker node uzerinde bu tainti tolere edemeyen podlar da varsa o podlar da o worker node uzerinden silinerek baska uygun nodelar uzerinde yeniden olusturulur. 

// Node'lara taint ekleme.
`kubectl taint node "node_ismi" "anahtar=değer:eylem"` | `kubectl taint node minikube platform=production:NoSchedule`

//Node'lardan taint kaldırma.
`kubectl taint node "node_ismi" "anahtar-"` | `kubectl taint node minikube platform-`

Yaml dosyasinda podun birebir tolere edebilmesi icin node da girilen tolerations kisminin aynisi pod daki yaml dosyasinda da olmali.
- Taint ve toleration node affinity nin yapmis oldugunu yapmaz. Yani bu ozelliklere gore git surada schedule ol demez. *Yani bir pod surada olusturulsun node affinity, worker node unun uzerinde sadece su podlar calisabilsin taint and toleration.*
Calisan podlar varken taint ayarlarini degistirdigim zaman calisan podlar terminate olacaktir.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleratedpod1
  labels:
    env: test
spec:
  containers:
  - name: toleratedcontainer1
    image: ozgurozturknet/k8s
  tolerations: # bu podun taint i toleration yapilabilmesi icin birebir ayni degerlerin girilmesi gerekiyor
  - key: "platform"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
---
apiVersion: v1
kind: Pod
metadata:
  name: toleratedpod2
  labels:
    env: test
spec:
  containers:
  - name: toleratedcontainer2
    image: ozgurozturknet/k8s
  tolerations:
  - key: "platform"
    operator: "Exists" # eger platform ve noschedule olarak girilmis bir tanim varsa tolere et yani degeri onemli yeterki platform degerli olsun
    effect: "NoSchedule"
```

# DaemonSet
`DaemonSet`, tum (veya bazi) nodelarin bir podun bir kopyasini calistirmasini saglar. Clustera yeni node eklendikce, onlara podlar da eklenir. Clusterdan node kaldirildiginda, bu podlar da kalidirilir. Bir DaemonSet in silinmesi, olusturdugu podlari da temizleyecektir.

Deployment objelerine oldukca benzeyen bir k8s objesidir. Bir `daemonset` objesi olusturdugunuz zaman daemonset sistemde template altinda belirttiginiz pod tanimina gore bir pod olusturur. Varsayilan olarak her node uzerinde bir pod olusturulur. Fakat siz bunu degistirerek sadece belirli nodelar uzerinde de olusturulmasini saglayabilirsiniz. Storage provisioinng ve log uygulamalari gibi uygulamalarin her node de kolayca deploy edilmesini saglar. Bu uygulamalar icin DaemonSet kullanmak su avantaji saglar: 
  1. Is basitlesir 
  2. Yeni node geldiginde uygulamayi orada da olusturur. Yeniden bir islem yapmaniza gerek kalmaz. 

Ornegin 20 worker nodedan olusan bir K8s clusterimiz var. Bizim bu worker nodelarin tamaminda calismasi gereken bir uygulamamiz mevcut. Ornegin biz bu worker nodelarda olusan loglari merkezi bir log sunucusuna gondermek istiyoruz. Her bir worker nodeda da bir uygulama calistiracagiz ve bu uygulama o worker node da olusan loglari toplayacak ve merkezi log sunucusuna gonderecek. Ilk olarak tum worker nodelara baglanarak bu uygulamayi kurabilirim ama bu imkansiz. Ikinci olarak bu uygulamayi container image i haline getiririm ardindan gerekli ayarlarin oldugu 20 pod tanimi yaparim. Her birine node affinity eklerim ve bu podlari ayri ayri worker nodelar uzerinde calisacak sekilde deploy ederim. Bu iki yontem de zahmetli bu durumda yeni node eklersem bu islemleri yeniden yapmak zorunda kalirim.

*Deployment la benzer yaml dosyalari vardir ancak rollout ozelliklerini kullanmiyoruz temel fark bu.* 2 seye dikkat etmek gerekir. daemonset de de `label selector` tanimi vardir ve `daemonset` olusturacagi bu podlari bu `selector` a gore secer. Bu nedenle ayni label taniminin `spec` de de olmasi lazim. Minikube kurdugumuz icin bunu gormedik ama normalde `kubeadm` ile kendi kurdugumuz clusterlarda ya da cloud saglayicilarin  sagladigi clusterlarda master node uzerinde pod calistirmayiz. Master node larda `node-role.kubernets.io/master:NoSchedule` adinda bir `taint` eklidir. Dolayisiyla bunu tolere edecek bir tanim ekli degilse pod bu master node uzerinde schedule edilmez. Eger biz worker node un sadece `daemonset`lerde pod olusturmasini istiyorsak o zaman bir sey eklememize gerek yok. Ama bizim `daemonSet` imizin master node larda da pod olusturmasini istiyorsak o zaman bu toleration u buraya eklemek zorundayiz. 

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logdaemonset
  labels:
    app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

// DaemonSet objelerinin listelenmesi 
`kubectl get daemonset`

// DaemonSet objelerinin silinmesi
`kubectl delete daemonset "daemonset_ismi"` | `kubectl delete daemonset my-daemonset`

# Persistent Volume and Persistent Volume Claim
Pod yasam suresinden daha fazla saklamamiz gereken verileri sakladigimiz ve cluster disinda tuttugumuz volumelere `persistent volume` diyoruz.
3 nodelu bir cluster uzerinde mysql DB var. Tek deployment olacak bir replicaset tasarladik. Deployment in icerisinde mysql veri tabani dosyalarinin duracagi emptyDir volume olusturmasi icin tanim ekledik. Buna container a mount edecek tanimlari da olusturduktan sorna deployment objemizi de olusturduk. Mysql containlardan olusan podumuz uygun bir node uzerinde olusturuldu. Bu noktada container da sikinti olusturulursa container yeniden baslatilacak ve volume ekledigimiz icin verilerimize birsey olmayacak. Ancak diyelim ki podun calistigi worker node a birsey oldu ve terminate oldu. Podumuz uygun olan baska bir worker node uzerinde yeniden yaratilacak. Bu durum `ephemeral` volume kisminda anlattigim uygulama icin sikinti yaratmazken mysql uygulamasi icin sikinti olusturabilir. Cunku mysql yeniden olusturabilecegimiz degil uzun sure barindirmak zorunda oldugumuz kayitlari tutuyor. Poda ne olursa olsun bu kayitlari kaybetmememiz gerekiyor. Podu olusturduk mysql calisti ve verileri `emptyDir` tipindeki volume e yazmaya basladik. Bu volume fiziksel olarak o container in calistigi worker node uzerinde duruyor artik o worker node a erisilemiyor. Dolayisiyla pod baska bir worker node uzerinde yeniden olusturuldugunda bu dosyalara erisilemeycek. Bunun cozumu clusterin disinda olan ancak tum worker nodelar tarafindan erisilebilen bir klasorde da olusturuyor olabilmemiz lazim. Bunu yaparsak veri devamliligi saglayabiliriz. 

Bunun icin ilk once bir kac ayar yapmamiz gerekiyor: 

Ilk olarak volume olarak ayarladigimiz depolama birimi ile clusterimizin konusabilmesi gerekiyor. Bunun icin de cluster uzerinde bu depolama biriminin volume driverlarinin yuklenmesi gerekiyor. Kubernetes `nfs` ve `Isqz` gbi universal protokollere ait driverlarin yaninda `azure disk`, `azure file`, `Aws ebs`, `google persistent` disk gibi pek cok storagelarin driverlarini bunyesinde barindirmaktadir. Yani K8s default olarak bunlarla konusabiliyor durumda ancak depolama cozumleri yalnizca bunlarla sinirli degil. K8s bunlarin da disindaki storagelarla konusabilecek cozumleri `Container Storage Interface (CSI)` ile sagliyor.

CSI istege bagli blok ve dosya depolama sistemlerini K8s gibi Container Orchestration Systems (CO'ler) uzerindeki containerized is yuklerine maruz birakmak icin bir standart olarak gelistirilmistir. Container Stoarge Interface'in benimsenmesiyle K8s volume katmani gercekten genisletilebilir hale geldi. Ucuncu taraf depolama saglayicilari, CSI kullanarak cekirdek K8s koduna dokunmak zorunda kalmadan K8s te yeni depolama sistemlerini aciga cikaran eklentiler yazabilme imkanina kavustu. 

CSI k8s storage altyapsinin nasil ayarlanmasi gerektigini belirten bir standart. Depolama cozumu uretenler bu standarta uyan driverlar yazarak K8s in kendi altyapilari ile de konusabilmelerine imkan saglamaktadir. Ozetle baglanabilecegimiz depolama birimleri nfs, azure, aws, gcp gibi degilse bu sefer kullanilacak storage in `csi` driver ini sisteme yuklemek gerekiyor.

Ilk adim cluster ile depolama aracini birbirine bagliyoruz ve konusabilecek hale getiriyoruz. Sonra depolama birimleri uzerinde k8s de kullanmak uzere depolama birimleri yaratiyoruz. Sonra bu depolama birimlerinin k8s deki karsiliklarini k8s altinda persistent volume olarak bir obje seklinde olustururuz. 
K8S clusterimiz altinda `nfs` tabanli bir depolama birimi oldugunu varsayalim. k8s altinda nfs driverlari bulundugu icin ek bir driver yuklememize gerek kalmadan bu iki ortam birbiriyle gorusebilir durumda. Sirada bu storage uzerinde bizim erisebilecegimiz bir depolama birimi yaratma isi var. `nfs` cihazina baglaniyor ve `tmp` diye bir paylasim alani yaratiyorz. Sonrasinda bunun k8s deki karsiligini yaratma isi var bunun karsiligi k8s de `persistent volume` dur. 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
   name: mysqlpv
   labels:
     app: mysql
spec: # depolama unitesine baglanirken kullanilacak driver a gore degisiyor
  capacity:
    storage: 5Gi # drive a gore degismez
  accessModes: # driver a gore degismez
    - ReadWriteOnce # birden fazla pod a 
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /
    server: 10.255.255.10
```

Burada olusturulacak yaml dosyasindaki spec kismi baglanilacak driver a gore degisiyor. Driver a gore degismeyen 3 secenek var:  
  1. `Capacity`: bizim ne kadarlik bir volume yaratmak istedigimzi belirtiyor 5Gi yani 5 gibibayt gibi. 
  2. `AccesModes`: kisminda bu volume un ayni anda birden fazla volume e baglandigi zaman ne sekilde bir davranis sergileyecegini secebiliyrouz. Burada 3 senecek mevcut `readWriteOne` okuma yazma -tek node, `readOnlyMany` sadece okuma -birden fazla node, `readWriteMany` okuma yazma - birden fazla node. Buradaki secenekler driver in yeteneklerine ve baglanilacak storage a degisebiliyor. 
  3. `persistentVolumeReclaimPolicy`: Secenegi ise podumuzun isi bitip birakildiktan sonra bu volume e nasil davranilacagini bildiriyoruz. Burada `retain`: volume kullanildiktan sonra oldugu gibi kaliyor. Bizler bu dosyalari manuel olarak tasiyoruz ve kurtarma imkanina sahip oluyoruz. `recycle` seceneginde icindeki bilgiler siliniyor ancak volume silinmiyor. Volume tekrar kullanilabiliyor. `Delete` ise volume ile is bittiginde tamamen siliyor.
   
- Volume olustugunda pod a nasil bagliyoruz. Direk olarak bir persistent volume bir pod ile bagalayamiyoruz. Oncelikle persistent `volume claim` kisaca `pvc` bizlerin sistemde bulunan `pv` lerden isimize yarayan bir tanesini secmemize yani bunu kullanmak adina talep yaratma objesidir. Neden bu talep etme farkli objede tanimlaniyor. Mesela ben developer olabilirim ve yonetme isi farkli bir yonetici tarafindan yapiliyor olabilir. Dolayisiyla o islemleri ben bilemiyor olabilirim.
  
Peki bu volumeleri poda nasil baglariz? Yaml dosyasinda volume tanimlarinin altinda yeni bir volume olustur ve bunun hedefini `persistentVolumeClaim` olarak belirler sonrasinda bunu pod altinda gerekli path e mount ederiz. Bu pod olusturuldugu anda bu path devreye girer. Bu podun olusturulacagi worker node uzerinde bu volume ile ilgili ayarlamalar yapilir. Sonucunda bu pod bu pathde ilgili volume baglanir ve bu path e yazilan dosyalar bu volume e yazilir.

`Pv` yaratirken label onemli cunku pvc ler bu labellara gore istek yapilmaktadir. Pv lerin yaratilmasina kadar olan kisim system yoneticilerin daha sonra bu `pvc` nin kullanilmasi kismi ise developerlara ait olan bir kisim. 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem # hangi mode ile baglanmak istiyorum
  resources:
    requests:
      storage: 5Gi # ne kadarlik volume talep ediyorum
  storageClassName: ""
  selector:
    matchLabels: # pvc ile pv yi label uzerinden eslestiriyorum
      app: mysql # bu pvc bu etikete sahip pv tarafindan saglansin istiyorum.
```

Pod tanimlarinda persistentVolume Claim altinda bir claim olusturuz ve bunu pod altinda gerekli path mount ederiz. Bu pod olusturuldugu anda pv deki tanima gore worker node uzerinde ayarlamalar yapilir. Storage cihazina baglanilir driver yapmasi gerekenleri yapar ve sonunda bu pod icerisindeki container ilgili pathin depolama cihazi uzerinde duran bu volume baglanir. Bu noktadan itibaren bu path e yazilan dosyalar aslinda bu volume e yazilir. Eger bu pod bulundugu node uzerinden baska bir node calisirilirsa ayni islemler burada da gerceklestirlir ve yeni pod a bu volume baglanir.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysqlsecret
type: Opaque
stringData:
  password: P@ssw0rd!
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysqldeployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              name: mysqlvolume
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysqlsecret
                  key: password
      volumes:
        - name: mysqlvolume
          persistentVolumeClaim:
            claimName: mysqlclaim
```

// NFS Server 
`docker volume create nfsvol`
`docker network create --driver=bridge --subnet=10.255.255.0/24 --ip-range=10.255.255.0/24 --gateway=10.255.255.10 nfsnet` |
`docker run -dit --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v nfsvol:/data --network nfsnet -p 2049:2049 --name nfssrv ozgurozturknet/nfs:latest`

// Persistent Volume objelerinin listelenmesi
`kubectl get pv`

// Persistent Volume objelerinin silinmesi
`kubectl delete pv "pv_ismi`
`kubectl delete pv mypv`

// Persistent Volume Claim objelerinin listelenmesi
`kubectl get pvc`

// Persistent Volume objelerinin silinmesi
`kubectl delete pvc "pvc_ismi`
`kubectl delete pvc mypvc`

# Storage Class
Storage class yoneticilerin sunduklari depolama "siniflarini" tanimlamalari icin bir yol saglar. Farkli siniflar, hizmet kalitesi duzeylerine veya yedekleme ilkelerine veya cluster yoneticileri tarafindan belirlenen istege bagli ilkelere eslenebilir. k8s siniflarin neyi temsil ettigi konusunda fikir sahibi degildir. Bu kavram bazen diger depolama sistemlerinde "profiller" olarak adlandirilir. 

10 ayri ekip yonetilen k8s cluser oldugunu dusunun 10'larca farkli node ve 100'lerce pod olusturuluyor. Uygulamalar surekli yaratiliyor siliniyor. Bu durumda her seferinde `pvc` ve `pv` olusturmamiz cok zahmetli olacaktir. Bu soruna cozum olarak `storage class` objesi olusturuldu. 

Pv yi manuel olursturmak yerine claime gore dinamik olusturma imkani getirdi. Bu ozellikle cloud servis saglayicilarin dinamik ortamlarinda her seyi otomatize edebilirken sirf volume olusturma islemini manuel olusturma sacmaligini cozdu. Storage class objelerini baglantimiz olan depolama uniteleri uzerinde ne cesit volume yaratacagimizi belirten templateler olarak dusunebilirsiniz. Ornegin uzerinde hem ssd hem de hdd olan storage oldugunu dusunun. Bu durumda ben yavas diskler uzerinde otomatik volume yaratan yavas isimli bir storage class ve hizli diskler uzerinde volume yaratan hizli isimli 2. bir storage class yaratabilirim. Development ekipleri de uygulamalarinda kullanmak icin volume talep etmek adina olusturduklari `pvc` de uygulamalarinin ihtiyacina gore bu storage class lardan birini secerler ve istedikleri boyutta bir volume storage class objesi tarafindan otomatik olarak olusturulup `pvc` ye bind edilir.

// storage class lari getir
`kubectl get storageclass`

Iki farkli depolama unitesi icin 4 farkli storage class olusturmus durumdayiz. Bu listedeki provisioner tanimi bizlera hangi tipde cihaz ustunde volume olusturdugumuzu belirtiyor. Kubernetes de default olarak nerdeyse tum cloud saglayicilarin ve bir cok standart protokolun provision tanimlarini kendi uzerinde barindiriyor yani kendi driverlarini kendi uzerinde barindiriyor. Fakat sizlerin baska tipde storage classlarniz varsa onlarin da kendi provisionlarina eklemenize imkan veriyor. Bir tane normal bir tane de premium disk olurturmus. Azure icin bu su demek standart daha yavas ve sla yi daha dusuk, premium ise daha hizli ve sla yi daha yuksek. Depolama urunlerinin sagladiklari turlere gore farkli turlerde volume yaratilmasina imkan sagliyor. Storage class bir template dir. Biz bu template i kullanarak otomatik volume yaraticaz. Ben bunu daha yavas bir sey yaratmak istersem `azurefile` kullan dicem daha hizli bir sey yaratmak istersem `azurefile-premium` kullanicam, yavas bir disk bazli bir storage yaratmak istersem `managed` disk yaraticam, hizli bir ssd yaratmak istersem `managed-premium` yaraticam. Bir cok storage class yaratarak ben depolama unitelerimin degisik ozelliklerine gore pv ler yaratilmasi icin sablonlar olusturabilirim. Diger bir husus `default (default)` yaziyor buna dikkat etmek lazim. `pvc` olusturdugumuz zaman hangi tipde volume istedigimizi storage class belirterek sagliyoruz fakat o kismi bos gecersek de varsayilan olarak hangi storage class kullanilsin secenegini bu sekildee default olarak atayarak saglabiliriz. Yani `pvc` nin icinde isim belirtmez iseniz `default (default)` isimli template kullanilacaktir. Isterseniz default u da degistirebilirsiniz. Diger bir husus `reclapimpolicy` yani bu storage class ile isimiz bittikten sonra ne olacak? Sadece `retain` ve `delete` secenegi mevcut, azure default olarak `delete` secmis. Son olarak `volumebindingmode` kisminda `Immediate` yaziyorsa hemen volume un olusturulup pod a atanmasini soyler. `WaitForCounsumer` ise pod yaratilana kadar beklemesi ve pod yaratildiktan sonra storage class in yaratulmasi gerektigini soyler.

Cloud ortaminda bu secenekleri bilmek gerekmiyor clous servis saglayicilar bunlari otomatik sagliyor ama bu detaylari kendi clusterinizi yaratiyorsaniz olabilir. Ama size saglayan format bunlari size belirtir bilgi verir. 
pv.yaml dosyalarindan tek farki burada label kullanmiyorum onun yerine kullanmak istedigim `storageClassName` yerine `statndarddisk` yani diskin ozelligini yaziyorum. 

Bu file ile kendime ait bir storage class olusturabiliyorum. Aslinda bunu cloud ortaminda calsiyorsaniz gerek yok. Azue in dokumantasyonundan bu ayarlari bakarak olusturdum. Admin olarak tek yapmam gerekiyor. 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standarddisk
parameters:
  cachingmode: ReadOnly
  kind: Managed
  storageaccounttype: StandardSSD_LRS
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Pu pvc su manaya geliyor: git `standartdisk` tanimindaki ozelliklere gore bana 5Gi lik ve `accessModes` u `ReadWriteOnce` olan bir `pvc` olustur. Bu pvc yaml dosyasi ile artik selector ile hangi pv i kullanmam gerektigini secmiyorum. Onun yerine `storageClassName` alaninda storage class secmem gerekiyor. 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: "standarddisk"
```

# StatefulSet
1. `StatefulSet` tarafindan olusturulan her pod, stateful set taniminda belirlediginiz pvc'ye gore olusturulan bir pvye sahip olur yani her podun kendine ait bir pv si olur.
2. StatefulSet altinda podlar sirayla olusturulup sirayla silinir. Ornegin 3 podlu bir stateful set olusturdugunuz zaman oncelikle pod olusturulur. Bu pod ayaga kalkip readiness ve liveness checklerinden gecip islemlerini tamamlamadan bir sonraki pod olusturulamaz. Ne zaman ki bu pod hazir running durumuna gecer o zaman bir sonraki pod olusturulur. Ayni sekilde 3.pod da 2. pod tamamlanmadan olusturulamaz. Tam tersi durumda da bu gecerlidir. Siz 3 podlu bir statefullseti 2. poda indirirseniz k8s random bir pod secip onu silme yoluna gitmez. En son yaratilan pod hangisi ise ilk olarak o silinir.
3. StefulSet tarafindan olusturulan her poda statefulsetin adi 0-1-2-3 seklinde devam eden sabit bir isim verilir. Isimler random secilmez ve bu isimler ayni zamanda containerin hostname'i olarak da atandigi icin her uygulama bu isimle ulasilabilir.

Sorun: 3 pod olusturacak bir deployment var. Bu obje bir `replicaset` olusturuyordu ve bu replicaset de bizim belirledigimiz tanima gore de random isimler verilen podlar olusturuyordu. `Scale out` ile `scale in` ile bunlarla oynayabiliyorum ama k8s bunlardan hangisi once yaratilip yaratilmadigina bakmiyor randomly siliyor. Podlar uzerinde bir state tutmadigi icin bu durum sikinti yaratmiyor. Ancak yeni bir nosql veri tabani deploy etmek istiyorsunuz. Bir tane master instance olusturur ve bunun uzerinden cluster olsuturur ardindan bu clustera yeni instancelar eklersiniz. Yazma islemlerini bu master uzerinden yaparken sorgulari herhangi bir instance a gonderebilirsiniz. Tum veri bu instancelar uzerinde dagitik bir sekilde durur. Boyle bir veri tabani olan apche cassandra yi 3 instance dan olusacak bir cluster olacak sekilde su ana kadar ogrendigimiz obje sekilleri ile k8s e deploy etmeye calisalim. Ilk olarak bu cassandra master instance i deploy edecegiz ardindan buna baglanarak ya da baslangic komutlari ile bunu master haline donusturecek sekilde bir cluster haline getirecegiz. Sonrasinda 2. instance i olusturacagiz ve ardindan 2. instance a baglanip 1. instance daki cassandra cluster a bu 2. instance i dahil edecek ayarlari yapacagiz. sonra 3. instance da da ayni islemleri yapacagiz. Ilk olarak `singleton pod` olacak sekilde olusturmaya karar verdik. Master olacak pod tanimini yaptik icine pvc tanimini ekledik ki cassandra verileri tutabilecek pv ye sahip olabilsin. Pod ayaga kalkti bunu master haline getirecek komutlari ya baslangic olarak ya da sonradan baglanarak halletik. Ardindan 2. bir pod tanimi daha yaptik. Bunu da 2. instance olarak ayaga kaldirdik. Sonra 3. derken cluster ayaga kalkti. Sorun 1: her seyi manuel yaptim. Sonra `singleton pod` olusturduk bu da fail durumu kontrol eden bir mekanizma yok. Son olarak 4. bir instance eklersem yeni pod ve maneul ekleme olacak. Bu sorunu deployment la dener isem bu hepsi birbirinin ayni pod u olusturdu ve hepsi ayni anda olustu ve bu nedenle hangisinin once olsutugunu bilmiyorum neyse birine manuel baglandim ve birini master a cevirdim digerlerini de worker yaparak cluster a kattim. Aradan zaman gecti pod lardan birine ulasilamiyor sonra yeniden olustu ama o zaman bu master pod olursa o zaman cluster coker. Bunlarin hepsi cassandra gibi `stateful` objelerde sorun yaratiyor. Bunun cozumu ise `StatefulSet` objesidir.

`Stateful` objesi deploymenta cok benzeyen bir obje temelde 3 farki var: 
1. stateful tarafindan olusturulan her pod `statefulset` taniminda belirlediginiz pvc ye gore olusturulan bir pv ye sahip olur. Yani 
2. Her pod un kendine ait bir `pv` si bulunur. 
3. Bunun yaninda `stateful` altinda pod lar sirayla olusturulurp sirayla silinir. Pod lar silinirken de bu durum aynisidir. Random olarak yapmaz ve son yaratilan pod ilk olarak silinir. Her pod a statefulun adi-0 ve sirayla olaack sekilde sabit bir isim verilir. Isimler ayni zamanda container hostname olarak atandigi icin her uygulama bunlara ulasabilir. Geriye kalan tum ozellikleri deploymentla aynidir. Bu 3 ozellik sayesinde stateful objesi ile yaratmamiz kolaylasir. 

Senaryomu `statefulset` objesi ile yapmak istedigimde senaryo soyle ilerleyecektir:
3 pod olusturacak bir `stateful` objesi deploy ettim. K8s hemen ilk podu ayaga kaldirdi. Ben bu poda bir baslangic scripti eklemistim. Bu script ortamda benim belirledigim isimde bir cluster var mi yok mu bunu konrtol ediyor. Yoksa da bu isimde cluster yaratiyor. Cluster yaratildi. `Liveness` ve `readiness` prob tamamlandi ve hem cluster ve pod hazir. 1. pod saglikli bir sekilde calismaya basladiginda 2. pod olusturulmaya baslandi bunda da script var ancak simdi ortamda cluster mevcut. Dolayisiyla mevcut clustera dahil oldu `liveness` ve `readiness` problar tamamlandi ve bu pod da hazir. 3. pod sonrasinda cluster tamamlandi. Peki cassandra-1 podu silinirse `stateful` objesi ayni podu ayni isimle ayni persistent volume u atayarak ayaga kaldiracak ve sorun olmayacak. Yeni pod eklersem ne olacak yeni olusturacak ve cluster a dahil olacak. Scale down oldugu zaman da en son yaratilan pod silinecek.

Stateful tanimlanmasi deployment a cok benziyor. Replica sayisini belirtiyor ve label selector u yaziyoruz. Template kisminda olusturulmak istenen pod tanimini yaziyoruz. Yaml dosyasinda `volumeClaimTemplates` kisminda her pod icin pvc olusturulmasini soyluyorum. Bu pvc lerde standart isimli storage class i kullanarak her bir pod icin birer pv olusturacak. Yani her podun ayni pv ye baglanmasi yerine her poda ayri bir pv yaratilacak. Bunu da yaratacak pvc leri de tempate ile yaratiyouz. Bu template i buraya yaziyoruz ki pvc ler olusturulsun o pvc lerde pv leri olusturacak sonrasinda o podlara tek tek baglayacak. Servis tanimlanmasinda ise clusterIp none diyoruz. Buna `headless` servis diyoruz. Bunu olusturdugumuz anda clusterIp tipi bir servis olusturulacak fakat ona bir ip atanmayacak bu bana su sekilde yardimci oluyor. Ben ne zaman bu servis ismine gormek istersem o bana bu servis altindaki podlardan bir tanesinin ip sini donecek bunun yaninda her bir pod a da `statefulset ismi-<pod olusturma sirasi>.statefulset ismi` ismi seklinde erisme imkani da verecek. Podlar 0 dan baslayarak isimlendirilir. Ready olduktan sonra diger pod olusturulur.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
  - port: 9042
  selector:
    app: cassandra
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
  labels:
    app: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      terminationGracePeriodSeconds: 1800
      containers:
      - name: cassandra
        image: gcr.io/google-samples/cassandra:v13
        imagePullPolicy: Always
        ports:
        - containerPort: 7000
          name: intra-node
        - containerPort: 7001
          name: tls-intra-node
        - containerPort: 7199
          name: jmx
        - containerPort: 9042
          name: cql
        resources:
          limits:
            cpu: "500m"
            memory: 1Gi
          requests:
            cpu: "500m"
            memory: 1Gi
        securityContext:
          capabilities:
            add:
              - IPC_LOCK
        lifecycle:
          preStop:
            exec:
              command: 
              - /bin/sh
              - -c
              - nodetool drain
        env:
          - name: MAX_HEAP_SIZE
            value: 512M
          - name: HEAP_NEWSIZE
            value: 100M
          - name: CASSANDRA_SEEDS
            value: "cassandra-0.cassandra.default.svc.cluster.local"
          - name: CASSANDRA_CLUSTER_NAME
            value: "K8Demo"
          - name: CASSANDRA_DC
            value: "DC1-K8Demo"
          - name: CASSANDRA_RACK
            value: "Rack1-K8Demo"
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - /ready-probe.sh
          initialDelaySeconds: 15
          timeoutSeconds: 5
        volumeMounts:
        - name: cassandra-data
          mountPath: /cassandra_data
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
```


// StatefulSet objelerinin listelenmesi
`kubectl get statefulset`

// StatefulSet objelerinin silinmesi
`kubectl delete statefulset "statefulset_ismi"` | `kubectl delete statefulset my-statefulset`

# Job
Bir job objesi, bir veya daha fazla pod olusturur ve belirli sayida pod basariyla sonlandirilana kadar pod yurutmeyi yeniden denemeye devam eder. Job belirtilen sayida podun basariyla tamamlanma durumunu izler. Belirtilen sayida basarili tamamlamaya ulasildiginda, job (yani gorev) tamamlanir. Bir job un silinmesi olusturdugu podlari temizlemeyecektir. Bu sayede podlar tarafindan olusturulmus loglar kontrol edilebilir.

Hangi durumlarda kullanilir? 
  1. Tek seferlik calisip yapmasi gerekeni yapip kapanan uygulamalari job olarak deploy edebiliriz. ornegin maintaince scriptler. 
  2. Bir kuyruk veya bir bucket da islenmesi gereken pek cok isimiz oldugunda bunlari eritme adina bunlar eriyene kadar calisacak uygulamalari job seklinde deploy ederiz.

Apisi batch/v1, spec kismida 4 secenek var. `paralelism:` pod olusturulma islemini kacar kacar yapacagini. `completions:` job altinda kac tane basarili pod calismasini istiyoruz. Template kisminda container bilgilerini yaziyoruz. Bu ikisinin hemen altinda `backofflimit` podlar olusturulmaya baslandikten sonra kac sefer fail ederse job u fail et demek. `ActiveDeadlineSeconds` ise job u fail etmeyi second cinsinden belirtiyoruz. 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  parallelism: 2
  completions: 10
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never #OnFailure 
```

Job yarattiktan sonra isi biten podlari manuel olarak silmek gerekiyor ki yer kaplamasin artik.

// Job objelerinin listelenmesi
`kubectl get job` 

// Job objelerinin silinmesi
`kubectl delete job "job_ismi` | `kubectl delete job myjob`

// CronJob objelerinin listelenmesi
`kubectl get cronjob`

//CronJob objelerinin silinmesi
`kubectl delete cronjob "cronjob_ismi` | `kubectl delete cronjob mycronjob`

# CronJob
Bir `CronJob` objesi bir crontab (cron tablosu) dosyasinin bir satiri gibidir. Belirli bir zamanlamaya gore Cron formatinda yazilmis bir Jobu periyodik olarak calistirir.

Job objesini manuel baslatmak yerine her pazar ya da her 5 dk da bir baslatilmasina imkan taniyan bir obje. Linuxdeki crontab in aynisi. 

```yaml
apiVersion: batch/v1beta1 # not stable until kubernetes 1.21.
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
  
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
#
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
#
# https://crontab.guru/
```

# Authorization and authentication
K8s kimlik dogrulama eklentileri araciligiyla API isteklerinin kimligini dogrulamak icin istemci sertifikalari tasiyici belirtecleri bir kimlik dogrulama proxy'si veya HTTP temel kimlik dogrulamasi kullanir.
Api'de bir kullanici objesi bulunmamasina ragmen, cluster'in certificate authority'si tarafindan imzalanmis gecerli bir sertifika sunan herhangi bir kullanici (CA) kimligi dogrulanmis olarak kabul edilir. Bu yapilandirmada k8s sertifikanin 'konu' kismindaki ortak ad alanindan kullanici adini belirler (or. "/CN=bob")

Oradan, rol tabanli erisim denetimi (RBAC) alt sistemi, kullanicinin bir kaynak uzerinde belirli bir islemi gerceklestirme yetkkisina sahip olup olmadigini belirler.

Her platformun kullanici yaratma ve yonetme servisleri bulunur (IAM). Ancak K8s boyle bir hizmet sunmaz. Kullanici yaratamazsaniz. User objesi bulunmaz. Kimlik olusturma ve dogrulama isi cluster disinda yapilacak sekilde tasarlanmistir. 

x509 client certs, static token file, openID connect tokens, webhook token authentication, authenticating proxy gibi altyapilari kullanarak bu isin disarida halledilmesine imkan saglar. K8s kurulumunda kube-adm ayarlamalarinda bu altyapilardan hangisini kullanacaginizi belirlersiniz. Bu kimlik dogrulama isini bu altyapilara havale edersiniz. Bunun yaninda her k8s clusterinda bir kok sertifika yetkilisi yani `root sertifika authority` altyapisi da bulunur. Bu root sertifikalari tarafindan imzalanan x509 client certifikalari da authentication icin kullanilir. K8s bu ve benzeri uygulamalar tarafindan kimligi dogrulanmis kullanicilarin k8s ile gorusmesina imkan verir. Fakat kimligin dogrulanmasi bir kullanicinin cluster da her seyi yapabilcegi anlamina gelmez. Burada yetkilendirme yani `autherization` kavrami devreye girer. Manuel bir cluster kurmuyorsak cloud servisleri tarafindan bu islemler bizim adimiza otomatik olarak yapilir.

Minikube olusturulurken de certifika olusturulur ve kube config dosyasinin icine bu bilgiler girilir. Boylelikle her seferinde bu sertifikalarinizla api-server a istek gondermis olursunuz. Siz gidip bir obje yaratir gibi client sertifikasi yaratamazsiniz.

Ornegin bir firmada developer olarak ise basladiniz ve k8s clusteriniza baglanmak istiyorsunuz. Admine gittim bu istegimi soyledim. O da bana x500 client certificate leri ile authenticate olacak sekilde calistigini soyledi. Eger ben bir private key ve certificate suging request dosyasi hazirlayip ona gonderirsem o k8s certificate authority sini kullanarak, bunun imzalayarak bana bir certicate yaratabilecegini soyledi. Develepor olarak bunlari olusturalim ilk olarak private key ve csr dosyalarini olusturacagim bir klasor yaratalim. Private key icin openssl uygulamasini kullanmam gerekiyor. Openssl uygulamasi ile bir tane private key olusturdum. `openssl genrsa -out sezginerdem.key 2048` 2. adim olarak bu anahtari kullanarak bir certificate signing request yani csr hazirlamam gerekiyor. Csr benim certifika saglayayiciya bak bu ozelliklerde bana digital certificate sagla dememize imkan saglayan dosyalardir. Bu dosyayi olusturacak ardindan bunu admine gonderecegiz. O da k8s in certificate authority si ile bunu onaylayip imzalayarak bizim certificate imizi olusturacak ve bize bir certificate gonderecek biz de bununla k8s e ulasacagiz. `openssl req -new -key sezginerdem.key -out sezginerdem.csr -subj "/CN=sezgin@drsezginerdem.gmail/O=DevTeam"` kodu ile csr yaratiyorum. Bu kodda openssl ile request yaratmak istedigim icin `req` opsiyonunu kullandim. `-new` ile yeni bir csr olmasini, `-key` ile bu klasorde olan anahtari gosteriyorum, `-out` ile sonucun yazilacagi sezginerdem.csr olmasini istiyorum, `-subj` ile certifika bilgilerini giriyorum `CN` kullanici adi `O` ise bu kullanicilarin hangi gruplara uye olacagini belirleyecek. k8s kendi uzerinde bir kullanici objesi tutmaz. Sizin hangi kullanici oldugunuzu x500 certifikalarinizi bu alanlardan algilar. Bizler hangi kullanici adina sahip olmak istiyorsak ve bu kullaniciyi eklemek istedigimiz gruplar var ise bunlari `csr` olusturma asamasinda bu sekilde belirtiyoruz. Artik developer olarak benim isim bitti. Bu olusturdugum `csr` dosyasini k8s admina gonderecegim bundan sonraki islemleri o halledecek. Admin olarak ise yeni bir k8s objesi yaratacagim. Developer in bana vermis oldugu certificate signing request adindaki bu obje ile developerin bize verdigi cse i k8s e gonderecegim. O da bana certifica olusturacak. README.md deki adimlari teker teker gerceklestiriyorum. csr objesini olusturdum. 

//CertificateSigningRequest oluşturma
```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ozgurozturk
spec:
  groups:
  - system:authenticated
  request: $(cat ozgurozturk.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

Condition pending durumunda gorunuyor. Bunu onaylamam gerekiyor ki bu islem tamamlansin. 
`kubectl certificate approve sezginerdem` ile onayliyorum.bu serifikayi yaml dosyasi ile cikariyorum ve decode etmem gerekiyor. 
`kubectl get csr sezginerdem -o jsonpath='{.status.certificate}' | base64 -d >> sezginerdem.crt` komutu csr i goruntulemek istedigimi ve sonrasinda bunu jsonpath ile output olarak bunun icindeki status.certificate kismini al sonrasinda base64 il decode ediyorum; ve sezginerdem.crt ile kaydediyorum ve certificate olusturuldu. 
Bu certificayi developer a email ile gonderdim. Developer bu certifikayi kubeconfig dosyasina ekler ve yeni bir contex yaratirsa cluster a bununla baglanabilir. `kubectl config set-credentials sezgin@drsezginerdem.gmail --client-certificate=sezginerdem.crt --client-key=sezginerdem.key` ile kulanici set ediyorum. Bu kullanicinin minikube cluster ile iliskilendirilemsi gerekiyor. 
`kubectl config set-context sezginerdem-context --cluster=minikube --user=sezgin@drsezginerdem.gmail` ile iliskilendirme islemimi gerceklestiriyorum. 
`kubectl config use-context sezginerdem-context` komutu ile kullanici degisimi yaptim. Bu asamaya kadar clustera baglanma yetkilendirmesi idi. Ancak listeleme dahil hicbir islem yapmaya bu kullanici yetkili degil. Authenticate oldu ancak authorizate yetkileri ise kisitli.

// Key ve CSR oluşturma
`openssl genrsa -out ozgurozturk.key 2048` 
`openssl req -new -key ozgurozturk.key -out ozgurozturk.csr -subj "/CN=ozgur@ozgurozturk.net/O=DevTeam"`

//CertificateSigningRequest oluşturma
```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ozgurozturk
spec:
  groups:
  - system:authenticated
  request: $(cat ozgurozturk.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```


// CSR onaylama ve crt'yi alma
`kubectl get csr`
`kubectl certificate approve ozgurozturk`
`kubectl get csr ozgurozturk -o jsonpath='{.status.certificate}' | base64 -d >> ozgurozturk.crt` 

// kubectl config ayarları
`kubectl config set-credentials ozgur@ozgurozturk.net --client-certificate=ozgurozturk.crt --client-key=ozgurozturk.key`
`kubectl config set-context ozgurozturk-context --cluster=minikube --user=ozgur@ozgurozturk.net`
`kubectl config use-context ozgurozturk-context`

# Authentication
All user managed by kube-apiserver, kube-apiserver oncelikle kullaniciyi authencicate eder sonrasinda process requeste bakar. Nasil authenticate eder
  1. Static password file (this is not recommend)
  2. Static token file (this is not recommend)
  3. Certificates
  4. identity services (third party applications like elda, kerberotos etc)

1. Static password file and 2. Static token file: You can use a .csv file for authentication. 
file has a three coloumns password, username and user ID. And you can use a forth column for group. This file is the same as for static token file. you can write token instead of password. 

# Role Based Acced Control "RBAC"
AUTHENCICATION != Authorization

`Aurhentication`=kimlik dogrulama --soyledigi kisi mi-- != `authorization`=yetki kontrolu --eyleme yetkisi var mi--

`RBAC objeleri:` rol tabanli erisim denetimi (RBAC) kurulusunuzdaki bireysel kullanicilari rollerine dayali olarak bilgisayar veya ag kaynaklarina erisimi duzenleme yontemidir. RBAC yetkilendirmesi yetkilendirme kararlarini yonlendirmek icin rbac.authorization.k8s.io API grubunu kullanir ve Kubernetes API araciligiyla ilkeleri dinamik olarak yapilandirmaniza olanak tanir. 

Bu mekanizma role, role binding ve cluster, cluster role binding objeleri ile calisir. Yapilacabilecek islemleri yani yetkiler role veya cluster role binding objeleri olarak tanimlanir. Daha sonra role binding ve cluster role binding objeleri araciligiyla bu roller servis hesaplari, kullanicilar ya da gruplara bind edilir yani baglanir. Kullanicilar da bagli bulunduklari rollerde belirlenen yetkilere kavusurlar.

Ornegin ben default namespace de pod objesi okuyup listeleyebilecek yetkiler belirledigim bir role yaratirim. Daha sonra bir role binding yaratarak kullaniciya bind ederim. Boylelikle bu kullanici default namespace altindaki podlari goruntuleyebilir. Eger bind ettigim role yalnizca bu ise kullanici sadece bu islemi yapabilir. Baska ns de podlari goruntuleyemez. Ya da servis objesi olusturamaz. Kullanicilar sadece bind edildikleri rollerde tanimlanan yetkilere sahip olur. 

Role, rolebinding, cluster, cluster binding birer k8s objesidir. Tum objelerde oldugu gibi apiversion, kind, metadata alanlari vardir ayrica rules kisminda ise bu rule atanan yetkiler belirlenir. `Rule` kisminda ilk once `apiGroups` kismi nda hangi api de kullanilacagini belirtiriz. Burasi bos olursa `coreApi` yani v1 api ile ilgili oldugunu, resource ise hangi objeler ile ilgili oldugu ve sub objeler ile ilgili oldugunu, `verbs` ise get, watch, list gibi hangi action lari yapilabilcegi konularina yetki verilir. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"] # "services", "endpoints", "pods", "pods/log" etc.
  verbs: ["get", "watch", "list"] # "get", "list", "watch", "post", "put", "create", "update", "patch", "delete"
```

Cluster-role ise role objesi ile nerede ise aynidir. Ilk fark obje tipi cluster role ikincisi ise namespace tanimi girilmemis. Role belirledigimiz namespace icin gecerli olan namespace icin yetki vermek icin kullanilir. Cluster role de belirlenen yetkiler ise tum cluster capinda gecerlidir. Yani ben birazdan bu rolu bir kullaniciya baglarsam o kullaniciya bu rolde belirledigim haklar sadece default namespace icin gecerli olur. Fakat cluster-rolu baglarsam hangi ns de oldugu farketmeyecek. Burada tanimladigim yetkiler tum cluster capinda gecerli olacak. Bir diger fark ise su ana kadar gordugunuz her obje mutlaka ns altinda yaratilmali idi. Fakat k8s altinda namespace e bagli olmayan cluster seviyesinde objeler de var. Ornegin node lar da birer k8s objesi ve bunlar herhangi bir ns e bagli degil. Ya da `customResourceDefinition` lar da herhangi bir ns e bagli objeler degil. Bu tarz non ns objelerle ilgili yetkilendirme yaparken de cluster role kullaniyoruz. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

Bu tanimlanan yetkilerin kullanicilara baglanmasi gerekiyor bunu da `role binding` ile yapiyorouz. `Cluster role binding` ise cluster rolu baglamak icin kullanilan obje lerdir. `Subject` kisminda bu rolun kime bind edilecegini bildiriyoruz. Neyi baglamak istedigimizi `roleRef` altinda yaratiyoruz. Cluster role bindig de ise user yerine group atamasi yapiyoruz bu gruba dahi olan her kullanici bu yetkilere sahip olacak. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: sezgin@drsezginerdem.gmail # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

## service account
Kullanici hesaplari insanlar icindir. Service accounts, podlarla calisan processler tarafindan kullanilmak uzere tasarlanmistir.

k8s de deploy ettigimiz uygulamarin da kube-api ile gorusecek cluster uzerinde islem yapmasi gerekebilir. Ornegin k8s uzerinde uygulamalarinizi deploy edecek bir uygulama gelistirdiniz ve bunu bir pod olarak deploy ettiniz. Bun uygulamanin yeni uygulamalar deploy edebilmesi icin k8s api ile konusabilmesi ve deployment islemlerini gerceklestirmesi gerekiyor. Dolayisiyla bir pod uzerinde kosan bu uygulamanin oncelikle bir sekilde `authentication` adimini gecerek kimligini dogrulamasi ve sonrasinda bu islemleri yapabilmesi adina gerekli `authorizationa` yani yetkiye sahip olmasi gerekiyor. Bu ve benzer senaryolarda kullanilabilmesi adina `service account` adinda bir object tipi bulunmakta. Sertifika temelli authentication icin certificate islemleri gerekli uygulamalar icin ise service accountlar var. Yani AWS de role atamak gibi. 

K8s de her namespace icin bir adet `default service account` olusturur. Her poda aksi belirtilmedigi surece bu service account baglanir. Default service account un hicbir yetkisi yoktur. Ama bizler istersek bunlara da role ya da cluster role bind ederek gerekli yetkiyi verebiliriz.

Bir service account olusturdugumuz zaman bu service account icin bir secret olusturulur. Bu secret da 3 bilgi bulunur. Ilki service account un olusturuldugu namespace in adi. Ikinci bilgi ise bizlerin k8s api server ile iletisim kurarken https, tls baglantisi kurmamiz gerektiginden bu baglantinin hata vermemesi adina gerekli serfitika bilgisi. Son olarak da kimlik dogrulamasinda kullanabilecegim bir jwt yani jason web token bilgisi. Certificate tabanli authentication yapabildigimiz gibi http header bilgisine json web tokenlari birer token olarak gomerek de authentication olabiliriz. Her bir service account icin bu 3 bilginin oldugu bir secret yaratilir ve bu secret `var/run/kubernetes.io/serviceaccount` klasorune mount edilir. Boylece podun uzerindeki uygulamaya git buradaki dosyalardan bu degerleri oku ve kullan diyebiliriz.

Bu yaml dosyasnda bir service account olusturdum. Daha sonra bu service account u bind edecek bir rol binding olusturdum. Pod taniminda `serviceaccountname` ile hangi poda hangi servisin atanacagini belirtiyorum. Eger bu anahtarda herhangi bir sey belirtilmez ise `default service account` atanacaktir.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: testsa
  namespace: default
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: podread
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: testsarolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: testsa
  apiGroup: ""
roleRef:
  kind: Role
  name: podread
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  namespace: default
spec:
  serviceAccountName: testsa
  containers:
  - name: testcontainer
    image: ozgurozturknet/k8s:latest
    ports:
    - containerPort: 80
```

Her k8s kurulumunda cluster icinden k8s api server a erisilebilsin diye kubernetes isimli bir service yaratilir biz de cluster icindeki bi svc ye gidersek otomatik olarak kube api servere gitmis oluruz.

## Ingress
`Ingress controller:` L7 application load balancer kavraminin kubernetes spesifikasyonlarina gore calisan ve k8s'de deploy ederek kullanabildigimiz turudur. Nginx, HAproxy, Traefik en bilinen ingress controller uygulamalaridir. 

Ingress kurulumunu yaptiktan sonra eger cloud ortaminda calisiyorsak bu ingress controller dis dunyaya bir load balancer servisi ile expose oluyor ve public bir ip adresine sahip oluyoruz ve bu noktadan sonra artik dis dunyadan k8s e erisim sadece bu ip adresinden saglaniyor. Sonrasinda bu ingress controllerin configurasyonu yani hangi servise dis dunyadan hangi url e nasil ulasilacak kismini `ingress objeleri` dedigimiz objeler ile belirliyoruz.

`Ingress:` Genellikle HTTP olmak uzere bir clusterdaki servislere harici erisimi yoneten bir API nesnesidir. Yuk dengeleme, SSL sonlandirmasi ve path-name tabanli yonlendirme ozelliklerini destekler. 
Ingress objeleri bizim ingress controller i k8s objeleri araciligiyla ayarlamamizi sagliyor. L7 application load balancerimiz olan ingress controller a baglanarak www.example.com a gelen uygulamalari a uygulamasina www.example.com/contact a gelen uygulamari b uygulamasina ve www.sezginerdem.com adresine gelen uygulamari ise c uygulamasina yonlendir diye o uygulamanin menulerinin ozelliklerini kullanarak ayar yapmak yerine tum bu istekleri ingress objesi seklinde olusturuyoruz. Deploy ettigimiz zaman ingress controllerlar bu objeyi okuyarak gerekli duzenlemeyi otomatik olarak yapiyor. Ingress controller bizlere L7 ALB lerin sundugu ssl termination, path base routing gibi bir cok ozelligi k8s objeleri olarak deploy edebilmemizi sagliyor.

Iki ingress controller one cikmis durumda nginx ve traefik. Her ingressin kurulumu farklidir. Hepsinin kendi dokumantasyonundan bakmak gerekiyor. Nginx ingress kuruldugu zaman nginx kendine bir ns yaratir ve objelerini burada yaratir. 
Her ingress in kurulumu farkilidir. Hepsinin kurulumu kendi sayfalarindan nasil kurulacagini gorebilirsiniz. Nginx kurulduktan sonra kendisine bir ns yaratir ve daha sonra kurulacak uygulamalari bu ns uzerinde olusturur. 

Aws gibi servis saglayicilari uzerinde kullandigimiz eks kullandigimizi varsayalim. Bunun uzerinde bir uygulama deploy ettik ve uygulamayi da load balancer tipi bir uygulama ile expose ettik. Load balancer olusturunca AWS bizim icin bir load balancer yaratiyor ve buna public bir ip atiyordu ve bu ip adresine gelen tum istekleri bu servise yonlendiriyordu. Ben de bu ip adres ile bu servisin domainini DNS ile eslestirerek kullanicilarimin erismesini sagliyordum. Ayni k8s clusteri icerisinde ikinci bir uygulama daha deploy ettigimizi ve ayni sekilde load balancer tipi bir servis ile dis dunyaya actigimizi dusunun ayni surec isyelecek aws bir load balancer daha yaratacak bir tane daha public ip olacak 3. ve 4. de de aynisi olacak. Her bir load balancer icin ayri para odemem gerekiyor. Ikinci sorunda ise benim microservis mimarisinde bir uygulamam var. Bu uygulama da soyle calisiyor eger kullanicilar www.example.com adresiden gelirse a uygulamasi tarafindan sayfa sunuluyor fakat kullanici www.example.com/contact adresine giderse b uygulamasi tarafindan sayfa gosteriliyor bu nedenle mevcut load balancer ile benim bu ortami kurgulamam imkansiz. Cunku DNS de bu sekilde path base bir tanimlama yapamam. Bunun bir sekilde kullanici isteklerini anlamam ve buna gore url i anlama ve arkadaki uygulamaya ya da hangi servise gitmesi gerektigini bilmem gerekiyor yani beni klasik L4 tabanli bir load balancer  yerine L7 tabanli application layer inda calisan bir load balancer a ihtiyacim var. 
Bu iki sorun da ingress controller ve ingress objeleri ile cozulmektedir. 

Servisler nodeport tipinde olursa sadece cluster icinden erisim saglanir. Bu servislerin dis dunyadan erisim saglanmasi icin ingress controller yaratilmasi gerekmektedir. Ingress de ayni pod veya servis gibi bir k8s objesidir.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: appingress
  annotations: # servisinizi http degil de https olarak olusturmak istoyorsak bunu anotationslarda belirmek gerekiyor bunu da dokumantasyondan bakabilirsiniz
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: k8sfundamentals.com # domainine ait bir servis yayinladigimi soyluyor
      http:
        paths:
          - path: /blue # bu pathe bir istek gelirse
            pathType: Prefix
            backend:
              service:
                name: bluesvc # bu servisten cevap verilsin
                port:
                  number: 80
          - path: /green # bu pathe bir istek gelirse
            pathType: Prefix
            backend:
              service:
                name: greensvc # buradan cevap verilsin
                port:
                  number: 80
```

## ImagePullPolicy and Image Secret
Guvenlik nedeni ile image larinizi private repolarda tutmaniz gerekmektedir. Bu repolardan image cekmek icin authenticate olmak gerekiyor. 
Private registry lerde image lari tuttugunuz zaman bunlara authenticate olmak gerekir. Bu islemi yapmadan image cekilememektedir. 
Bu authenticate icin oncelikle bir secret olusturacagim. Bu secret tipi "docker-registry" olacak.

Private repodan cekilen bir image ile minikube olusturulan pod. Bunu cekmek icin authonticate olmak gerekiyor.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagesecrettest1
  labels:
    app: imagesecret
spec:
  containers:
  - name: imagesecretcontainer
    image: ozgurozturkregistry.azurecr.io/k8s:latest # bu bir private docker registry yani bu image i indirmek icin authenticate olmak gerekir
    ports:
    - containerPort: 80
  ```

Registrye authenticate olmak icin url, username ve password olmasi gerekiyor. Bunun icin bir secret olusturmamiz ve sonrasinda  bu secreti pod tanimina eklememiz gerekiyor.

// Authetication gerektiren bir Docker registry'den image çekebilmek için oluşturulması gereken secret'in oluşturulması. `docker-registry` tipinde bir secret tipi olusturdum
`kubectl create secret docker-registry "secret_ismi" --docker-server="registry_url" --docker-username="kullanıcı_adı" --docker-password="şifre"`
Ör: `kubectl create secret docker-registry regscrt --docker-server=ozgurozturkregistry.azurecr.io --docker-username=ozgurozturkregistry --docker-password=wqRjEDdVhrM9Hj4D=gWwvV3YXyq9Y4ID`

Daha sonra pod tanimimin icine `imagePullSecret` yazarak ve tanimladigim secret ismini alta yazarak authenticate islemini tamamliyorum. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagesecrettest2
  labels:
    app: imagesecret
spec:
  containers:
  - name: imagesecretcontainer
    image: ozgurozturkregistry.azurecr.io/k8s:latest #private repository
    imagePullPolicy: Always # IfNotPresent, Never
    # If the tag is latest, k8s defaults imagePullPolicy to Always. Otherwise k8s defaults imagePullPolicy to IfNotPresent
    ports:
    - containerPort: 80
  imagePullSecrets:
  - name: regscrt # az once olusturdugum secret ismini buraya yaziyorum
```

*2. Yol ise:* Image in cekilmesinde takip edilen 3 tane anahtar mevcut. Bu anahtarlari `imagePullPolicy` altinda tanimliyoruz. Eger bunun altinda `always` yazili ise her pod olusturulmaya calisildiginda her seferinde bu image cekilecek makinede olup olmadigi onemli degil. `never` secili olursa sadece localden cekilecek hicbir zaman repository den cekmeyecek. `IfNotPresent` da ise once locale bakacak yoksa repository e gidecek. Eger latest tagli bir image kullaniyorsaniz. `imagePullPolicy` `always` olarak set ediliyor. Fakat `latest` disinda bir sey kullaniliyorsa `imagePullPolicy` set edilmedi ise `IfNotPresent` olarak set edilir. 

## Static Pod
Kubelet in belirlediginiz bir dosyasina yaml dosyalarini koyarak kubeletin pod yaratma islemine `static pod` denir.
Boylelikle api serverla konusmadan yalnizca dosyalari klasore ekleyerek pod yaratma islemini gerceklestirmis oluruz.
*Neden static pod?* Kubernetes kendi de podlarlardan olusmaktadir. Kubernetes kurulumunda ilk once kubelet kurulur bunu da kubeletin default manifest dosyalarinin bulundugu klasorden yapar. Bu dosyanin bulundugu yere yaml dosyalarini koyarsak bu dosyalardan pod olusturulur. Daha sonra kubenetes api uzerinde de aynisini olusturur. Gercek hayatta cok isimize yaramiyor ancak bunu bilmek ve k8s in nasil ayaga kalktigini bilmek adina onemlidir.

etc/kubernetes/manifest dosyasi bu dosyanin default olarak bulundugu path. Ancak isterseniz degistirebilirsiniz.

## Helm
Bir uygualamin kurulusunun paket hale getirilmesini saglayan bir arac. CNCF projesi. Kubernetes uzerinde paket yuklenmesini sagliyor. Choco, brew gibi bir paket yoneticisidir. 
Kubernetesin resmi bir uygulamasi degil ancak nerede ise resmi uygulamasi haline gelmis durumda.
Paket veya script olarak yuklenebilir. Helmi yuklerken 2 ve 3 versiyonunu yukleyebilirsiniz. Yeni basliyorsaniz helm 3 le devam edin.
1. `Chart:` Kubernetese yuklediginiz uygulamanizin paketlenmis halidir. Paket hali chart.
2. `Release:` Chartin kubernetese yuklenmis hali ise releasedir. 
3. `Repository:` Helm chartlarin bir arada tutuldugu/bulundugu yerdir. 

`ArtifactHUB:` Kubernetesdeki pluginler, helm chartlari hepsi ayri repositorylerde idi. ArtifactHUB butun bu repositorylerin aranabildigi bir platform. Diyelim ki wordpress adinda bir helm charti aramak istiyorum bunu buradan bularak k8s e nasil kuracagimi ogrenebilirim. 

Helm uzerinden chartlari aramak istiyorsak `helm search` ile arama yapabiliriz. Ya da belirli bir repository den de arama yapabilirsiniz. `helm search repo wordpress`derseniz localimdeki helm reposundan arama yapar. Helm chartlari da yuklerken once reposunu ekliyorum ondan sonra install komutu ile yukluyorum. 

Chartin uzerinden bir release olusturmak istedigimizde default olarak kurar. Bazen degiskenleri kendimiz belirleyebiliz. Value lari kendim olsuturabilirim. 

Helm ile upgrade ler ya da begenmezsek rollback ler yapabiliyorum. Bu helmin en guclu yonlerinden bir tanesi. 

# OS Upgrade
// pod olmadiginda yeniden baslatmak eger deployment varsa options kullanmali-uunschedule eder ve tum podlari siler
`kubectl drain <node-name> --ignore-daemonsets`

// node-02 unschedule et ancak icindeki uygulamalara zarar verme - unschedule eder ama podlari silmez
`kubectl cordon <node-name>`

// node-01 i schedulable hale getirmek
`kubectl uncordon <node-name>`

## Etcd Backup - Resource Configs
// tum resourcelari yaml dosyasina kaydet
`kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml`

Velero=Kubernetes backup tool

// take a snopshot etcd
`ETCDTL_API=3 etcdctl snapshot save <snaphot.db>`

// snaphotlarin statusunu goruntuleme
`ETCDTL_API=3 etcdctl snapshot status <snaphot.db>`

// etcd snaphotlarini restore etmek
`service kube-apiserver stop`
`ETCDCTL_API=3 etcdctl snapshot restore <snapshot name> --data-dir </var/lib/etcd-from-backup(snaphot path)>`
//configure etcd.service file to use the new etcd file dir
`--data-dir=/var/lib/etcd-from-backup`
`systemctl daemon-reload`
`service etcd restart`
`service kube-apiserver start`

- etcd commands
  To see all the options for a specific sub-command, make use of the -h or --help flag.
  For example, if you want to take a snapshot of etcd, use:

  etcdctl snapshot save -h and keep a note of the mandatory global options.

  Since our ETCD database is TLS-Enabled, the following options are mandatory:

  --cacert   verify certificates of TLS-enabled secure servers using this CA bundle

  --cert     identify secure client using this TLS certificate file

  --endpoints=[127.0.0.1:2379]    This is the default as ETCD is running on master node and exposed on localhost 2379.

  --key                           identify secure client using this TLS key file


## Monitoring - Prometheus Stack
Kubernetes de 4 farkli yerde monitoring yapilmasi gerekmektedir:
1. Cluster in durumu. Hangi objeler var neler deploy edilmis
2. Objelerin bu durumu nedir. Deployment yarattik ama tum replicalar calsiiyor mu, podlar ne kadar cpu tuketiyor
3. Nodelari izlememiz gerekiyor. Worker nodelarin cpu, io kullanimi.
4. UYgulamalarin mevcut durumu. Uygulamalarin loglarini takip etmek gerekir.

Eger monitoringle ozellikle bir toolunuz yoksa. `k get pods` la kisa bir bilgi alabilirsiniz. Veya `describe` ile ayrintili bilgi alabilirim.
Metriclerin izlenmesini prometheus sayesinde yapabilrim. Sucunularimizdan gerekli end pointleri bildirerek metricleri toplama islemini prometheus yapar. Bunun bir ati ozelligi bir agent kurmaya ihtiyac duymamasi. Dagitik mimarilerde onemli bir ozellik. Kubernetes in nerdeyse standart monitoring araci haline geldi. 

### Prometheusun kubernetese kurulumu
Oncelikle `kube metrics` adli bir arac kurmaniz gerekiyor. Bu kube api ile konusarak objelerimizin durumunu metriclerini alarak bunlari kube api araciligiyla cekiyor ve prometheus un okuyabilcegi bir hale getiriyor. Datayi donusturuyor. Bunun yaninda nodeun cpu, memory gibi metcirleri linux makinasindan cekmek istiyorsak bu sefer nodelarin uzerine node exporteri kurmamiz gerekiyor. Linux un bu metricler icin endpointi yok bunun icin de bu metricleri push edecek `node exporter` in kurulmasi gerekiyor. Ayni zamanda prometheus kubernetes api ile de konusabiliyor. Bu sekilde de bir dizayn yapabilirsiniz. Daha sonra prometheus a gelen bilgilerin nasil store edilecegini de ayarlamamiz gerekiyor. Eger biz alert gondermek, email gondermek, sms gondermek istiryorsak. Bu kisimda alert de kurmamiz gerekiyor. Prometheus ceklein metriclerin gorsel halde sunulmasinda cok basarili degil. Bunun icin grafanayi kullaniyoruz. Grafana sadace prometheus icin degil herhangi bir data setinizi gorsellestirmek amaciyla kullanilabilmektedir. Prometheus un kurulum asamasindaki bu karmasikligi cozmek icin prometheus helm charti kullanilmaktadir. 

* kube-prometheus-stack kur

```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
$ helm install kubeprostack --namespace monitoring prometheus-community/kube-prometheus-stack
```

* Prometheus'u kontrol et

```
$ kubectl --namespace monitoring port-forward svc/kubeprostack-kube-promethe-prometheus 9090
```

queries:
Şu ana kadar oluşturulmuş tüm podlar: ```kube_pod_created```

Namespace'e göre dağılım ^^:```count by (namespace) (kube_pod_created)```

Mevcut çalışan podlar: ```sum by (namespace) (kube_pod_info)```

Ready durumunda olmayan podlar "namespace'e göre dağılım": ```sum by (namespace) (kube_pod_status_ready{condition="false"})```

* Alert manager kontrol et

```
kubectl --namespace monitoring port-forward svc/kubeprostack-kube-promethe-alertmanager 9093   
```

* Grafana'a bağlan

```
kubectl --namespace monitoring port-forward svc/kubeprostack-grafana 8080:80
```
username: admin

password: prom-operator

```
kubectl get secret kubeprostack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode ; echo  
```

Kubernetes de operator yontemi: Kubernetes de operatirler olusturabiliyoruz. Benim yani admin in yapacagi isi bu operatorlerle otomatize hale getiriyorum. Benim prometheus stack imi de ayni zamanda bu opearator yonetiyor benim adima. 

Prometheus default olarak 9090 portundan calisir. Kubernetes uzerinde dploy edilmis bir servis objesi uzerinden ulasilir. `kubectl --namespace monitoring port-forward svc/kubeprostack-kube-promethe-prometheus 9090` komutu ile port forward ediyorum ki prometheus a bagalabileyim. 

`kubectl --namespace monitoring port-forward svc/kubeprostack-grafana 8080:80` komutu ile de grafana ya baglanabilmek icin port forward yapiyorum. Burada gorsellestirmek icin pek cok hazir panal mevcut. Bu hazir panellerden monitirong yapabilirsiniz. Sifirdan panel kurmaya gerek yok. Ayrica grafana adresinden olusturulan dashboardlari alarak kullanabilirsiniz. Buradaki dashboardlarin id sini kendi grafanamiza yapistirarak upload eder ve kullanabilirsiniz. 

# Kubernetes de Loglama
Kubernetes de loglama icin kullanilan iki stack elesticsearch ve kibana. Elasticsearch loglarin search edilmesini saglayan bir tool iken kibana ise bu loglarin gorsellestirilmesini sagliyor. Aslinda elastic ile kubernetes deki metricleri de cekme imkanimiz oluyor ancak bunun icin cok da uygun bir tool degil. Loglari toplamak icin elastic stackin icinde iki tane urun var bunlardan bir tanesi logstation bu elastic in bir urunu. Logstation i deploy ediyorsunuz bu urun her worker node uzerindeki container engine ile haberleserek onun urettigi loglari toplayabiliyor. Diger bir alternatif de fluentd uygulmasi. Bunu yalnizca kubernetes veya elastic ile kullanmak zorunda degilsiniz baska toollarla birlikte de kullanabilirsiniz. Bunu bir deamonset olarak clusterimizdaki worker nodelarimiz uzerine bir daemon olacak sekilde yaratilacak ve containerlarimizin olusturdugu loglari bu fluentd toplayacak yani bir log agent olacak sekilde calisacak ve elasticsearch e gonderecek. Biz bu stack e `elasticsearch fluentd kibana stack` diyouruz. Bunun kurulum asamalari monitoring dosyasi altinda gorulebiliyor.