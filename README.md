# 🔧 Lab 1 — Configure, Verify and Troubleshoot IPv4 Addresses

![Cisco](https://img.shields.io/badge/Cisco-CCNA-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/101%20Labs-Lab%201-purple?style=for-the-badge)
![IPv4](https://img.shields.io/badge/IPv4-Addressing-red?style=for-the-badge)

---

## 📋 Description

Premier lab du livre **101 Labs Cisco CCNA (200-301)**.
L'objectif est de configurer, vérifier et dépanner des
adresses **IPv4** sur des routeurs Cisco avec des interfaces
**Serial** et **Loopback**.

### Objectifs
- ✅ Configurer les **hostnames** sur R1 et R3
- ✅ Configurer l'interface **Serial S0/0** (DCE) sur R1
- ✅ Configurer les interfaces **Loopback** sur R1 et R3
- ✅ Vérifier la configuration avec les commandes **show**
- ✅ Dépanner les adresses IPv4

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | Cisco 2911 | R1 | Routeur DCE |
| 🔀 Routeur | Cisco 2911 | R3 | Routeur DTE |

---

## 🗺️ Topologie

```
[R1]────S0/0 (DCE)────172.16.1.2/26────S0/0────[R3]
172.16.1.1/26
```

<img width="1920" height="1080" alt="Lab01-IPv4-Configure-Verify-Troubleshoot" src="https://github.com/user-attachments/assets/447c1d81-78ab-47ad-8b8d-73205b9e8599" />


---

## 📊 Plan d'adressage

### Lien Serial R1 ↔ R3

| Interface | IP | Masque | Routeur |
|-----------|-----|--------|---------|
| S0/0 (DCE) | 172.16.1.1 | /26 (255.255.255.192) | R1 |
| S0/0 | 172.16.1.2 | /26 (255.255.255.192) | R3 |

### Interfaces Loopback R3

| Interface | IP | Masque |
|-----------|-----|--------|
| Loopback10 | 10.10.10.3 | /25 (255.255.255.128) |
| Loopback20 | 10.20.20.3 | /28 (255.255.255.240) |
| Loopback30 | 10.30.30.3 | /29 (255.255.255.248) |

---

## ⚙️ Configuration complète

### 🔧 Task 1 — Hostnames

```cisco
! Sur R1
enable
configure terminal
hostname R1
end

! Sur R3
enable
configure terminal
hostname R3
end
```

---

### 🔧 Task 2 — Interface Serial + Loopback

#### Configuration R1

```cisco
enable
configure terminal
hostname R1

! Interface Serial S0/0 (DCE)
! DCE = fournit le clock rate à R3
interface serial 0/0
ip address 172.16.1.1 255.255.255.192
clock rate 768000
no shutdown
exit

end
write
```

#### Configuration R3

```cisco
enable
configure terminal
hostname R3

! Interface Serial S0/0 (DTE)
interface serial 0/0
ip address 172.16.1.2 255.255.255.192
no shutdown
exit

! Interface Loopback 10
interface loopback 10
ip address 10.10.10.3 255.255.255.128
exit

! Interface Loopback 20
interface loopback 20
ip address 10.20.20.3 255.255.255.240
exit

! Interface Loopback 30
interface loopback 30
ip address 10.30.30.3 255.255.255.248
exit

end
write
```

---

### 🔧 Task 3 — Commandes de vérification

#### 1. Résumé de toutes les IPs configurées

```cisco
R1# show ip interface brief
R3# show ip interface brief
```

**Résultat attendu R1 :**
```
Interface    IP-Address      OK? Method Status   Protocol
Serial0/0    172.16.1.1      YES manual up        up
```

**Résultat attendu R3 :**
```
Interface    IP-Address      OK? Method Status   Protocol
Serial0/0    172.16.1.2      YES manual up        up
Loopback10   10.10.10.3      YES manual up        up
Loopback20   10.20.20.3      YES manual up        up
Loopback30   10.30.30.3      YES manual up        up
```

---

#### 2. Statut de l'interface (up/down)

```cisco
R1# show interfaces serial 0/0
R3# show interfaces serial 0/0
```

**Résultat attendu :**
```
Serial0/0 is up, line protocol is up ✅
  Internet address is 172.16.1.1/26
  ...
  DTE mode               ← R3 est DTE
  DCE mode, clock rate 768000  ← R1 est DCE
```

---

#### 3. Masque de sous-réseau appliqué

```cisco
R1# show ip interface serial 0/0
R3# show ip interface loopback 10
```

**Résultat attendu :**
```
Serial0/0 is up, line protocol is up
  Internet address is 172.16.1.1/26
  Broadcast address is 255.255.255.255
  ...
```

---

#### 4. Vérification complète

```cisco
! Voir la configuration complète
R1# show running-config
R3# show running-config

! Voir les routes
R1# show ip route
R3# show ip route

! Tester la connectivité
R1# ping 172.16.1.2
R3# ping 172.16.1.1
```

---

## 🧪 Tests de connectivité

```
✅ R1# ping 172.16.1.2    (R1 → R3 via Serial)
✅ R3# ping 172.16.1.1    (R3 → R1 via Serial)
✅ R3# ping 10.10.10.3    (Loopback10)
✅ R3# ping 10.20.20.3    (Loopback20)
✅ R3# ping 10.30.30.3    (Loopback30)
```

---

## 🔍 Calcul des sous-réseaux

| Réseau | Masque | Nb hôtes | Plage IPs |
|--------|--------|----------|-----------|
| 172.16.1.0**/26** | 255.255.255.192 | 62 | .1 → .62 |
| 10.10.10.0**/25** | 255.255.255.128 | 126 | .1 → .126 |
| 10.20.20.0**/28** | 255.255.255.240 | 14 | .1 → .14 |
| 10.30.30.0**/29** | 255.255.255.248 | 6 | .1 → .6 |

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `clock rate 768000` | Configure le DCE (fournit horloge) |
| `no shutdown` | Active l'interface |
| `show ip interface brief` | Vue rapide de toutes les IPs |
| `show interfaces s0/0` | Détails d'une interface |
| `show ip interface s0/0` | Détails IP d'une interface |
| `show running-config` | Configuration complète |
| `interface loopback X` | Crée une interface virtuelle |

---

## 📖 Notions importantes

### DCE vs DTE
```
DCE (Data Communications Equipment)
→ Fournit le signal d'horloge (clock rate)
→ Commande : clock rate 768000
→ C'est R1 dans ce lab

DTE (Data Terminal Equipment)
→ Reçoit le signal d'horloge
→ Pas de clock rate nécessaire
→ C'est R3 dans ce lab
```

### Interface Loopback
```
→ Interface virtuelle toujours UP
→ Utilisée pour les tests
→ Utilisée pour OSPF Router-ID
→ Ne nécessite pas de câble physique
→ Toujours disponible même si
  les autres interfaces tombent
```

---

## 🛠️ Outils

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-8.x-orange?style=flat-square&logo=cisco)
![GNS3](https://img.shields.io/badge/GNS3-2.x-green?style=flat-square)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
