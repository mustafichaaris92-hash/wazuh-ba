# 🛡️ Vodič 02: Instalacija CentOS Stream 10 na Hyper-V

### Kreiranje virtualne mašine i osnovna konfiguracija sistema

| Stavka | Vrijednost |
|:---|:---|
| **Projekat** | Wazuh BA |
| **Kategorija** | Installation / Virtualization |
| **Platforma** | Microsoft Hyper-V |
| **Operativni sistem** | CentOS Stream 10 |
| **Preduslov** | Vodič 01 – Priprema Hyper-V okruženja |

---

## 🎯 Cilj vodiča

Kreiramo virtualnu mašinu i instaliramo CentOS Stream 10, koji će biti osnova za instalaciju Wazuh platforme u narednim vodičima.

## 📥 1. Preuzimanje ISO datoteke

Preuzmi **x86_64 DVD ISO** sa službene stranice: [centos.org/download](https://www.centos.org/download/)

```text
CentOS-Stream-10-latest-x86_64-dvd1.iso
```

Provjeri SHA256 checksum prema vrijednosti objavljenoj na službenom mirroru:

```powershell
Get-FileHash "C:\ISO\CentOS-Stream-10-latest-x86_64-dvd1.iso" -Algorithm SHA256
```

> ⚠️ **Važno:** CentOS Stream 10 cilja arhitekturu **x86-64-v3**. Stariji procesori bez potrebnih instrukcija mogu imati problem pri pokretanju instalacije.

## 🖥️ 2. Kreiranje virtualne mašine

Hyper-V Manager → **New** → **Virtual Machine**, pa unesi sljedeće postavke:

| Korak | Postavka |
|:---|:---|
| **Name** | `WAZUH-CENTOS-10` |
| **Generation** | Generation 2 *(ne može se naknadno mijenjati)* |
| **Startup memory** | `8192 MB` |
| **Networking** | `External-Switch` ili `Default Switch` |
| **Virtual hard disk** | `WAZUH-CENTOS-10.vhdx`, 100 GB |
| **Installation Options** | Install from bootable image file → izaberi ISO |

> 💡 **Memorija:** isključi opciju `Use Dynamic Memory`. Wazuh Indexer (OpenSearch) bolje radi s fiksnom memorijom, pa je 8 GB fiksno dobar početak. Za veći broj agenata planiraj više.

> ℹ️ **Disk:** Hyper-V po defaultu kreira dinamički VHDX, koji zauzima fizički prostor samo prema stvarno upisanim podacima, do postavljenog maksimuma.

## ⚙️ 3. Podešavanje prije pokretanja

Desni klik na VM → **Settings**:

1. **Processor** → Number of virtual processors: `4`
2. **Security** → ako instalacija ne može da se pokrene, isključi `Enable Secure Boot`. Alternativa je template **Microsoft UEFI Certificate Authority**.
3. **SCSI Controller / DVD Drive** → provjeri da je ISO povezan.

Klikni **Apply** → **OK**.

## 🚀 4. Pokretanje instalacije

Desni klik na VM → **Connect** → **Start**. Kada se pojavi boot meni, izaberi instalaciju CentOS Stream 10.

## 🧩 5. Anaconda installer

U instalacijskom programu podesi:

| Sekcija | Postavka |
|:---|:---|
| **Language** | English (United States) |
| **Installation Destination** | Virtualni disk od 100 GB, Automatic Storage Configuration |
| **Software Selection** | `Server` (bez grafičkog okruženja) |
| **Time & Date** | `Europe/Sarajevo` |
| **Root Account** | Snažna lozinka |
| **User Creation** | Korisnik `wazuhadmin` s administratorskim privilegijama |

Klikni **Begin Installation**, sačekaj završetak i izaberi **Reboot System**.

> ⚠️ Ako se VM nakon restarta ponovo pokrene s instalacijskog medija, isključi ISO iz virtualnog DVD Drivea i pokreni VM ponovo.

## 🔧 6. Osnovna konfiguracija

Prijavi se kreiranim korisnikom i izvrši:

```bash
sudo hostnamectl set-hostname wazuh-centos-10
sudo dnf update -y
sudo reboot
```

> 💡 Ispravno vrijeme je važno za korelaciju sigurnosnih događaja i analizu logova.

## 🧪 7. Provjera ispravnosti

Nakon restarta izvrši:

```bash
cat /etc/os-release
hostnamectl
ip addr
ip route
ping -c 4 1.1.1.1
ping -c 4 google.com
df -h
free -h
timedatectl
```

Potvrdi da:

- ✅ CentOS Stream 10 je instaliran i sistem se normalno pokreće.
- ✅ Hostname je ispravno podešen.
- ✅ Mrežni adapter ima IP adresu i radi internet konekcija.
- ✅ DNS rezolucija radi (ako prvi `ping` radi, a drugi ne, provjeri DNS).
- ✅ Disk, memorija, vrijeme i vremenska zona su ispravni.

## 🛠️ Najčešći problemi

- **VM se ne pokreće ili instalacija ne starta:** provjeri da je Generation 2, da je ISO ispravan i da procesor podržava x86-64-v3. Privremeno isključi Secure Boot.
- **Nema interneta:** provjeri koji je Virtual Switch povezan na VM i da host ima aktivnu konekciju (`ip addr`, `ip route`).
- **Nedostaje prostora na disku:** proširi virtualni disk u Hyper-V Manageru, pa zatim particiju i filesystem unutar Linuxa.

## 🏁 Rezultat

Imamo funkcionalnu VM `WAZUH-CENTOS-10` s ažuriranim CentOS Stream 10 sistemom.

➡️ **Sljedeći korak:** Vodič 03 – Wazuh arhitektura i instalacija komponenti.

---

## 📚 Službeni izvori

- [Microsoft Learn – Create a virtual machine in Hyper-V](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v)
- [CentOS – Download](https://www.centos.org/download/)
- [CentOS Stream 10 Release Notes](https://www.centos.org/centos10/)

**🛡️ Wazuh BA – Uči. Implementiraj. Testiraj. Zaštiti.**