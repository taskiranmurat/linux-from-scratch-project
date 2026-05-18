# linux-from-scratch-project
Debian 12 üzerinde LFS 12.4 metodolojisi ile sıfırdan derlenmiş bağımsız Linux işletim sistemi inşası ve detaylı teknik dokümantasyonu.

<img width="661" height="498" alt="image5" src="https://github.com/user-attachments/assets/19fb708a-e3a3-46c1-87e0-f3cd44f9acc9" />

## Bölüm 1
##  Amaç ve Kapsam

Bu projede, VMware sanallaştırma platformu üzerinde çalışan **Debian 12** ana sistemi (Host) kullanılarak, **Linux From Scratch (LFS v12.4)** metodolojisi ile bağımsız bir Linux dağıtımının sıfırdan inşa edilmesi hedeflenmiştir. 

Projenin temel amacı; hazır bir Linux dağıtımı kullanmak yerine, işletim sistemini kaynak kodlarından manuel olarak derleyerek Linux çekirdeğinin ve sistem mimarisinin çalışma mantığını derinlemesine deneyimlemektir. 

**Proje kapsamında gerçekleştirilen temel aşamalar:**
* Sistem inşası için gerekli **Toolchain** (derleme araç takımı) ortamının hazırlanması,
* Temel sistem paketlerinin ve kütüphanelerin kaynak koddan manuel olarak derlenmesi,
* **Kernel (Çekirdek) konfigürasyonu** ve sistemin kendi başına  boot edilebilir hale getirilmesi.

Bu çalışma sonucunda; Linux dosya sistemi hiyerarşisi, bağımlılık yönetimi, derleme süreçleri (SBU yönetimi) ve Linux sistem mimarisi derinlemesine analiz edilerek dökümante edilmiştir.

##  Yöntem ve Teknik Özellikler

Bu çalışma, sanallaştırma platformu olarak **VMware** kullanılarak, ana işletim sistemi (Host OS) üzerinde ayrılmış bağımsız bir disk bölümünde (`/dev/sdb`) gerçekleştirilmiştir. 

Sistem inşası sırasında **LFS 12.4** standartları ve dökümantasyonu birebir takip edilmiş, tüm temel paketler kaynak kodlarından optimize edilerek derlenmiştir. Derleme sürecinin ikinci aşamasında, host sistemden tamamen izole bir yapı elde etmek amacıyla **chroot (change root)** ortamına geçiş yapılmış ve nihai işletim sistemi bu izole sandbox üzerinde adım adım inşa edilmiştir.

### Sistem Mimarisi ve Bileşen Tablosu

| Bileşen / Özellik | Kullanılan Teknoloji / Versiyon |
| :--- | :--- |
| **Host İşletim Sistemi** | Debian 12 (Bookworm) |
| **Sanallaştırma Ortamı** | VMware Workstation / ESXi |
| **LFS Versiyonu** | LFS v12.4 |
| **Hedef Çekirdek (Kernel)** | Linux Kernel v6.16.1 |
| **Dosya Sistemi Yapısı** | Ext4 ) |
| **Bootloader** | GRUB v2.12 |
| **Sistem Mimarisi** | x86_64 (64-bit) |
| **Hedef Disk Bölümü** | `/dev/sdb`  |


##  LFS Sistem İnşa Süreci ve Yol Haritası

Linux From Scratch (LFS) sistemi, resmi dökümantasyonda yer alan 11 ana bölümün mantıksal bir sıra ile takip edilmesiyle inşa edilmiştir. Bu uzun soluklu derleme süreci temel olarak **4 ana faza** ayrılmaktadır:

### Faz 1: Ortamın Hazırlanması ve Kaynak Kodların Temini (Bölüm 2 - 4)
* **Bölüm 2 - Hazırlık Aşaması ve Host Sistem Gereksinimleri:** Yeni bir Linux disk bölümü (partition) oluşturulmuş ve `Ext4` dosya sistemi LFS için hazırlanmıştır.
* **Bölüm 3 - Paket ve Yamaların İndirilmesi:** Sistemde derlenecek tüm açık kaynak kodlu paketler (`.tar.xz`, `.tar.gz`) ve stabilite yamaları (patches) host sisteme indirilmiştir.
* **Bölüm 4 - Çalışma Ortamı Yapılandırması:** LFS kullanıcısı tanımlanmış, ortam değişkenleri (`$LFS`, `$LFS_TGT`) ayarlanmış ve derleme için gerekli dizin hiyerarşisi kurulmuştur.

### Faz 2: Geçici Toolchain İnşası - Cross-Compilation (Bölüm 5 - 6)
* **Bölüm 5 - İlk Derleme Araçları (Toolchain):** Host sistemin kütüphanelerinden bağımsızlaşmak adına `Binutils`, `GCC` ve `Glibc` gibi temel derleme bileşenleri **Cross-Compilation (Çapraz Derleme)** yöntemiyle inşa edilmiştir.
* **Bölüm 6 - Temel Araçların Derlenmesi:** İlk oluşturulan toolchain kullanılarak, bir sonraki aşamada sistemi ayağa kaldıracak geçici yardımcı programlar derlenmiştir.

### Faz 3: İzole Ortama Geçiş ve Nihai Sistem İnşası (Bölüm 7 - 8)
* **Bölüm 7 - Chroot ve Sanal Dosya Sistemleri:** Host sistemin `/dev`, `/proc`, `/sys` gibi sanal dosya sistemleri LFS diskine bağlanmış ve `chroot (change root)` komutu ile host sistemden tamamen izole, bağımsız bir "Sandbox" ortamına geçilmiştir.
* **Bölüm 8 - Temel Sistem Yazılımlarının Derlenmesi:** Chroot ortamının sağladığı izole avantajla, sistemin nihai paketleri  kendi kütüphaneleriyle sıfırdan derlenmiştir. 

### Faz 4: Sistem Yapılandırması ve Boot Süreci (Bölüm 9 - 11)
* **Bölüm 9 - Sistem Konfigürasyonu:** Ağ ayarları (Network configuration), `fstab` (disk bağlama tablosu), saat dilimi ve yerel dil (locale) ayarları yapılmıştır.
* **Bölüm 10 - Kernel ve Bootloader Kurulumu:** İşletim sisteminin kalbi olan **Linux Çekirdeği (Kernel 6.16.1)** donanıma uygun şekilde konfigüre edilerek derlenmiş ve `GRUB 2.12` bootloader kurulumu tamamlanmıştır.
* **Bölüm 11 - Kapanış:** Kurulum başarıyla tamamlanarak sistem ilk kez bağımsız (standalone) olarak boot edilmeye hazır hale getirilmiştir.


## Bölüm 2: Hazırlık Aşaması ve Host Sistem Gereksinimleri

Bu bölümde, LFS sisteminin kurulumu için gerekli olan host araçlarının uygunluğu kontrol edilmekte ve eksik olması durumunda gerekli kurulumlar yapılmaktadır. Ardından, LFS sisteminin kurulacağı disk bölümü hazırlanır. Bu süreçte yeni bir disk bölümü oluşturulur, bu bölüm üzerinde bir dosya sistemi kurulur ve sistem kullanımı için mount işlemi gerçekleştirilir.

### 2.1 Host Sistem Uyumluluk Kontrolü

LFS sisteminin kurulabilmesi için kullanılan host sistemin belirli yazılım gereksinimlerini karşılaması gerekmektedir. Bu gereksinimler, sistemin sorunsuz bir şekilde derleme işlemini gerçekleştirebilmesi için minimum versiyonları belirtilmiş araçlardan oluşmaktadır. LFS 12.4 kitabında belirtilen minimum yazılım versiyonlarının (Bash, GCC, Glibc, Binutils, Coreutils vb.) host sistemde mevcut olduğu kitapla birlikte gelen `version-check.sh` betiği ile teyit edilmiştir.

### 2.2 Partition Oluşturma 

LFS sisteminin host sistemden tamamen bağımsız bir alanda barındırılması için VMware üzerinde 65 GB'lık yeni bir sanal disk `/dev/sdb` oluşturuldu ve şu işlemler uygulanmıştır:

* **Disk Bölümlendirme:** `fdisk` arayüzü kullanılarak disk bölümlenmiştir.
* **Dosya Sistemi:** LFS kök dizini için `Ext4` dosya sistemi ile formatlanmıştır.
* **Mount İşlemi:** Oluşturulan bölüm host sistem üzerinde `/mnt/lfs` dizinine bağlanarak çalışma alanı tanımlanmıştır.

`/dev/sdb` boş diskimdir. root’ta `fdisk /dev/sdb` komutunu enterlayıp gireriz.


<img width="928" height="298" alt="image6" src="https://github.com/user-attachments/assets/0c7c52ed-e45c-42ea-a19b-84fd04db727f" />

500MB boot için ayırdık 1GB swap için ayırdık geri kalan 63.5 GB root dizin için ayırdık. 

Bu bölümlere dosya sistemleri aşağıdaki komutlarla oluşturuluyor.

```bash
mkfs -v -t ext4 /dev/sdb1
mkfs -v -t ext4 /dev/sdb2
mkswap /dev/sdb3

export LFS=/mnt/lfs
umask 022
```
Burada export LFS=/mnt/lfs LFS kurulacağı dizini tanımlar. Yani tüm işlemler bu klasör altında yapılır. umask 022 ise oluşturulan dosyaların izinlerini ayarlar.

Şimdi mount etme zamanı.
```bash
mkdir -pv $LFS
mount -v -t ext4 /dev/sdb2 $LFS

mkdir -v $LFS/boot
mount -v -t ext4 /dev/sdb1 $LFS/boot
/sbin/swapon -v -p1 /dev/sdb3
```

## Bölüm 3: Paketler ve Yamalar (Patches)
Paketler için bir konum belirleyeceğiz. LFS’nin kurulacağı bölüm içinde bir sources dizini oluşturuyoruz.
```bash
mkdir -v $LFS/sources
```
```bash
chmod -v a+wt $LFS/sources
```
Sources klasörünün içine bu komutu yazdığımız da  bu dizinin izinlerini kalıcı hale getiririr.

```bash
wget [https://www.linuxfromscratch.org/lfs/view/stable/wget-list-sysv](https://www.linuxfromscratch.org/lfs/view/stable/wget-list-sysv)
wget --input-file=wget-list-sysv --continue --directory-prefix=$LFS/sources
```

Kaynak kod listesini wget ile çekiyoruz ve ardından tüm paketleri /mnt/lfs/sources klasörüne indiriyoruz.
```bash
wget [https://www.linuxfromscratch.org/lfs/view/stable/md5sums](https://www.linuxfromscratch.org/lfs/view/stable/md5sums)
pushd $LFS/sources
md5sum -c md5sums
popd
```
İndirilen dosyaların eksiksiz ve güvenli olduğunu doğrulamak için md5sums kontrolünü gerçekleştiriyoruz.
```bash
chown root:root $LFS/sources/*
```
Son olarak chown komutu ile sources klasörünün içindeki tüm kaynak dosyaların sahipliğini root kullanıcısına devrediyoruz.

## Bölüm 4: Son Hazırlıklar

Bu bölümde geçici sistemin kurulumu ve hazırlığı için bazı ek işlemler gerçekleştirilecektir. `$LFS` dizini altında geçici araçların kurulacağı bir dizi dizin oluşturulacak, ayrıcalıksız bir kullanıcı eklenecek ve bu kullanıcı için uygun bir derleme ortamı hazırlanacaktır.

### 4.1 Dizin Oluşturma

LFS sisteminin inşası sırasında derlenecek geçici araçların (tools) ve sistem konfigürasyonlarının barındırılacağı dizin yapısı aşağıdaki komutlar ile kök dizin altında oluşturulmuştur:

<img width="944" height="488" alt="image16" src="https://github.com/user-attachments/assets/bd018114-b38a-4df3-8ad8-12c62c6bb360" />

<img width="777" height="295" alt="image3" src="https://github.com/user-attachments/assets/b1b3a316-beaa-4cbe-89ec-132f076f3443" />

Dizinler bu şekilde oluşturuldu.

### 4.2 LFS Kullanıcısı Ekleme

Sisteme `root` kullanıcısı olarak giriş yapıldığında, tek bir hata bile host sisteme zarar verebilir veya sistemi tamamen yok edebilir. Bu nedenle, sonraki bölümlerde yer alan paketler ayrıcalıksız  bir kullanıcıyla inşa edilecektir. Kurulum sürecinde komutları çalıştırmak üzere “lfs”  adında yeni bir grup ve bu gruba üye “lfs” adında yeni bir kullanıcı oluşturacağız. Aşağıdaki komutlarla oluşturulur.
```bash
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs

passwd lfs
```
lfs user ve passwordü oluşturuldu. Ardından bu kullanıcıya lfs altındaki tüm dizinlere tam erişim vereceğiz  aşagıdaki komutlarlarla.
```bash
chown -v lfs $LFS/{usr{,/*},var,etc,tools}
case $(uname -m) in x86_64)
 chown -v lfs $LFS/lib64 ;;
 esac 
```
<img width="905" height="463" alt="image12" src="https://github.com/user-attachments/assets/b4e6b625-1537-42cf-990c-3e60495fd07f" />

Bu şekilde oluşturuldu. 

### 4.3 Ortam Ayarlama

LFS kullanıcısı için temiz bir çalışma ortamı oluşturmak amacıyla Bash kabuğuna ait başlangıç dosyalarını hazırlıyoruz.
```bash
cat > ~/.bash_profile << "EOF" exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash EOF 

cat > ~/.bashrc << "EOF"
 set +h
 umask 022 
LFS=/mnt/lfs LC_ALL=POSIX 
LFS_TGT=$(uname -m)-lfs-linux-gnu 
PATH=/usr/bin 
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
 PATH=$LFS/tools/bin:$PATH CONFIG_SITE=$LFS/usr/share/config.site 
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
 EOF
```
“exec env -i ... /bin/bash” komutu ile host sistemden gelen tüm ortam değişkenleri  temizlenmiş böylece LFS derleme sürecine host sistemin kütüphanelerinin sızması tamamen engellenmiştir. Bu, LFS'nin "saf" ve bağımsız olmasını sağlayan en kritik güvenlik adımıdır.

Çekirdek sayısı paketlerin derleme hızını artırır o yüzden sistemdeki çekirdek sayısına göre bash te ayarlamalıyız.
```bash
export MAKEFLAGS=-j8
cat >> ~/.bashrc << "EOF" 
export MAKEFLAGS=-j$(nproc)
 EOF 

source ~/.bash_profile
```
Benim 8 çekirdek olduğu için ona göre ayarladım.

---

# Teorik Altyapı: LFS Çapraz Araç Zinciri ve Geçici Araçların Mantığı

Burada LFS çapraz araç zincirini ve 8.bölüme geçmeden önce bazı geçici araçları oluşturacağız. 8. Bölüm de ise LFS’yi inşa edeceğiz. Dolayısıyla 5,6 ve 7. bölümler de inşa ettiğimiz her şey 8.Bölümü inşa etmek için geçici bir kullanım içindir. 8. Bölümdeki her şey sıfırdan oluşturulan nihai linux sistemini oluşturan unsurlardır. Bu yüzden 5,6 ve 7.Bölümler çok önemlidir. 

Bölümün amacı, host sistemden tamamen izole edilmiş geçici bir derleme ortamı oluşturmaktır. Bu ortamda kullanılan derleyici ve araçlar, host işletim sistemine bağımlı olmayacak şekilde hazırlanır. Böylece oluşturulacak LFS sistemi temiz, kararlı ve tekrar üretilebilir olur.


---

### Cross-Compilation (Çapraz Derleme) Nedir?

**Cross-compilation**, bir sistem üzerinde çalışırken başka bir sistem (veya mimari) için çalışacak yazılımların derlenmesi işlemidir. LFS’de bu yöntem, host sistem kütüphanelerine yanlışlıkla bağımlılık oluşmasını engellemek ve izolasyonu sağlamak için zorunlu bir yöntemdir.

### Build / Host / Target Kavramları

LFS mimarisinde bu üç kavram şu şekilde konumlandırılmıştır:

* **Build:** Derlemenin fiilen yapıldığı mevcut sistemdir. Bizim senaryomuzda **Build = Debian 12 (VMware üzerindeki host sistemimiz)**. Tüm paketler ilk olarak burada derlenir.
* **Host:** Oluşturulan yazılımın üzerinde çalışacağı sistemdir. Bizim senaryomuzda **Host = LFS (henüz kurulmamış, `/mnt/lfs` altında hazırlanan sistem)**. Dosyaları şu an diskimizdedir ve sürecin sonunda boot edeceğimiz nihai sistemdir.
* **Target:** Derleyicinin (Compiler) kendisi için kod ürettiği hedef mimaridir. Bizim senaryomuzda **Target = LFS'in mimarisi (x86_64-lfs-linux-gnu)**. Derleyici Debian için değil, tamamen LFS için binary üretir.

---

### 3 Aşamalı Derleme Mantığı (The Three-Stage Bootstrapping)

Derleyici (Compiler) ve standart kütüphaneler (Glibc) birbirine göbekten bağımlıdır. Bu döngüsel bağımlılığı kırmak için LFS şu 3 aşamalı mantığı kullanır:

#### STAGE 1 – Cross-Compiler (Kitap Bölüm 5)

En temeldir.Debian üzerinde, LFS için çalışan bir derleyici oluşturuyor. Debian’ın gcc’si,binutils’i kullanılıyor  ve **“$LFS/tools/bin/x86_64-lfs-linux-gnu-gcc”** üretiliyor. Bu derleyici Debian’da çalışır ama LFS için kod üretir. 
* *Amaç:* Debian kütüphanelerine hiç bulaşmadan LFS için temiz ve bağımsız bir başlangıç yapmaktır.

#### STAGE 2 – Geçici LFS Compiler (Kitap Bölüm 6)
Stage 1’de ürettiğim cross compiler’ı kullanarak. LFS için çalışan ama hâlâ Debian üzerinde derlenen bir compiler yapıyorum. Yani Debian’da çalışıyor ama LFS ortamına kuruluyor. Kurulum yeri /mnt/lfs/tools.
* *Amaç:* Host sistemin (Debian) kendi `gcc` derleyicisine olan güven bağımlılığını tamamen kesmek ve kendi ürettiğimiz güvenli `gcc` ile yola devam etmektir.

#### STAGE 3 – Native Compiler (Kitap Bölüm 7 ve 8)
Artık host sistemle bağlar koparılır ve `chroot /mnt/lfs` komutuyla oluşturulan yeni ortama girilir. Bu aşamada artık Debian tamamen devre dışıdır; yeni kök dizin (`/`) LFS'in kendisidir. Bu izole ortamda `gcc`, `binutils` ve `glibc` tamamen yeniden derlenir. 
* *Sonuç:* Derleyici artık hem LFS üzerinde çalışır hem de LFS için yerel (native) kod üretir.

Compiler ve standart kütüphaneler birbirine bağımlıdır. Bu 3 aşama ile önce sınırlı bir cross compiler oluşturulur sonra bu compiler ile LFS ortamı izole edilir ve en sonunda native compiler yeniden derlenir.

---

## Bölüm 5: Compiling a Cross-Toolchain

Bu bölüm bir çapraz derleyicinin (cross-compiler) ve onunla ilgili araçların nasıl inşa edileceğini göstermektedir. Burada çapraz derleme işlemi "sahte" bir şekilde gerçekleştirilse de prensipler gerçek bir çapraz araç zinciriyle (cross-toolchain) aynıdır.

Bu bölümde derlenen programlar, sonraki bölümlerde kurulacak dosyalardan ayrı tutulması amacıyla `$LFS/tools` dizini altına kurulacaktır. Öte yandan kütüphaneler inşa etmek istediğimiz sistemin kendisine ait oldukları için nihai yerlerine kurulurlar.

Yapım süreci önce `/mnt/lfs/sources` dizininde olmalıyız. Daha sonra her paketten verileri çıkarıyoruz. Az önce çıkarılan dizine giriyoruz kitaptaki talimatları izleyerek paketi oluşturuyoruz. `/mnt/lfs/sources` dizinine geri dönerek her şeyi derledikten sonra o paketi siliyoruz. 

Önce Binutils 2.45’i kuruyoruz. `tar -xvf` ile açıyoruz.
<img width="634" height="34" alt="image13" src="https://github.com/user-attachments/assets/9d6740c0-bc83-4d64-9702-b5ae276a171d" />

Çıkardığımız dizine geçiyoruz. Kitaptaki talimatları takip ediyoruz. 

<img width="1032" height="195" alt="image25" src="https://github.com/user-attachments/assets/e493baa3-ea83-47f1-8ff5-64b6ed5697ca" />

Bu şekilde talimatları izleyip kuruyoruz. Yapmamız gereken son şey /sources dizinine geri dönüp çıkarılan paketi yani binutils paketini silmek. 

<img width="531" height="48" alt="image19" src="https://github.com/user-attachments/assets/d8774be2-a941-46a0-9682-bfb4ebeb75fe" />

Bu bölümde geriye kalan 4 paketi de aynı şekilde yapacağız. Yani gcc,linux API header,glibc ve Libstdc++ paketlerini. 


## Bölüm 6: Cross Compiling Temporary Tools

Bu bölüm yeni inşa edilen çapraz araç zincirini (cross-toolchain) kullanarak temel yardımcı programların nasıl çapraz derleneceğini göstermektedir. Bu yardımcı programlar nihai konumlarına kurulur ancak henüz kullanılamazlar. Temel görevler hala ev sahibi sistemin (Debian) araçlarına dayanmaktadır. Bununla birlikte kurulan kütüphaneler linking işlemi sırasında kullanılır.

Yardımcı programların kullanımı bir sonraki bölümde "chroot" ortamına girildikten sonra mümkün olacaktır. Ancak bunu yapmadan önce bu bölümde inşa edilen tüm paketlerin tamamlanması gerekir. Bu nedenle henüz ev sahibi sistemden bağımsız olunamaz.

Temporary Tools geçici sistemdir. Bu bölümün ana fikri LFS sisteminin kendi kendini derleyebilecek seviyeye gelmesidir. Debian çalışıyor LFS `/mnt/lfs` altında chroot henüz yok.

Bu bölümde LFS sistemi için gerekli olan temel kullanıcı araçları ve derleme araçları geçici olarak oluşturulmuştur. Bu araçlar, host sistemden (Debian) bağımsız bir ortam hazırlamak ve bir sonraki aşamada chroot ortamına geçişi mümkün kılmak amacıyla derlenmiştir.

* *Bu bölümde derlenen paketlerin amacı* tam özellikli bir sistem kurmak değil, LFS sisteminin kendi kendini inşa edebileceği minimum bir kullanıcı alanı oluşturmaktır.

Bu bölümde aşağıdaki paketleri kurup kitaba göre yapılandıracağız. Paketler gruplayarak anlatılmıştır;








































