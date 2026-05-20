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
| **Sanallaştırma Ortamı** | VMware Workstation |
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

### 6.1 Paketler

* **Metin işleme ve temel Unix araçları (M4, Sed, Grep, Gawk, Diffutils)**
Metin işleme araçları (sed, grep, gawk, diffutils) derlenerek betiklerin, configure script’lerin ve build sistemlerinin düzgün çalışması sağlanmıştır. Bu araçlar kaynak kodların işlenmesi ve yamaların uygulanması için kritik öneme sahiptir.

* **Dosya ve arşiv yönetimi araçları (Tar, Xz, Gzip, File)**
Arşiv ve dosya tanıma araçları kurularak kaynak paketlerin açılması, sıkıştırılması ve dosya tiplerinin doğru şekilde algılanması sağlanmıştır.

* **Temel sistem komutları (Coreutils, Findutils)**
Coreutils ve Findutils paketleri ile ls, cp, mv, chmod, find gibi temel sistem komutları LFS ortamına kazandırılmıştır. Bu araçlar olmadan bir Linux sisteminin kullanılabilir olması mümkün değildir.

* **Kabuk ve terminal altyapısı (Bash, Ncurses)**
Bash kabuğu ve Ncurses kütüphanesi derlenerek LFS sisteminde komut satırı etkileşimi sağlanmıştır. Ncurses terminal tabanlı uygulamaların doğru şekilde çalışması için gereklidir.

* **Derleme araçları (Make, Patch)**
Make ve Patch araçları, yazılım paketlerinin derlenmesi ve kaynak kodlara yamaların uygulanabilmesi için kurulmuştur. Bu araçlar LFS sisteminin kendi yazılımlarını derleyebilmesi açısından zorunludur.

* **Binutils – Pass 2**
 Bu aşamada Binutils ikinci kez derlenerek LFS sistemine daha uyumlu bir linker ve assembler elde edilmiştir.

* **GCC – Pass 2**
GCC Pass 2, LFS sisteminin kendi kendini derleyebilmesinin temelini oluşturur.

Bu paketleri önceki bölümde olduğu gibi teker teker tar ile çıkarılıp daha sonra kitaptaki talimatlara göre kurulmuştur.

Chapter 6’da LFS sistemi için gerekli olan tüm temel kullanıcı araçları ve derleyiciler geçici olarak oluşturulmuş, sistem chroot ortamına girmeye hazır hale getirilmiştir.

## Bölüm 7: Chroot Ortamına Giriş ve Geçici Araçların İnşası

Bu bölümde sistemden (Host - Debian) tamamen izole edilmiş, kendi içine kapalı bir ortam oluşturulur. Artık paketleri ana sistemin kütüphaneleriyle değil, bölüm 6'da oluşturduğumuz temiz araçlarla derleyeceğiz.

#### Bu Bölümde Yapılacaklar:
* Changing Ownership
* Preparing Virtual Kernel File Systems
* Entering the Chroot Environment
* Creating Directories
* Creating Essential Files and Symlinks
* Ek Paketlerin Derlenmesi
* Cleaning Up and Saving the Temporary System

---

### Changing Ownership
Şu anda, `$LFS` altındaki tüm dizin hiyerarşisinin sahibi, yalnızca ev sahibi (host) sistemde mevcut olan `lfs` kullanıcısıdır. Eğer `$LFS` altındaki dizinler ve dosyalar olduğu gibi bırakılırsa, karşılık gelen bir hesabı bulunmayan bir kullanıcı kimliğine (UID) ait olacaklardır. Bu durum tehlikelidir; çünkü daha sonra oluşturulacak bir kullanıcı hesabı bu aynı kullanıcı kimliğini (UID) alabilir ve `$LFS` altındaki tüm dosyaların sahibi haline gelebilir. Bu da söz konusu dosyaları olası kötü niyetli müdahalelere açık hale getirir.

Bu aşamada LFS dizin ağacının sahipliği `root` kullanıcısına devredilmiştir. Önceki bölümlerde derleme işlemleri `lfs` kullanıcısı ile yapılırken, chroot ortamına geçiş öncesinde sistem dosyalarının `root` tarafından yönetilmesi gerekmektedir.

### Preparing Virtual Kernel File Systems
Çekirdek (Kernel) ile iletişim kurabilmek için `/dev`, `/proc`, `/sys` ve `/run` gibi sanal dosya sistemleri ana sistemden LFS alanına bağlanır. Bu chroot içindeki sistemin donanım ve süreç bilgilerine erişebilmesini sağlar.

<img width="551" height="136" alt="image21" src="https://github.com/user-attachments/assets/44f6545e-1ee7-4570-a42d-0a341fd83b77" />

Klasörler oluşturuldu sonraki işlem mount etmektir.

### Entering the Chroot Environment
`chroot` komutu ile sistem kök dizini `/mnt/lfs` olacak şekilde yeni bir çalışma ortamına girilmiştir. Bu noktadan itibaren çalıştırılan tüm komutlar LFS sistemine aitmiş gibi davranır. Bu andan itibaren çalıştırılan komutlar Debian sistemine zarar veremez. Artık tamamen LFS'in kendi içindeyiz.

<img width="594" height="211" alt="image4" src="https://github.com/user-attachments/assets/de2ed6ca-7071-42c6-a024-e922b857fc2e" />


### Creating Directories
LFS sistemi için gerekli olan temel dizin yapısı (/bin, /lib, /usr, /etc vb.) oluşturulmuştur. Bu dizinler, standart bir Linux sisteminde bulunması gereken hiyerarşik dizin yapısı.

<img width="833" height="587" alt="image18" src="https://github.com/user-attachments/assets/f212c712-ef7e-44ff-8075-2e90f2f91f5e" />

<img width="927" height="831" alt="image1" src="https://github.com/user-attachments/assets/a0f49472-d824-4a5c-8af9-4828276d4bf0" />

<img width="863" height="300" alt="image22" src="https://github.com/user-attachments/assets/9027c5d7-81bf-44f3-952d-5ffcba769380" />

Dizinler oluşturuldu.

### Creating Essential Files and Symlinks
Sistem çalışması için gerekli olan temel yapılandırma dosyaları ve sembolik linkler oluşturulmuştur. Özellikle `/bin`, `/lib` ve `/usr` dizinleri arasındaki bağlantılar sistemin tutarlı çalışması için önemlidir.

### Ek Geçici Araçların Derlenmesi (Gettext, Python, Perl vb.)
Bu bölümdeki paketler nihai sistemi kurarken kullanılacak olan yardımcı araçlardır. Ana sistemdeki versiyonlarla uyumsuzluk yaşamamak için LFS'nin kendi temiz kütüphanelerini kullanan bu araçlar chroot içinde derlenir.

### Cleaning Up and Saving the Temporary System
Bu aşamada geçici dosyalar temizlenmiş ve sistem bir sonraki bölüme hazır hale getirilmiştir. Amaç mümkün olan en sade ve kontrollü bir temel sistem bırakmaktır.

## Bölüm 8: Temel Sistem Paketleri ve İşlevleri

LFS sisteminin gerçek anlamda oluşturulduğu bölümdür. Hepsi sırayla açıklanmıştır ve bu sırayla chroot içinde derlenmiştir. Sırayla gidilmelidir çünkü birbirlerine bağımlılıkları vardır.

* **Man-pages-6.15:** Sistem komutlarının kullanım kılavuzlarını ve dökümantasyonunu sağlar. 2400’den fazla man sayfası içermektedir.
* **Iana-Etc-20250807:** Network servisleri ve protokolleri için veri sağlar.
* **Glibc-2.42:** Glibc paketi ana C kütüphanesini içerir. Bu kütüphane bellek ayırma, dizin arama, dosya açma ve kapatma, dosya okuma ve yazma, dize işleme, desen eşleştirme, aritmetik işlemler ve benzeri temel rutinleri sağlar.
* **Zlib-Bzip2-Xz-Lz4-Zstd:** Bu paketler dosyaları sıkıştırmak ve açmak için kullanılan programları içerir. LZ4 ve Zstd gerçek zamanlı hız gerektiren (kernel boot, loglama) işlemlerde tercih edilirken; Xz ve Bzip2, kaynak kod paketlerinin depolama alanından tasarruf etmesi amacıyla kullanılır. Zlib ise sistem genelinde en yaygın kullanılan standart uyumluluk katmanıdır.

* **File-5.46:** File paketi verilen bir dosyanın veya dosyaların türünü belirlemek için kullanılan bir yardımcı program içerir.
* **Readline-8.3:** Komut satırında düzenleme ve history özelliklerini sağlar.
* **M4-1.4.20:** Yazılım yapılandırma ve kod üretim süreçlerinin temel makro motorudur.
* **Bc-7.0.3:** Bu paket sayısal işlem yapabilen bir programlama dili içerir.
* **DejaGNU-1.6.3:** DejaGnu paketi GNU araçları üzerinde test paketlerini çalıştırmak için bir çerçeve içerir.
* **Pkgconf-2.5.1:** Derleme sırasında kütüphane bağımlılıklarını ve yollarını yönetir.
* **Binutils-2.45:** Kaynak kodun makine diline çevrilmesi (link/assemble) için gereken araç setidir.
* **GMP-MPFR-MPC:** Bu 3 paket GCC derleyicisi için gerekli olan yüksek hassasiyetli matematiksel hesaplama ve karmaşık sayı aritmetiği kütüphaneleridir.
* **Attr, ACL, Libcap:** Bu paketler standart Linux izin sistemini modern güvenlik ihtiyaçlarına göre genişletir. Attr ve ACL ile dosya bazlı çok detaylı erişim kısıtlamaları getirilirken, Libcap ile “root” yetkileri parçalanarak süreç bazlı güvenlik sağlanmıştır. Bir programın tüm sisteme hükmetmek yerine, sadece ihtiyacı olan özel yetkiyi kullanmasına izin vererek sistem güvenliğini artırır.
* **Libxcrypt-4.4.38:** Parolaların tek yönlü karma algoritmasıyla işlenmesi için modern bir kütüphane içermektedir.
* **Shadow-4.18.0:** Kullanıcı ve grup hesaplarını güvenli bir şekilde yönetmek için kullanılan bir araç setidir. "Shadow" ismi, kullanıcı şifrelerinin herkesin okuyabildiği `/etc/passwd` dosyası yerine, sadece root kullanıcısının erişebildiği `/etc/shadow` dosyasında gizlenmesi (shadowing) tekniğinden gelir.
  * `useradd` / `usermod` / `userdel`: Kullanıcı hesabı oluşturma, düzenleme ve silme işlemleri.
  * `passwd`: Kullanıcı şifrelerini değiştirme ve hash algoritmalarını uygulama.
  * `groupadd` / `groupmod`: Kullanıcı gruplarını yönetme.
  * `login`: Kullanıcı kimlik doğrulamasını yaparak oturum açma sürecini yönetme.
  * `chage`: Şifre geçerlilik süresi ve hesap kilitleme gibi "eskime" politikalarını belirleme.
* **GCC-15.2.0:** C, C++ ve diğer dillerdeki kaynak kodlarını, bilgisayarın işlemcisinin anlayabileceği binary dönüştüren ana derleyicidir.

* **Ncurses-6.5-20250809:** Terminal ekranında metin tabanlı kullanıcı arayüzleri oluşturulmasını sağlar. Karakterlerin ekranda belirli koordinatlara yerleştirilmesine, renklerin yönetilmesine ve pencerelerin çizilmesine olanak tanır. System konfigürasyonu yaparken mavi gri çizgili ekranı ncurses yapar.
* **Sed-4.9:** Bir metin akışı veya dosya üzerinde; arama, bulma, değiştirme, ekleme ve silme gibi işlemleri dosyayı interaktif bir editörle Vim gibi açmaya gerek kalmadan gerçekleştirir.
* **Psmisc-23.7:** Sistemdeki süreçleri görüntülemek, analiz etmek ve yönetmek için özelleşmiş araçlar sunar.
  * `pstree`: Çalışan süreçleri birbirleriyle olan bağlarına göre bir "aile ağacı" şeklinde görselleştirir. Hangi sürecin hangisi tarafından başlatıldığını görmeyi sağlar.
  * `killall`: Bir süreci ismiyle sonlandırır.
* **Gettext-0.26:** Yazılımların mesajlarını, hata çıktılarını ve menülerini kullanıcının diline çevirmek için kullanılan standart kütüphane ve araç setidir.
* **Bison-3.8.2:** Karmaşık yazılımların ve dillerin gramer yapısını çözümleyen bir ayrıştırıcı oluşturucudur. Bu paket özellikle derleme süreçlerinde kaynak kodun mantıksal analizinin yapılabilmesi için temel bir altyapı sunmaktadır.
* **Grep-3.12:** Dosyaların içeriğinde arama yapmak için kullanılan programlar içerir.
* **Bash-5.3:** Komutları yorumlayan, işleyen ve kernele ileten ana komut satırı kabuğudur Shell.
* **Libtool-2.5.4:** Paylaşımlı (shared) ve statik (static) kütüphanelerin oluşturulması, yüklenmesi ve yönetilmesi süreçlerini basitleştiren genel bir kütüphane destek betiğidir.
* **GDBM-1.26:** GNU Database Manager, sistemindeki verilerin key-value mantığıyla çok hızlı bir şekilde saklanmasını ve okunmasını sağlayan geleneksel bir veritabanı kütüphanesidir.
* **Gperf-3.3:** Bir anahtar kümesinden mükemmel bir karma fonksiyonu oluşturur.
* **Expat-2.7.1:** XML verilerini okumak ve anlamlandırmak için kullanılan C dilinde yazılmış XML belgelerini ayrıştırmak (parsing) için kullanılan bir kütüphanedir. 
* **Inetutils-2.6:** Temel ağ iletişimi için programlar içerir.
  * `ping`: Hedef sunucuya ICMP paketleri göndererek ağ erişilebilirliğini ve gecikme süresini test eder.
  * `hostname`: Sistemin ağdaki ismini görüntüler veya ayarlar.
  * `ifconfig`: Arayüzlerini (interfaces) görüntülemek ve yapılandırmak için kullanılan geleneksel araçtır.
  * `telnet` / `ftp`: Uzak sunuculara bağlantı ve dosya transferi için kullanılan klasik istemcilerdir.
  * `traceroute`: Paketlerin hedefe giderken geçtiği router'ları ve ağ yolunu izler.
* **Less-679:** Terminal üzerinde büyük metin dosyalarını veya log çıktılarını sayfa sayfa okumanı sağlayan akıllı bir dosya görüntüleyicidir.
* **Perl-5.42.0:** Genel amaçlı bir programlama dili ve script interpreter’dır.
* **XML::Parser-2.47:** Perl programlama dili için yazılmış bir arayüz modülüdür. Düşük seviyeli C kütüphanesi olan Expat'ı kullanarak, Perl betiklerinin XML belgelerini çok hızlı bir şekilde okumasını, parçalamasını ve işlemesini sağlar.
* **Intltool-0.51.0:** Sistemdeki çeşitli dosya formatlarından (XML, desktop dosyaları, şema dosyaları vb.) çevrilebilir metinleri ayıklayan ve bunları Gettext ile uyumlu hale getiren bir araç setidir.
* **Autoconf-2.72:** Yazılımın sisteme uygun şekilde derlenmesini sağlayan configure scriptini üretir.
* **Automake-1.18.1:** Geliştiricilerin yazdığı basit ve yüksek seviyeli şablon dosyalarını (Makefile.am) make komutunun anlayabileceği karmaşık ve detaylı Makefile dosyalarına dönüştürür.
* **OpenSSL-3.5.2:** Ağ üzerinden iletilen verilerin üçüncü şahıslar tarafından okunmasını engelleyen TLS (Transport Layer Security) ve SSL (Secure Sockets Layer) protokollerini uygular. Ayrıca dosya şifreleme, dijital imza ve sertifika yönetimi (CA) için gerekli araçları sunar.
  * `libcrypto`: Genel amaçlı bir kriptografi kütüphanesidir. Şifreleme algoritmalarını ve karma (hashing) fonksiyonlarını barındırır.
  * `libssl`: TLS protokolünün uygulanmasını sağlar; güvenli bağlantıların (HTTPS gibi) temelidir.
  * `openssl` (komut satırı): Sertifika oluşturma, şifreleme testleri ve anahtar üretimi için kullanılan çok güçlü bir araçtır.
* **Libelf from Elfutils-0.193:** Sistemdeki ikili dosyaların ELF (Executable and Linkable Format) formatı iç yapısını analiz eden ve yöneten temel kütüphanedir.
* **Libffi-3.5.2:** LFS sisteminde farklı programlama dilleri arasında dinamik bir iletişim köprüsü kuran kritik bir arayüz kütüphanesidir. Özellikle Python gibi yüksek seviyeli dillerin, sistemin en alt katmanındaki C fonksiyonlarını çalışma zamanında (run-time) çağırabilmesini mümkün kılar.

* **Python-3.13.7:** LFS sisteminin en güçlü otomasyon ve uygulama geliştirme platformudur.
* **Flit-Core-3.12.0:** Python tabanlı kütüphanelerin modern standartlarda paketlenmesini ve kurulmasını sağlayan hafif bir inşa motorudur.
* **Packaging-25.0:** Python projelerinde sürüm, bağımlılık ve paket bilgilerini yönetmek için kullanılır.
* **Wheel-0.46.1:** Python paketlerinin hızlı ve sorunsuz kurulması için gerekli altyapı. Python paketlerini önceden derlenmiş, taşınabilir “wheel” formatında sunar. Dosya uzantısı: `.whl`. Pip ve diğer paket yöneticileri wheel formatını desteklemek için bu kütüphaneye ihtiyaç duyar.
* **Setuptools-80.9.0:** Python paketlerini indirmek, derlemek, kurmak, yükseltmek ve kaldırmak için kullanılan bir araçtır.
* **Ninja-1.13.1:** Yazılım derleme süreçlerini hızlandırmak için tasarlanmış düşük seviyeli bir inşa aracıdır.
* **Meson-1.8.3:** Hem son derece hızlı hem de olabildiğince kullanıcı dostu olacak şekilde tasarlanmış açık kaynaklı bir derleme sistemidir.
* **Kmod-34.2:** Linux kernel modüllerini yüklemek, kaldırmak ve yönetmek için kullanılır. Çekirdeğe sonradan eklenen donanım sürücüleri veya özellikleri kontrol eder. Modül durumunu görüntüler veya modülleri otomatik yükler.
  * `lsmod`: Yüklü modülleri listeler.
  * `modprobe`: Modül yükler veya çıkarır.
  * `depmod`: Modül bağımlılıklarını yönetir.
* **Coreutils-9.7:** Her işletim sisteminin ihtiyaç duyduğu temel yardımcı programları içerir.
  * `ls`: Dosyaları listeler.
  * `cp`: Kopyalar.
  * `mv`: Taşır.
  * `rm`: Siler.
  * `mkdir`: Klasör oluşturur.
  * `chmod`: İzin değiştirir.
  * `chown`: Sahip değiştirir.
  * `cat`: Okuma.
  * `head` / `tail`: Baş/son okuma.
  * `sort`: Sıralama.
  * `cut`: Kesme.
  * `whoami`: Kullanıcı kimliği.
  * `uptime`: Çalışma süresi vb…
 
* **Diffutils-3.12:** Dosyalar veya dizinler arasındaki farklılıkları gösteren programlar içerir.
* **Gawk-5.3.2:** Metin dosyalarını satır satır okuyup işlemek, filtrelemek ve dönüştürmek için kullanılır.
* **Findutils-4.10.0:** Sistem içinde dosya ve dizin aramak, filtrelemek ve işlem yapmak için kullanılır.
  * `find`: Dosya arar.
  * `xargs`: Komutlara argüman gönderir.
  * `locate`: Hızlı dosya arama (veritabanı üzerinden) yapar.
* **Groff-1.23.0:** Metin ve görüntüleri işlemek ve biçimlendirmek için programlar içerir.
* **GRUB-2.12:** LFS sisteminin "başlatıcısı" ve işletim sistemi ile donanım arasındaki ilk gerçek temastır. Bilgisayar açıldığında BIOS veya UEFI'den kontrolü devralan, çekirdeği (kernel) belleğe yükleyen ve sistemi ayağa kaldıran Grand Unified Bootloader'dır.
* **Gzip-1.14:** Dosyaları sıkıştırmak ve açmak için programlar içerir.
  * `gzip`: Dosyaları sıkıştırır.
  * `gunzip`: .gz uzantılı sıkıştırılmış dosyaları orijinal haline geri döndürür.
* **IPRoute2-6.16.0:** Ağ (network) ayarlarını görüntüleme ve yönetme işlemlerini yapar.
  * `ip`: Paketin kalbidir. `ip addr` (adresleme), `ip link` (arayüz yönetimi), `ip route` (yönlendirme tablosu) gibi komutlarla ağın her detayını kontrol eder.

* **Kbd-2.8.0:** Linux’ta klavye düzeni (layout), tuş davranışları ve konsol ayarlarını yönetir.
* **Libpipeline-1.5.8:** Unix/Linux pipeline’larını yönetmeyi kolaylaştırır. Birden fazla komutun çıktısını bir sonraki komuta aktarmayı kolaylaştırır. Pipe (|) kullanımını programatik hale verir.
* **Make-4.4.1:** Kaynak kodu derlemek ve programları oluşturmak için kullanılır. Makefile içindeki kurallara göre hangi dosyanın yeniden derleneceğini belirler. Değişiklikleri takip ederek gereksiz derlemeleri önler.
* **Patch-2.8:** Kaynak kod dosyalarına düzeltme (diff) eklerini uygular.
* **Tar-1.35:** Tar paketi, tar arşivleri oluşturmanın yanı sıra çeşitli diğer arşiv işlemlerini gerçekleştirme olanağı sağlar. Tar, önceden oluşturulmuş arşivlerden dosyaları çıkarmak, ek dosyaları depolamak veya önceden depolanmış dosyaları güncellemek veya listelemek için kullanılabilir.
* **Texinfo-7.2:** Yazılımların kılavuzlarını ve dökümantasyonunu oluşturmak için kullanılır. GNU yazılımları genellikle Texinfo ile belgelenir.
  * `make info`: Terminal için doküman üretir.
  * `make pdf`, `make html`: Basılı veya web dokümanı üretir.
* **Vim-9.1.1629:** Terminal tabanlı metin düzenleme ve programlama editörüdür. Kod yazmak, konfigürasyon dosyalarını düzenlemek, dosya yönetmek için kullanılır.
* **MarkupSafe-3.0.2:** Python yardımcı kütüphanesidir. Python’da güvenli metin işleme sağlar, özellikle HTML veya XML’deki özel karakterleri kaçırmak (escape) için kullanılır. Yani Python uygulamalarında güvenli metin işleme için gerekli altyapı.
* **Jinja2-3.1.6:** Python tabanlı şablon (template) motorudur. Python ile dinamik içerik oluşturmayı kolaylaştırır, özellikle web uygulamalarında HTML veya metin dosyaları üretmek için kullanılır.
* **Udev from Systemd-257.8:** Linux cihaz yönetim sistemidir. Linux sistemde donanım cihazlarının (USB, disk, ağ kartı vb.) yönetimini ve otomatik tanınmasını sağlar.
* **Man-DB-2.13.1:** Man sayfalarını bulmak ve görüntülemek için programlar içerir.
  * `apropos`: Belirli bir anahtar kelimeye göre tüm kılavuz sayfalarının başlıklarında arama yapar (örneğin: `apropos network`).
  * `whatis`: Bir komutun ne işe yaradığını tek satırlık bir özetle açıklar.
 
 * **Procps-ng-4.0.5:** İşletim sisteminde o an çalışan süreçlerin durumunu, bellek kullanımını ve işlemci yükünü izlemek, yönetmek için kullanılan bir kütüphane ve araçlar kümesidir.
  * `top`: Sistem kaynaklarını (CPU, RAM) en çok tüketen süreçleri canlı ve interaktif bir şekilde listeler.
  * `ps`: O an çalışan süreçlerin anlık bir "fotoğrafını" çeker (Process Status).
  * `free`: Sistemin toplam, boş ve kullanılan bellek (RAM/Swap) miktarını gösterir.
  * `kill`: Belirli bir sürece sinyal göndererek durdurma, yeniden başlatma, sonlandırma yönetilmesini sağlar.
  * `sysctl`: Çekirdek parametrelerini (kernel parameters) çalışma anında görüntülemeye ve değiştirmeye yarar.
  * `uptime`: Sistemin ne kadar süredir açık olduğunu ve ortalama yükü gösterir.

* **Util-linux-2.41.1:** Linux çekirdeğiyle en alt seviyede konuşan, disk bölümlendirmeden terminal yönetimine, kullanıcı girişlerinden dosya sistemi bağlamaya kadar sistemin çalışması için gereken en kritik düşük seviyeli araçları barındırır.
  * **Disk Yönetimi:** `fdisk`, `cfdisk` (disk bölümlendirme), `mount`/`umount` (diskleri sisteme bağlama/ayırma).
  * **Terminal ve Giriş:** `login` (kullanıcı girişi), `agetty` (terminal bağlantısı), `more` (metin görüntüleme).
  * **Sistem Bilgisi:** `lscpu` (işlemci detayları), `lsblk` (blok aygıtları/disk listesi).
  * **Dosya Sistemi:** `mkfs` (dosya sistemi oluşturma/formatlama), `fsck` (dosya sistemi kontrolü).
  * **Zaman ve Kimlik:** `hwclock` (donanım saati yönetimi), `su` (kullanıcı değiştirme).
  * *Not:* Bu paket olmadan bir diski formatlayamaz, sisteme bağlayamaz ve hatta kullanıcı girişi yapamazsın. LFS sisteminin "kullanılabilir" bir bilgisayar haline gelmesini sağlayan ham mekanik araçlardır.

* **E2fsprogs-1.47.3:** Ext2, ext3, ext4 dosya sistemlerini oluşturmak, doğrulamak ve yönetmek için kullanılan temel kütüphaneleri ve programları sağlar.
  * `mke2fs` / `mkfs.ext4`: Bir disk bölümünü Ext4 formatında biçimlendirir (dosya sistemini oluşturur).
  * `e2fsck`: Dosya sistemindeki hataları tarar ve onarır (Windows'taki chkdsk karşılığı).
  * *Not:* Bir disk bölümü oluşturmak yetmez; o bölümün üzerine veri yazılabilmesi için bir "düzen" (dosya sistemi) kurulmalıdır. E2fsprogs bu düzenin mimarıdır.

* **Sysklogd-2.7.2:** Sysklogd sistemin merkezi logging mekanizmasıdır. Çekirdek ve kullanıcı alanı uygulamalarından gelen tüm operasyonel verileri sistem kayıtlarına dönüştürerek, sistemin izlenebilirliğini, güvenliğini ve hata teşhis süreçlerini yönetir. `/var/log` altındaki dosyalara kaydeden temel servistir.
* **SysVinit-3.14:** Sistemin başlatılmasını, çalıştırılmasını ve kapatılmasını kontrol eden programlar içerir.
* **Stripping:** Stripping aşaması, LFS sistemindeki ikili dosyaları (binary) ve kütüphaneleri gereksiz hata ayıklama verilerinden arındırarak optimize eder. Bu işlem, sistemin disk kapladığı alanı minimize ederken, programların yüklenme hızını artırarak saf ve performanslı bir işletim sistemi yapısı sunar.

<img width="518" height="161" alt="image26" src="https://github.com/user-attachments/assets/dcb77004-cd34-41fc-a9bf-7fc495361e19" />

Stripping yapılmadan önce 4.6 Gb civarındaydı stripping yapıldıktan sonra 1.9 gb düşmüştür.

## Bölüm 9: Sistem Yapılandırması

LFS sürecinde sistemin artık çalışabilir ve boot edilebilir hale getirildiği bölümdür. Bu bölümün temel amacı kernel yüklendikten sonra sistemin nasıl davranacağını, donanımları nasıl tanıyacağını ve dış dünya ile nasıl iletişim kuracağını belirleyen kurallar setini (konfigürasyon dosyalarını) oluşturmaktır.

### LFS-Bootscripts-20250827
Sistemin açılış ve kapanış süreçlerini yöneten operasyonel betikler bütünüdür. Donanım hazırlığı, dosya sistemi kontrolü ve ağ servislerinin başlatılması gibi kritik görevleri hiyerarşik bir düzende yürüterek, sistemin kararlı bir şekilde çalışma durumuna geçmesini sağlar.


 
* **Genel network ayarları**
<img width="736" height="415" alt="image8" src="https://github.com/user-attachments/assets/4d9e3e94-e5a8-4484-9c24-38fbb5c31eeb" />



LFS sisteminin açılışta ağa otomatik olarak bağlanabilmesi ve kalıcı bir IP adresine sahip olması için ağ arayüzü konfigürasyon dosyası oluşturulmuştur. 

`/etc/sysconfig/` dizinine geçiş yapılarak `ifconfig.ens33` dosyası statik IP mimarisine uygun şekilde şu parametrelerle yapılandırılmıştır:

* **DNS ayarları**

<img width="615" height="201" alt="image23" src="https://github.com/user-attachments/assets/ad93fd3a-e2ba-484a-bd84-63182f8fae99" />

DNS istemci yapılandırma dosyası olan `/etc/resolv.conf`, `cat > /etc/resolv.conf << "EOF"` komut blokları kullanılarak chroot ortamında sıfırdan oluşturulmuştur. Bu sayede internete çıkabilir.

* **Hostname ayarları**

<img width="653" height="44" alt="image2" src="https://github.com/user-attachments/assets/bf9e6f57-8adc-4672-901f-08e66b3e2fbb" />

adı lfs12-4_Murat olmuştur.

* **Host ayarları(/etc/hosts)**

<img width="490" height="232" alt="image20" src="https://github.com/user-attachments/assets/104550e3-4284-4d14-8f69-0256f86002de" />



Ağ arayüzü ve DNS tanımlamalarının ardından, sistemin dışarıdaki bir DNS sunucusuna ihtiyaç duymadan kendi ismini (hostname) ve yerel adresleri çok hızlı bir şekilde çözümleyebilmesi amacıyla `/etc/hosts` dosyası yapılandırılmıştır.

 LFS sisteminin ağdaki tam nitelikli alan adı (FQDN) `lfs12-4_Murat.localdomain` ve kısa adı `lfs12-4_Murat` olarak tanımlandı ve ilgili IP adresleriyle şu şekilde eşleştirildi.

 127.0.0.1 localhost.localdomain 
 
 localhost Standart IPv4 loopback adresidir. Sistemin kendi içindeki ağ servislerinin kesintisiz haberleşmesini sağlar.

	127.0.1.1 lfs12-4_Murat.localdomain 
 lfs12-4_Murat Ağ kartı aktif olmasa bile bazı sistem servislerinin ve uygulamaların hostname üzerinden yerel makineye hızlıca erişebilmesi için tanımlanan döngü adresidir.

	192.168.1.75 lfs12-4_Murat.localdomain 
 lfs12-4_Murat ens33 ağ kartına atanan statik yerel IP adresi ile sistemin ismi birbirine bağlanmıştır. Bu sayede yerel ağdaki diğer cihazlar ve iç servisler sisteme bu isimle erişebilir hale getirilmiştir.


* **Consol,Klavye ve font ayarları**

<img width="392" height="215" alt="image10" src="https://github.com/user-attachments/assets/fb83eeb8-c008-48d3-94ff-c74af2bc58a6" />

/etc/sysconfig/console dan ayarlanabilir.


## 10. Bölüm: LFS Sistemini Önyüklenebilir Hale Getirme

LFS sistemini önyüklenebilir hale getirildiği bölüm. Bu bölümde `/etc/fstab` dosyasının oluşturulması, yeni LFS sistemi için bir çekirdeğin (kernel) derlenmesi ve LFS sisteminin başlangıçta önyükleme için seçilebilmesi için GRUB önyükleyicisinin yüklenmesi ele alınmaktadır.

<img width="964" height="461" alt="image7" src="https://github.com/user-attachments/assets/804d1c10-ec98-4b2e-a0b1-214334a63af6" />

<img width="692" height="343" alt="image17" src="https://github.com/user-attachments/assets/101a6cd2-9cf1-4481-aeed-9e5be3f2ecd9" />

/etc/fstab dosyası oluşturuldu. sdb diskinin uuıd’lerine göre yapılandırıldı.

* **Linux-6.16.1:** İşletim sisteminin çekirdeğini oluşturur. Donanım kaynaklarını (CPU, RAM, Disk) uygulamalar arasında paylaştırır ve sistemin fiziksel birimleriyle (VMware sürücüleri, ağ kartları vb.) doğrudan iletişim kurar.

`make menuconfig` komutu ile aşağıdaki bölüme gireriz. Kitaba göre yönlendirilip yapılandırılmalıdır. Debian VMware de olduğu için Debian’ın diskleri varsayılanlar arasında SCSI arayüzü ile tanımlanmıştır; o yüzden işaretlemeleri yaparken SCSI ile olan işaretlemeler unutulmamalı.

<img width="916" height="491" alt="image14" src="https://github.com/user-attachments/assets/9ab3cf4d-9061-42ed-93b5-47ed149861f0" />

* **GRUB Yapılandırma Dosyasının Oluşturulması:** `/boot/grub/grub.cfg` dosyasını oluşturarak, sistemin açılış menüsünü ve boot seçeneklerini tanımlar. GRUB'un otomatik olarak değil, LFS mimarisine uygun şekilde manuel yapılandırıldığı aşamadır.

<img width="702" height="283" alt="image15" src="https://github.com/user-attachments/assets/6db2561e-bd03-4a1d-93cf-486210bfa82c" />

## 11. bölüm: Açılış 

<img width="1122" height="813" alt="image11" src="https://github.com/user-attachments/assets/5cf47ebf-9b90-404c-96c4-d2a3e89a92e5" />

LFS'in ayağa kalktığı  ve boot edildiği güzel bir görüntüdür bu.

<img width="661" height="498" alt="image5" src="https://github.com/user-attachments/assets/94fec95a-a8df-4cd7-9a30-19a230c52af5" />


## Test

<img width="383" height="180" alt="image24" src="https://github.com/user-attachments/assets/949504c9-7a10-4f64-b598-9934ea186f1f" />

hostname,kimliği ve sürüm bilgisi görebiliyoruz. lfs12-4_Murat bizim hostname ismimiz login root password:2553796mu

<img width="587" height="299" alt="image9" src="https://github.com/user-attachments/assets/dc49190a-03b5-4835-943d-ed349c3a5ad6" />

Ping atabiliyoruz ve internete çıkabiliyoruz.


