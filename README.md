# hello-network

# Proje 1 — Noktadan Noktaya Ağ (Peer-to-Peer)

## Proje Hakkında

Bu projede, herhangi bir router veya switch kullanmadan iki cihazın doğrudan birbirleriyle iletişim kurması sağlandı.

Senaryo basit:

> İnternet yok, router yok, switch yok. Sadece iki cihaz ve aralarında bir bağlantı var.

Amaç; cihazların ağ üzerindeki kimliklerini anlamak, birbirlerini bulmalarını sağlamak ve ilk `ping` paketini başarılı şekilde göndermek.

---

## Öğrenilen Temel Kavramlar

### OSI Modeli — Layer 1, 2 ve 3

Bu projede ağ iletişiminin temelini oluşturan ilk üç katmana odaklanıldı.

| Katman              | Görevi                                       | Örnek                 |
| ------------------- | -------------------------------------------- | --------------------- |
| Layer 3 — Network   | Mantıksal adresleme ve ağlar arası iletişim  | IP                    |
| Layer 2 — Data Link | Aynı ağdaki cihazların fiziksel adreslenmesi | MAC                   |
| Layer 1 — Physical  | Verinin fiziksel ortam üzerinden taşınması   | Ethernet kablosu, NIC |

Basitçe düşünürsek:

* **Layer 3:** "Hangi cihaza/ağa gitmeliyim?"
* **Layer 2:** "Bu cihazın MAC adresi ne?"
* **Layer 1:** "Veriyi fiziksel olarak nasıl göndereceğim?"

Veri gönderilirken bu katmanlardan geçerek paketlenir (**encapsulation**). Alıcı tarafta ise bu işlem tersine çevrilerek veri açılır (**decapsulation**).

---

## IP ve MAC Adresi

Ağ iletişiminde IP ve MAC adreslerinin farklı görevleri vardır.

### IP Adresi

IP adresi cihazın ağ üzerindeki **mantıksal adresidir**.

Örneğin:

```text
172.16.110.128
```

IP adresi bulunduğu ağa göre değişebilir.

### MAC Adresi

MAC adresi ise ağ kartının (**NIC**) Layer 2 seviyesindeki adresidir.

Örneğin:

```text
00:0c:29:56:b0:fc
```

Aynı ağ üzerinde bir cihazın fiziksel olarak tanınmasında kullanılır.

Kısaca:

```text
IP  → Hangi ağdaki cihaz?
MAC → Aynı ağdaki hangi cihaz?
```

---

## ARP ve ICMP

### ARP — Address Resolution Protocol

Bir cihaz aynı ağdaki başka bir cihazın IP adresini biliyor fakat MAC adresini bilmiyorsa ARP kullanılır.

Örneğin:

```text
172.16.110.128 kimde?
MAC adresini söyle.
```

Bu istek ağ üzerinde yayınlanır (**broadcast**).

Hedef cihaz cevap verdiğinde IP ve MAC eşleştirmesi ARP cache'e kaydedilir.

---

### ICMP — Ping

`ping` komutu ICMP kullanarak hedef cihazın erişilebilir olup olmadığını kontrol eder.

Basit akış:

```text
Mac
 │
 │ ICMP Echo Request
 ▼
Kali Linux
 │
 │ ICMP Echo Reply
 ▼
Mac
```

Bu sayede iki cihaz arasındaki bağlantının çalışıp çalışmadığı kontrol edilebilir.

---

# Laboratuvar Ortamı

### Kullanılan Sistemler

* **Host:** macOS
* **Virtual Machine:** Kali Linux
* **Virtualization:** VMware Fusion
* **Network:** Host-Only

Host-Only ağ kullanıldığı için Kali ile macOS arasında sanal bir ağ oluşturuldu. Bu ağın internete çıkması gerekmiyor.

---

# Uygulama

## 1. Ağ Arayüzlerini Kontrol Etme

Kali Linux üzerinde:

```bash
ip a
```

komutu çalıştırıldı.

Çıktıda temel olarak şu arayüzler görülebilir:

### `lo`

```text
127.0.0.1
```

Bu **loopback** arayüzüdür.

Cihazın kendi TCP/IP yapısını test etmek için kullanılır.

### `eth0`

Kali'nin sanal Ethernet arayüzüdür.

Örneğin:

```text
172.16.110.128
```

Aynı çıktıda:

```text
link/ether
```

ifadesinin yanında MAC adresi de görülebilir.

---

## 2. Ping ile Bağlantıyı Test Etme

Mac terminalinden Kali'nin IP adresine ping gönderildi:

```bash
ping 172.16.110.128
```

Örnek cevap:

```text
64 bytes from 172.16.110.128: icmp_seq=0 ttl=64 time=0.638 ms
```

Buradaki önemli kısımlar:

* `64 bytes from ...` → Kali'den ICMP Echo Reply alındığını gösterir.
* `time=0.638 ms` → Paketin gidiş-dönüş süresidir.

Host-Only sanal ağ kullanıldığı için gecikmenin çok düşük olması beklenir.

---

## 3. ARP Tablosunu Kontrol Etme

Ping işleminden sonra macOS'un Kali'nin IP adresini hangi MAC adresiyle eşleştirdiğini kontrol etmek için:

```bash
arp -a
```

komutu çalıştırıldı.

Örnek çıktı:

```text
? (172.16.110.128) at 0:c:29:56:b0:fc on bridge100 ifscope [bridge]
```

Burada:

```text
172.16.110.128
        ↓
0:c:29:56:b0:fc
```

eşleştirmesi görülüyor.

Yani macOS, Kali'nin IP adresini ilgili MAC adresiyle eşleştirmiş ve bu bilgiyi ARP cache içerisinde tutmuş.

### `bridge100` nedir?

`bridge100`, macOS'un VMware tarafından oluşturulan sanal Host-Only ağ arayüzlerinden biri için kullandığı arayüz ismidir.

---

# Sonuç

Bu projede iki cihaz arasında router veya switch olmadan temel ağ iletişimi gerçekleştirildi.

Süreç özetle şöyle:

```text
Physical Connection
        ↓
Layer 1 — Ethernet
        ↓
Layer 2 — MAC / ARP
        ↓
Layer 3 — IP
        ↓
ICMP
        ↓
Ping
```

Sonuç olarak:

* Cihazların ağ arayüzleri incelendi.
* IP ve MAC adresleri arasındaki fark görüldü.
* ARP ile IP → MAC eşleştirmesi gözlemlendi.
* ICMP `ping` ile bağlantı test edildi.
* macOS ve Kali Linux arasında başarılı iletişim sağlandı.

**Proje 1 tamamlandı.**
