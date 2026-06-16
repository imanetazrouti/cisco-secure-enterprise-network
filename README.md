# Simulation d'une Infrastructure Réseau Multi-Site Sécurisée

Ce projet présente la conception et la simulation complète d'une infrastructure réseau d'entreprise en pleine croissance sous **Cisco Packet Tracer**. L'architecture intègre des mécanismes de haute disponibilité (BGP multi-homing), de routage inter-VLAN, de services centralisés et une sécurité renforcée par un pare-feu dédié (Cisco ASA).

## 🚀 Architecture de la Topologie

L'infrastructure est découpée en 4 grandes zones :

1. **Le Siège Social (HQ) :** Utilise une architecture hiérarchique avec un switch multi-couches (Layer 3) centralisant les départements (R&D, Administration, RH, Invités), gérant le routage inter-VLAN et agissant comme agent de relais DHCP.
2. **La Zone DMZ & Bordure :** Protégée par un pare-feu Cisco ASA 5506-X hébergeant les serveurs de production (Web, FTP, DNS/DHCP) et un routeur de bordure (Edge Router).
3. **La Succursale (Branch Office) :** Une structure plus légère connectée en WAN, utilisant une topologie de type *Router-on-a-Stick* pour segmenter le Staff et les Invités.
4. **Le Cœur Internet (WAN Cloud) :** Simulation d'une redondance de fournisseurs d'accès Internet (ISP) via le protocole de routage dynamique **BGP** avec deux opérateurs majeurs (Maroc Telecom et Orange).

---

## 🗺️ Plan d'Adressage IP

L'ensemble du réseau a été segmenté afin d'optimiser l'espace d'adressage IP (VLSM) :

### Siège Social (HQ) & DMZ
| Service / Équipement | VLAN | Adresse Réseau | Masque | Passerelle (SVI) | Plage d'IPs utiles |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **R&D** | 10 | `10.1.0.0/25` | `255.255.255.128` | `10.1.0.1` | `10.1.0.2 - 10.1.0.126` |
| **Invités HQ** | 40 | `10.1.0.128/25` | `255.255.255.128` | `10.1.0.129` | `10.1.0.130 - 10.1.0.254` |
| **Administration** | 20 | `10.1.1.0/26` | `255.255.255.192` | `10.1.1.1` | `10.1.1.2 - 10.1.1.62` |
| **Ressources Humaines** | 30 | `10.1.1.64/27` | `255.255.255.224` | `10.1.1.65` | `10.1.1.66 - 10.1.1.194` |
| **Zone DMZ** | 50 | `10.1.1.96/28` | `255.255.255.240` | `10.1.1.97` | `10.1.1.98 - 10.1.1.110` |

### Succursale (Branch)
| Service / Équipement | VLAN | Adresse Réseau | Masque | Passerelle (Sous-int) | Plage d'IPs utiles |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **Staff Branch** | 110 | `10.2.0.0/26` | `255.255.255.192` | `10.2.0.1` | `10.2.0.2 - 10.2.0.62` |
| **Invités Branch** | 120 | `10.2.0.64/27` | `255.255.255.224` | `10.2.0.65` | `10.2.0.66 - 10.2.0.94` |

### Liaisons WAN & Inter-Équipements
| Liaison | Adresse Réseau | Masque | Interface Côté A / IP | Interface Côté B / IP |
| :--- | :--- | :--- | :--- | :--- |
| **Edge Router — ISP-A** | `200.10.10.0/30` | `255.255.255.252` | Edge G0/0/0 (`200.10.10.2`) | ISP-A (`200.10.10.1`) |
| **Edge Router — ISP-B** | `200.20.20.0/30` | `255.255.255.252` | Edge G0/0/1 (`200.20.20.2`) | ISP-B (`200.20.20.1`) |
| **Branch Router — ISP-A** | `200.30.30.0/30` | `255.255.255.252` | Branch G0/0/1 (`200.30.30.2`) | ISP-A (`200.30.30.1`) |
| **Branch Router — ISP-B** | `200.40.40.0/30` | `255.255.255.252` | Branch G0/0/2 (`200.40.40.2`) | ISP-B (`200.40.40.1`) |
| **Switch L3 — ASA (Inside)**| `10.1.254.0/30` | `255.255.255.252` | Switch L3 (`10.1.254.1`) | ASA Inside (`10.1.254.2`) |
| **ASA — Edge Router (Outside)**| `10.1.255.0/30` | `255.255.255.252` | ASA Outside (`10.1.255.1`) | Edge Router (`10.1.255.2`) |

---

## 🛡️ Politique de Sécurité (Cisco ASA)

Le pare-feu Cisco ASA 5506-X assure le cloisonnement des flux en appliquant des niveaux de sécurité stricts (*Security Levels*) pour protéger les données internes :

*   **Zone Inside (Niveau 100) :** Zone de confiance maximale contenant le Switch L3 et les VLANs utilisateurs (R&D, Admin, RH).
*   **Zone DMZ (Niveau 50) :** Zone intermédiaire isolant les serveurs de production accessibles (Web, FTP, DNS/DHCP) des menaces directes.
*   **Zone Outside (Niveau 0) :** Zone non fiable connectée à l'Edge Router et tournée vers l'Internet (ISP-A et ISP-B).

---

## 🛠️ Configuration des Services Réseau

Les services applicatifs essentiels de l'entreprise ont été centralisés sur des serveurs dédiés situés en DMZ :

*   **Serveur Web (HTTP/HTTPS) :** `10.1.1.98` (Hébergement du site de l'entreprise).
*   **Serveur de Partage (FTP) :** `10.1.1.99` (Stockage et échange de fichiers).
*   **Serveur Central (DNS / DHCP) :** `10.1.1.100`
    *   *DNS :* Résolution de noms pour l'infrastructure.
    *   *DHCP :* Distribution dynamique des configurations IP via les pools configurés :
        *   `Pool_VLAN10` (R&D HQ)
        *   `Pool_VLAN20` (Admin HQ)
        *   `Pool_VLAN30` (RH HQ)
        *   `Pool_VLAN110` (Staff Branch)

---

## 📂 Contenu du Dépôt

*   `Projet Réseau.pkt` : Le fichier de simulation Cisco Packet Tracer (prêt à être exécuté).
*   `topologie.png` : Capture d'écran du schéma de la topologie réseau.
*   `/configs` : Dossier contenant les sauvegardes textuelles des configurations (`running-config`) des routeurs, du commutateur de niveau 3 et du pare-feu ASA.

## 💻 Instructions de Test
1. Télécharger et installer **Cisco Packet Tracer**.
2. Ouvrir le fichier `Projet Réseau.pkt`.
3. Lancer des tests de connectivité (Ping / Traceroute) entre la Succursale et le Siège Social pour valider le routage BGP.
4. Tenter d'accéder au serveur Web (`10.1.1.98`) depuis un PC client pour vérifier les politiques d'accès de la DMZ.
