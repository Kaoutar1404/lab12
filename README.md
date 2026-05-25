# Bypass Root Detection Android avec Medusa

## Introduction
Ce projet montre comment utiliser Medusa et Frida pour contourner les mécanismes de détection de root sur une application Android.

Objectifs :
- Installer et configurer Frida
- Connecter un appareil Android
- Utiliser Medusa pour injecter des hooks
- Neutraliser les vérifications root Java et natives
- Tester le comportement d’une application protégée

---

# Étape 1 — Préparer l’environnement Android et Frida

## Installer Frida sur le PC

bash
pip install --upgrade frida frida-tools


## Vérifier l’installation

bash
frida --version


bash
python -c "import frida; print(frida.__version__)"


---

## Vérifier ADB

bash
adb version


## Vérifier l’appareil connecté

bash
adb devices


Résultat attendu :
bash
device


Si :
bash
unauthorized


- Rebrancher le téléphone
- Autoriser le débogage USB

---

# Étape 2 — Installer et lancer frida-server

## Vérifier l’architecture CPU Android

bash
adb shell getprop ro.product.cpu.abi


## Pousser frida-server

bash
adb push frida-server /data/local/tmp/


## Donner les permissions

bash
adb shell chmod 755 /data/local/tmp/frida-server


## Lancer frida-server

bash
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"


---

## Redirection des ports

bash
adb forward tcp:27042 tcp:27042


bash
adb forward tcp:27043 tcp:27043


---

## Vérifier la connexion Frida

bash
frida-ps -Uai


Résultat attendu :
- Liste des applications Android

---

# Étape 3 — Installer Medusa

## Cloner Medusa

bash
git clone <URL_du_depot_Medusa>


## Entrer dans le dossier

bash
cd Medusa


## Installer les dépendances

bash
pip install -r requirements.txt


## Vérifier la CLI

bash
python medusa.py --help


ou

bash
medusa --help


---

# Étape 4 — Comprendre la détection de root

## Vérifications Java courantes

Les applications Android peuvent vérifier :
- Build.TAGS
- File.exists()
- Runtime.exec("su")
- RootBeer.isRooted()

## Vérifications natives (NDK)

Les applications peuvent utiliser :
- open()
- access()
- stat()
- openat()

pour détecter :
bash
/system/xbin/su
/system/bin/su
busybox


---

# Étape 5 — Lancer l’application avec Medusa

## Injection dès le démarrage

bash
python medusa.py --usb --spawn com.example.rootcheck --module root-bypass


ou

bash
medusa --usb --spawn com.example.rootcheck --module root-bypass


---

## Injection avec options

bash
medusa run --package com.example.rootcheck --feature root-bypass --usb --no-pause


---

# Étape 6 — Attacher à une application déjà ouverte

bash
medusa --usb --attach "NomDuProcessus" --module root-bypass


Utilisé lorsque :
- L’application crash au démarrage
- Les hooks sont détectés trop tôt

---

# Étape 7 — Validation du bypass

## Sans Medusa
- L’application affiche :
bash
Root detected


## Avec Medusa
- Les protections root sont bypassées
- L’application fonctionne normalement
- Les fonctionnalités bloquées deviennent accessibles

---

## Logs attendus

bash
Build.TAGS -> release-keys


bash
Blocked Runtime.exec: su


bash
File.exists bypass for /system/xbin/su


---

# Étape 8 — Dépannage

## Medusa ne détecte pas l’appareil

Vérifier :

bash
adb devices


## Vérifier frida-server

bash
adb shell ps | grep frida


## Mettre à jour Frida

bash
pip install -U frida frida-tools


---

## L’application crash

Essayer :
bash
--attach


au lieu de :
bash
--spawn


---

## Détection de Frida

Certaines applications détectent :
- Les ports Frida
- Les chaînes "frida"
- Les hooks Java

Solutions :
- Utiliser un module anti-Frida
- Désactiver certains modules
- Injecter progressivement les hooks

---

# Structure du projet

bash
/project/
 ├── Medusa/
 ├── scripts/
 │    ├── bypass_root.js
 │    ├── bypass_native.js
 │    └── anti_frida.js
 │
 ├── preuves/
 │    ├── screenshots/
 │    └── logs/
 │
 └── README.md


---

# Outils utilisés

| Outil | Description |
|---|---|
| Frida | Instrumentation dynamique Android |
| Medusa | Framework d’injection basé sur Frida |
| ADB | Android Debug Bridge |
| frida-server | Service Frida Android |
| RootBeer | Détection root Android |

---

# Conclusion

Cette méthode permet :
- D’analyser les protections Android
- De contourner certaines détections root
- D’étudier les mécanismes anti-tampering
- De tester la sécurité des applications Android
- De comprendre les hooks Java et natifs avec Frida et Medusa
