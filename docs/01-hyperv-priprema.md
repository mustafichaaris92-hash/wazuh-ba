# 🛡️ Vodič 01: Priprema Hyper-V okruženja

### Provjera hardvera, uključivanje Hyper-V i kreiranje Virtual Switcha

| Stavka | Vrijednost |
|:---|:---|
| **Projekat** | Wazuh BA |
| **Kategorija** | Installation / Virtualization |
| **Platforma** | Microsoft Hyper-V |
| **Sljedeći vodič** | 02 – Instalacija CentOS Stream 10 |

---

## 🎯 Cilj vodiča

Pripremamo Windows host za rad s virtualnim mašinama. Nakon ovog vodiča imat ćemo aktivan Hyper-V i Virtual Switch, spremne za kreiranje VM-a u sljedećem vodiču.

> 💡 Ako već imaš funkcionalan Hyper-V i Virtual Switch, preskoči na **Vodič 02**.

## 📖 Šta je Hyper-V?

Hyper-V je Microsoftov hypervisor koji omogućava pokretanje više operativnih sistema (virtualnih mašina) na jednom fizičkom računaru. Svaka VM koristi dodijeljene resurse hosta: CPU, RAM, virtualni disk i mrežni adapter.

## ✅ 1. Provjera Windows izdanja

Hyper-V je dostupan na izdanjima **Windows 10/11 Pro i Enterprise**. Home izdanje ne podržava punu Hyper-V platformu.

Provjera verzije:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
```

## 🔍 2. Provjera hardverske podrške

Otvori Command Prompt i pokreni:

```cmd
systeminfo
```

U sekciji **Hyper-V Requirements** sve četiri stavke trebaju imati vrijednost `Yes`:

- VM Monitor Mode Extensions
- Virtualization Enabled in Firmware
- Second Level Address Translation
- Data Execution Prevention Available

> ℹ️ Ako je Hyper-V već aktivan, Windows prikazuje poruku `A hypervisor has been detected`. To nije greška, nego znači da je hypervisor već pokrenut.

### Uključivanje virtualizacije u BIOS/UEFI

Ako `Virtualization Enabled in Firmware` prikazuje `No`:

1. Restartuj računar i uđi u BIOS/UEFI (obično `F2`, `F10`, `F1` ili `Del`, zavisno od proizvođača).
2. Uključi opciju **Intel Virtualization Technology (VT-x)** ili **SVM Mode / AMD-V**.
3. Sačuvaj promjene, restartuj računar i ponovi `systeminfo`.

## ⚙️ 3. Aktiviranje Hyper-V

Otvori PowerShell kao **administrator** i izvrši:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Potvrdi restart kada Windows to zatraži.

> 💡 Alternativa: `Turn Windows features on or off` → označi **Hyper-V** → OK.
> Na Windows Server sistemima Hyper-V se instalira kao server role.

## 🧪 4. Provjera instalacije

Nakon restarta:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
Get-Service vmms
```

Očekivano:

```text
State  : Enabled
Status : Running
```

Dodatno, otvori **Hyper-V Manager** (Start → Hyper-V Manager). U lijevom panelu treba se pojaviti lokalni računar.

## 🌐 5. Virtual Switch

Virtual Switch povezuje virtualne mašine s mrežom.

| Tip | Opis |
|:---|:---|
| **External** | VM dobija pristup fizičkoj mreži preko fizičkog adaptera |
| **Internal** | Komunikacija između hosta i VM-ova, bez direktnog pristupa fizičkoj mreži |
| **Private** | Komunikacija isključivo između VM-ova |

### Opcija A: Default Switch (najbrži početak)

Hyper-V najčešće automatski kreira `Default Switch`, koji VM-ovima daje pristup internetu putem NAT-a. Za početni lab to je sasvim dovoljno.

### Opcija B: External Switch (preporučeno za lab s agentima)

1. Hyper-V Manager → **Virtual Switch Manager**.
2. Izaberi **External** → **Create Virtual Switch**.
3. Unesi naziv `External-Switch` i izaberi fizički mrežni adapter.
4. Ostavi uključeno `Allow management operating system to share this network adapter`.
5. Klikni **Apply** → **OK**.

> ⚠️ Kreiranje External Switcha može kratko prekinuti mrežnu konekciju hosta.

> 💡 Za Wazuh lab u kojem će agenti komunicirati s manager VM-om preporučuje se External Switch, jer VM dobija IP adresu iz tvoje mreže.

## 🏁 Rezultat

Nakon ovog vodiča imamo:

- Potvrđenu podršku za virtualizaciju.
- Aktivan Hyper-V.
- Funkcionalan Virtual Switch.

➡️ **Sljedeći korak:** Vodič 02 – Instalacija CentOS Stream 10 na Hyper-V.

---

## 📚 Službeni izvori

- [Microsoft Learn – Install Hyper-V](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/install-hyper-v)
- [Microsoft Learn – Hyper-V system requirements](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)

**🛡️ Wazuh BA – Uči. Implementiraj. Testiraj. Zaštiti.**