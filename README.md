# linux-from-scratch-project
Debian 12 üzerinde LFS 12.4 metodolojisi ile sıfırdan derlenmiş bağımsız Linux işletim sistemi inşası ve detaylı teknik dokümantasyonu.

## 1. Amaç ve Kapsam

Bu projede, VMware sanallaştırma platformu üzerinde çalışan **Debian 12** ana sistemi (Host) kullanılarak, **Linux From Scratch (LFS v12.4)** metodolojisi ile bağımsız bir Linux dağıtımının sıfırdan inşa edilmesi hedeflenmiştir. 

Projenin temel amacı; hazır bir Linux dağıtımı kullanmak yerine, işletim sistemini kaynak kodlarından manuel olarak derleyerek Linux çekirdeğinin ve sistem mimarisinin çalışma mantığını derinlemesine deneyimlemektir. 

**Proje kapsamında gerçekleştirilen temel aşamalar:**
* Sistem inşası için gerekli **Toolchain** (derleme araç takımı) ortamının hazırlanması,
* Temel sistem paketlerinin ve kütüphanelerin kaynak koddan manuel olarak derlenmesi,
* **Kernel (Çekirdek) konfigürasyonu** ve sistemin kendi başına  boot edilebilir hale getirilmesi.

Bu çalışma sonucunda; Linux dosya sistemi hiyerarşisi, bağımlılık yönetimi, derleme süreçleri (SBU yönetimi) ve Linux sistem mimarisi derinlemesine analiz edilerek dökümante edilmiştir.

## 2. Yöntem ve Teknik Özellikler

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


## 3. LFS Sistem İnşa Süreci ve Yol Haritası

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









