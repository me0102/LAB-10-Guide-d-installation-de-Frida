# Compte Rendu de Lab : Installation et Utilisation de Frida pour la Sécurité Mobile

**Étudiante :** Bouchra Ouzanzoul  
**Formation :** 2ème année cycle ingénieur - CyberSécurité et Télécommunications (ENSA Marrakech)  
**Cours :** Sécurité des applications mobiles  
**Date :** 27 Avril 2026

---

## Table des Matières

- [1. Objectifs du Lab](#1-objectifs-du-lab)
- [2. Prérequis et Environnement](#2-prérequis-et-environnement)
- [3. Déroulement du Lab](#3-déroulement-du-lab)
  - [3.1. Installation du Client Frida](#31-installation-du-client-frida)
  - [3.2. Déploiement du frida-server sur Android](#32-déploiement-du-frida-server-sur-android)
  - [3.3. Tests de Connexion et Injections Minimales](#33-tests-de-connexion-et-injections-minimales)
- [4. Exploration Avancée (Analyse de Sécurité)](#4-exploration-avancée-analyse-de-sécurité)
  - [4.1. Console Interactive](#41-console-interactive)
  - [4.2. Instrumentation et Hooking](#42-instrumentation-et-hooking)
- [5. Exercices Pratiques et Dépannage](#5-exercices-pratiques-et-dépannage)
  - [5.1. Preuves de fonctionnement](#51-preuves-de-fonctionnement)
  - [5.2. Cas de Dépannage rencontré](#52-cas-de-dépannage-rencontré)
- [Conclusion](#conclusion)

---

## 1. Objectifs du Lab

L'objectif de ce laboratoire était de mettre en place un environnement d'analyse dynamique complet pour applications Android en utilisant le framework Frida. Les étapes clés incluaient :

- L'installation du client Frida sur un poste de travail Windows.
- Le déploiement et la configuration de frida-server sur un appareil Android (émulateur).
- La validation de l'environnement par l'injection de scripts JavaScript (Java et natifs).
- L'exploration des capacités d'instrumentation pour l'analyse de sécurité (réseau, fichiers, stockage local).

---

## 2. Prérequis et Environnement

| Composant | Détails |
|-----------|---------|
| **Machine hôte** | Windows 10 (Version 10.0.26200) |
| **Appareil cible** | Émulateur Android (architecture x86_64) |
| **Outils** | Python 3.9.0, ADB (Android Debug Bridge version 1.0.41) |
| **Frida version** | 17.9.1 |

---

## 3. Déroulement du Lab

### 3.1. Installation du Frida

L'installation a été réalisée via le gestionnaire de paquets Python pip. La vérification a confirmé la version 17.9.1 tant au niveau de l'exécutable CLI que de la bibliothèque Python.
<img width="958" height="890" alt="Capture d&#39;écran 2026-04-27 082231" src="https://github.com/user-attachments/assets/648e6599-65de-4df8-95cf-5357ceb22b51" />

<img width="955" height="101" alt="Capture d&#39;écran 2026-04-27 082247" src="https://github.com/user-attachments/assets/2f344c84-32b7-4063-a93f-feb0b8fb92e1" />

<img width="1459" height="627" alt="Capture d&#39;écran 2026-04-27 082039" src="https://github.com/user-attachments/assets/eb21e60b-e19f-4a02-9d33-9945fb4daa46" />


### 3.2. Déploiement du frida-server sur Android

- **Identification de l'architecture :** La commande `adb shell getprop ro.product.cpu.abi` a retourné `x86_64`.
  
  <img width="673" height="80" alt="Capture d&#39;écran 2026-04-27 083003" src="https://github.com/user-attachments/assets/76aea72f-6f43-4552-a1a9-99fce069840f" />

- **Transfert et droits :** Le binaire `frida-server-17.9.1-android-x86_64` a été poussé vers `/data/local/tmp/` via ADB, puis rendu exécutable avec `chmod 755`.

  <img width="952" height="101" alt="Capture d&#39;écran 2026-04-27 084409" src="https://github.com/user-attachments/assets/dc229e09-37ad-4071-b021-4d1dcbed936a" />

- **Lancement :** Le serveur a été lancé en arrière-plan via `nohup`.

<img width="1057" height="409" alt="Capture d&#39;écran 2026-04-27 093045" src="https://github.com/user-attachments/assets/73333872-1386-470c-a133-6fd5ed04deba" />

- **Redirection de ports :** Les ports 27042 et 27043 ont été redirigés pour assurer la communication entre l'hôte et l'appareil.

<img width="1056" height="276" alt="Capture d&#39;écran 2026-04-27 093156" src="https://github.com/user-attachments/assets/8d3a22da-ca25-4b39-bf0a-12592ee5e187" />

### 3.3. Tests de Connexion et Injections Minimales

La validation a été effectuée en listant les processus actifs avec `frida-ps -D emulator-5556`.

- **Injection Java (hello.js) :** Le script a été injecté dans Chrome (`com.android.chrome`). Après l'utilisation de `%resume`, la console a affiché `[+] Frida Java.perform OK`.

  <img width="1060" height="575" alt="Capture d&#39;écran 2026-04-27 094148" src="https://github.com/user-attachments/assets/4fa8b52d-6b98-495b-bf02-6132234b1029" />

- **Hook Natif (hello_native.js) :** Tentative d'interception de la fonction `recv` dans la bibliothèque système. Le script a été chargé avec succès (`[+] Script chargé`).

<img width="939" height="428" alt="Capture d&#39;écran 2026-04-27 094921" src="https://github.com/user-attachments/assets/0e0b3de9-d172-4f97-a2d1-699c5d012fcb" />

---

<img width="947" height="781" alt="Capture d&#39;écran 2026-04-27 093319" src="https://github.com/user-attachments/assets/9b1e1231-2107-435a-be37-cb69b625e6e8" />


## 4. Exploration Avancée (Analyse de Sécurité)

### 4.1. Console Interactive

L'utilisation de la console interactive a permis d'extraire des informations critiques du processus Chrome :

- **Architecture :** x64.
- **Informations système :

<img width="826" height="186" alt="Capture d&#39;écran 2026-04-27 100412" src="https://github.com/user-attachments/assets/243ba8dc-8845-43e2-b4ac-5eacc2a936a4" />

- **Modules chargés :** Identification de bibliothèques sensibles telles que `libcrypto.so` et `libssl.so` via `Process.enumerateModules()`.

<img width="1057" height="838" alt="Capture d&#39;écran 2026-04-27 100329" src="https://github.com/user-attachments/assets/2b3a2b7d-4c37-483b-b3fb-28d8fa268007" />

<img width="1056" height="914" alt="Capture d&#39;écran 2026-04-27 100458" src="https://github.com/user-attachments/assets/3b102958-1385-46ff-8a0f-2e5dfb020027" />

- **Énumération Java :** Localisation des adresses mémoires de `connect`, `send` et `recv` dans la `libc.so`.

### 4.2. Instrumentation et Hooking

Plusieurs scripts avancés ont été déployés pour surveiller les vecteurs d'attaque courants :

| Vecteur | Cible | Données observées |
|---------|-------|------------------|
| **Fichiers** | Appels `open` et `read` | Descripteurs de fichiers accédés par l'application en temps réel |
| **Stockage local** | Classe `SharedPreferencesImpl` | Lecture et écriture des préférences (clés/valeurs) |
| **Base de données** | `SQLiteDatabase.execSQL` | Surveillance des requêtes SQLite |

---

## 5. Exercices Pratiques et Dépannage

### 5.1. Preuves de fonctionnement

- **Version Frida :** 17.9.1
- **Appareils connectés :** `emulator-5556 device`
- **Sortie d'injection :** Succès du `Java.perform` validé par message console.

### 5.2. Cas de Dépannage rencontré

Un problème majeur de privilèges a été diagnostiqué lors des premières tentatives d'injection (`unable to access process`).

- **Diagnostic :** Le frida-server tournait avec l'utilisateur `shell`, insuffisant pour instrumenter des processus tiers.
- **Correction :** Exécution de `adb root` suivi du redémarrage du serveur. L'usage de l'option `-D emulator-5556` a également permis de lever l'ambiguïté en présence de plusieurs instances d'émulateurs détectées par ADB.

---

## Conclusion

Ce lab a permis de valider la chaîne complète d'instrumentation dynamique avec Frida. La capacité à intercepter aussi bien des méthodes Java de haut niveau que des fonctions système natives constitue un atout majeur pour l'analyse de vulnérabilités, le reverse-engineering et le contournement de mécanismes de protection (Root detection, SSL Pinning) dans un cadre de cybersécurité mobile.
