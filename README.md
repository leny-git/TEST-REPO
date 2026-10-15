[cybersecurity_linux_masterclass_command_arsenal.md](https://github.com/user-attachments/files/33502416/cybersecurity_linux_masterclass_command_arsenal.md)
# 🛡️ Ultimate Linux Cybersecurity & Forensics Masterclass

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Linux-orange?style=for-the-badge&logo=linux" alt="Linux">
  <img src="https://img.shields.io/badge/Focus-Red%20%2F%20Blue%20Team-red?style=for-the-badge&logo=security" alt="Security">
  <img src="https://img.shields.io/badge/Level-Advanced-purple?style=for-the-badge" alt="Level">
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="License">
</p>

> *"Linux, siber güvenliğin kalbidir. Ne kadar derine inersen, o kadar güçlü olursun."*

---

## 📌 İçindekiler (Table of Contents)
1. [Giriş ve FHS (Filesystem Hierarchy Standard) Analizi](#1-giriş-ve-fhs-filesystem-hierarchy-standard-analizi)
2. [Yetkilendirme, SUID/SGID ve Privilege Escalation Avcılığı](#2-yetkilendirme-suidsgid-ve-privilege-escalation-avcılığı)
3. [Ağ Adli Bilişimi ve Trafik Analizi (Network Forensics)](#3-ağ-adli-bilişimi-ve-trafik-analizi-network-forensics)
4. [Log Analizi ve Incident Response (Olay Müdahale)](#4-log-analizi-ve-incident-response-olay-müdahale)
5. [Süreç, Bellek ve Rootkit Tespiti](#5-süreç-bellek-ve-rootkit-tespiti)
6. [Sistem Sıkılaştırma (Hardening) ve Güvenlik Best Practices](#6-sistem-sıkılaştırma-hardening-ve-güvenlik-best-practices)

---

## 1. Giriş ve FHS (Filesystem Hierarchy Standard) Analizi

Siber güvenlik uzmanı veya sızma testi (penetration test) uzmanı bir sisteme sızdığında veya inceleme yaptığında nereye bakacağını bilmelidir. Linux dosya sistemi hiyerarşisi bu yüzden kritik önem taşır.

| Dizin | Güvenlik Açısından Önemi | Kritik Dosyalar |
| :--- | :--- | :--- |
| `/etc` | Konfigürasyon dosyalarının kalbidir. Şifreler, servis ayarları burada saklanır. | `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/ssh/sshd_config` |
| `/var/log` | Log kayıtları. Saldırganların iz silmeye çalıştığı, Blue Team'in ise delil aradığı yerdir. | `/var/log/auth.log`, `/var/log/syslog`, `/var/log/apache2/` |
| `/proc` & `/sys` | Fiziksel diskte yer almayan, çekirdek ve bellek durumunu anlık yansıtan sanal dosya sistemleri. | `/proc/version`, `/proc/cmdline`, `/proc/net/tcp` |
| `/tmp` & `/dev/shm` | Geçici dosya alanları. Kötü amaçlı yazılımların (malware) sıklıkla payload sakladığı ve çalıştırdığı dizinler. | - |

### 🔍 Kritik Dosya İnceleme Komutları
```bash
# Sistemdeki kullanıcıları ve kabuk (shell) durumlarını listele
cat /etc/passwd | grep -v "/nologin\|/false"

# Sudo yetkisine sahip komutları kontrol et (Yetki aşımı tespiti için)
sudo -l

# Çekirdek (Kernel) sürümünü ve mimarisini öğren (Exploit araştırması için)
uname -mrs && cat /etc/os-release
```

---

## 2. Yetkilendirme, SUID/SGID ve Privilege Escalation Avcılığı

Linux güvenlik denetimlerinin en kritik adımı **Privilege Escalation** (Yetki Yükseltme) analizidir. Düşük yetkili bir kullanıcıdan `root` seviyesine çıkmak için kullanılan ana vektörler **SUID**, **Capabilities** ve **Zayıf Cron Job**'lardır.

### ⚡ SUID / SGID Dosya Avı
SUID (Set Owner User ID) bayrağına sahip dosyalar, çalıştırıldıkları kullanıcının değil, **dosya sahibinin** (genellikle root) yetkileriyle çalışır. Yanlış konfigüre edilmiş SUID dosyaları sistemin ele geçirilmesini sağlar.

```bash
# Sistemdeki tüm SUID yetkisine sahip dosyaları bul ve hataları gizle
find / -perm -4000 -type f 2>/dev/null

# SGID bayrağına sahip dosyaları listele
find / -perm -2000 -type f 2>/dev/null

# Belirli bir binary üzerinde SUID olup olmadığını kontrol et
ls -l /usr/bin/passwd
```

### ⚙️ Capabilities (Yetenekler) Kontrolü
Her zaman `root` olmak gerekmez; bazı modern Linux çekirdeklerinde işlemler doğrudan özel yeteneklerle donatılabilir.
```bash
# Dosya yeteneklerini (capabilities) tara
getcap -r / 2>/dev/null
```

---

## 3. Ağ Adli Bilişimi ve Trafik Analizi (Network Forensics)

Bir sunucuda arka kapı (backdoor) veya yetkisiz bir dinleme (listener) olup olmadığını anlamak için ağ soketleri ve trafik akışı incelenmelidir.

### 🌐 Aktif Bağlantı ve Dinlenen Port Analizi
```bash
# Dinlenen portları ve sürece ait PID/Program adını detaylı listele (TCP & UDP)
sudo ss -tulpn

# Alternatif olarak netstat kullanımı
sudo netstat -antup | grep LISTEN

# Belirli bir IP adresine yapılmış aktif bağlantıları izle
watch -n 1 "netstat -nat | grep ESTABLISHED"
```

### 🦈 Ağ Trafiği Yakalama (Tcpdump)
```bash
# Belirli bir arayüzde (eth0) geçen paketleri tcpdump ile yakalayıp pcap dosyasına kaydet
sudo tcpdump -i eth0 -w suspect_traffic.pcap

# Hassas portlardaki (örneğin SSH - 22) trafiği ekranda canlı izle
sudo tcpdump -nnvvXS -s0 port 22
```

---

## 4. Log Analizi ve Incident Response (Olay Müdahale)

Saldırı anında veya sonrasında saldırganın izini sürmek için log dosyaları incelenir. Kim, ne zaman, hangi IP üzerinden sisteme girdi?

### 🔎 Yetkisiz Erişim ve Brute-Force Tespiti (`auth.log`)
```bash
# Başarısız SSH giriş denemelerini ve kaynak IP'leri listele
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr

# Başarılı SSH oturum açma geçmişini incele
grep "Accepted" /var/log/auth.log

# Sudo komutunu kullanan kullanıcıların geçmişini dök
grep "COMMAND" /var/log/auth.log
```

### 📜 Journalctl ile Sistem Logları Analizi
```bash
# Son 1 saat içerisindeki kritik hata ve uyarı loglarını getir
journalctl -p err..warning --since "1 hour ago"

# SSH servisinin anlık log akışını takip et
journalctl -u ssh.service -f
```

---

## 5. Süreç, Bellek ve Rootkit Tespiti

Saldırganlar sistemde kalıcı olmak (persistence) için süreci gizleyebilir veya rootkit kurabilirler.

### 👻 Gizli Süreç ve Rootkit Taraması
```bash
# Anormal çalışan, kök dizinden (/) çalışan veya ismi gizlenmiş süreçleri listele
ps auxf

# Açık olan dosya tanımlayıcılarını (file descriptors) ve bunlara bağlı ağ soketlerini incele
sudo lsof -i -P -n

# Bilinen rootkit imzalarını taramak için chkrootkit ve rkhunter araçları
sudo chkrootkit
sudo rkhunter --check
```

---

## 6. Sistem Sıkılaştırma (Hardening) ve Güvenlik Best Practices

Bir Linux sistemini üretime almadan veya dış dünyaya açmadan önce mutlaka şu sıkılaştırma adımları uygulanmalıdır:

1. **SSH Sıkılaştırma (`/etc/ssh/sshd_config`):**
   * `PermitRootLogin no` (Root kullanıcısının doğrudan SSH ile girişini kapat)
   * `PasswordAuthentication no` (Şifre ile girişi kapat, sadece SSH Key kullan)
   * `Port <özel_port>` (Varsayılan 22 portunu değiştir)

2. **Güvenlik Duvarı Yapılandırması (UFW / IPTables):**
   ```bash
   # UFW'yi etkinleştir, tüm gelenleri engelle, sadece izin verilenleri aç
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow 22/tcp  # (veya değiştirdiğiniz SSH portu)
   sudo ufw enable
   ```

3. **Gereksiz Servislerin Kapatılması:**
   ```bash
   # Çalışmayan veya kullanılmayan servisleri devre dışı bırak
   sudo systemctl disable <servis_adi> --now
   ```

---

<p align="center">
  <b>Developed with 💻 & 🛡️ for Security Enthusiasts</b>
</p>
