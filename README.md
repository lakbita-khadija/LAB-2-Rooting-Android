# LAB 2 : Rooting Android
**Cours : Sécurité des applications mobiles**

---

## 1. Fiche Périmètre

| Champ | Détail |
|---|---|
| **Application** | Pink Timer App (`com.example.pinktimerapp`) — v1.0 |
| **Support** | AVD Android Studio — Pixel 5 (émulateur) |
| **Objectif** | Comprendre le rooting Android et ses impacts sécurité |
| **Données** | Fictives uniquement |
| **Réseau** | Réseau de test isolé |

---

## 2. Environnement Technique

| Paramètre | Valeur |
|---|---|
| **Date** | 26/04/2026 |
| **Auteur** | [Ton nom] |
| **Support** | AVD Pixel 5 — Android Studio |
| **Version Android** | 17 |
| **API Level** | 37 |
| **Application** | Pink Timer App (`com.example.pinktimerapp`) v1.0 |
| **Outil ADB** | Android SDK Platform-Tools |
| **OS hôte** | Windows |

---

## 3. Partie Technique — Rooting de l'AVD

### 3.1 Lancement de l'émulateur

L'émulateur a été lancé avec le flag `-writable-system` pour permettre le montage du système en lecture/écriture :

```bash
emulator -avd Pixel_5 -writable-system -scale 0.7
```

### 3.2 Activation du mode root

```bash
.\adb root
# restarting adbd as root
```

```bash
.\adb remount
# Successfully disabled verity
# Remounted /system as RW
# Remounted /vendor as RW
# Remounted /product as RW
# Remounted /system_dlkm as RW
# Remounted /system_ext as RW
# Remount succeeded
```
<img width="932" height="335" alt="Capture d&#39;écran 2026-04-26 175232" src="https://github.com/user-attachments/assets/8de7518d-c1fa-4412-801b-35eee3ef1ff2" />

### 3.3 Vérification des privilèges root

```bash
.\adb shell id
# uid=0(root) gid=0(root) groups=0(root),...
```
<img width="943" height="162" alt="Capture d&#39;écran 2026-04-26 175243" src="https://github.com/user-attachments/assets/1e7e2b6b-1839-41e4-843f-e719ba45375d" />

>  `uid=0(root)` confirme que les privilèges root sont actifs.

### 3.4 Vérification de l'état Verified Boot

```bash
.\adb shell getprop ro.boot.verifiedbootstate
# orange

.\adb shell getprop ro.boot.veritymode
# enforcing

.\adb shell getprop ro.boot.vbmeta.device_state
# (vide — non défini sur cet émulateur)
```

| Propriété | Résultat | Signification |
|---|---|---|
| `ro.boot.verifiedbootstate` | `orange` | Système modifié, intégrité non garantie |
| `ro.boot.veritymode` | `enforcing` | Verity actif mais contourné via overlayfs |
| `ro.boot.vbmeta.device_state` | *(vide)* | Non défini sur cet émulateur |

> 🟠 L'état **orange** indique que le système a été modifié. C'est le résultat attendu sur un émulateur rooté.


<img width="946" height="322" alt="Capture d&#39;écran 2026-04-26 175510" src="https://github.com/user-attachments/assets/956dd4fa-cf0f-49f4-bdef-6a2838a2b7b1" />



### 3.5 Sauvegarde des logs

```bash
.\adb logcat -d > logcat_root_check.txt
# Fichier créé : 7 507 205 octets
```

<img width="937" height="406" alt="Capture d&#39;écran 2026-04-26 175458" src="https://github.com/user-attachments/assets/6eb4b867-19e8-4f17-b6c0-4ffd88415af5" />


---

## 4. Application de Test

### 4.1 Application choisie

**Pink Timer App** — application Foreground Service Android créée pour les labs.

```bash
.\adb shell monkey -p com.example.pinktimerapp -c android.intent.category.LAUNCHER 1
```

<img width="458" height="991" alt="Capture d&#39;écran 2026-04-26 190350" src="https://github.com/user-attachments/assets/e6039c9f-a68b-4163-8aa3-0cdf0bdc7a9a" />


### 4.2 Trois scénarios de test

| # | Scénario | Étapes | Résultat attendu |
|---|---|---|---|
| 1 | Lancer l'application | Ouvrir Pink Timer App | Écran principal affiché, timer à 00:00 |
| 2 | Démarrer le timer | Appuyer sur "DÉMARRER" | Timer se met en marche |
| 3 | Arrêter le timer | Appuyer sur "ARRÊTER" | Timer s'arrête et revient à 00:00 |

---

## 5. Sécurité Android

### 5.1 Résumé (6 lignes)

1. **Sandboxing** : chaque application s'exécute dans un environnement isolé, elle ne peut pas accéder aux données des autres apps.
2. **Modèle de permissions** : l'utilisateur doit autoriser explicitement l'accès aux ressources sensibles (caméra, GPS, contacts...).
3. **Intégrité système** : Android vérifie l'intégrité du système au démarrage pour détecter toute modification non autorisée.

### 5.2 Verified Boot

**Objectif principal** : Garantir que le système qui démarre est celui prévu par le fabricant, sans modifications malveillantes.

**Chain of trust** : Série de vérifications où chaque composant vérifie l'authenticité du suivant avant de lui faire confiance. Comme une chaîne de gardiens où chacun vérifie l'identité du suivant.

**Pourquoi c'est critique** : Si le démarrage est compromis, toutes les protections ultérieures peuvent être contournées.

```
ROM → Bootloader → Vérification signature → Boot → Vérification système → Android
```

**État sur notre AVD** :
```bash
ro.boot.verifiedbootstate = orange  # Système modifié
```

### 5.3 AVB (Android Verified Boot 2.0)

1. AVB est la version moderne de Verified Boot, plus flexible et plus robuste que la version précédente.
2. Il ajoute une vérification cryptographique de l'intégrité de chaque partition au démarrage du système.
3. Il intègre une protection anti-rollback qui empêche d'installer d'anciennes versions vulnérables du système.

---

## 6. Définition du Rooting

1. Le rooting consiste à obtenir les privilèges super-utilisateur (`uid=0/root`) sur un appareil Android.
2. Cela modifie les protections du système et supprime la confiance garantie par le fabricant.
3. En laboratoire, il est utile pour observer les comportements système normalement inaccessibles.
4. Il est risqué et nécessite un environnement isolé, une traçabilité complète et une remise à zéro finale.

**Intérêt en labo** : Un environnement privilégié permet d'observer des artefacts système normalement inaccessibles, d'analyser les comportements runtime de l'application à bas niveau, et de tester la robustesse du stockage face à un attaquant privilégié. *(Labo autorisé uniquement.)*

---

## 7. Matrice de Risques

| # | Risque | Description |
|---|---|---|
| 1 | Intégrité non garantie | Les conclusions sur la sécurité réelle de l'app peuvent être biaisées |
| 2 | Surface d'attaque accrue | Si l'appareil sort du labo, il est exposé à des menaces externes |
| 3 | Données sensibles exposées | Toute donnée réelle présente sur l'appareil peut être compromise |
| 4 | Instabilité système | Les tests peuvent devenir non reproductibles et les résultats incohérents |
| 5 | Mélange comptes perso/test | Risque de fuite d'informations personnelles |
| 6 | Mauvais nettoyage fin de séance | Persistance de données sensibles sur l'émulateur |
| 7 | Réseau non isolé | Effets involontaires possibles sur des systèmes externes |
| 8 | Traçabilité insuffisante | Impossible de reproduire ou d'auditer les tests effectués |

---

## 8. Mesures Défensives

| # | Mesure | Description |
|---|---|---|
| 1 | Réseau isolé | Éviter toute communication non contrôlée vers l'extérieur |
| 2 | Données fictives uniquement | Éliminer tout risque de fuite de données réelles |
| 3 | Device/AVD dédié | Réservé exclusivement aux tests de sécurité |
| 4 | Wipe en fin de séance | Ne laisser aucune trace après les tests |
| 5 | Journal de configuration | Assurer la reproductibilité des tests |
| 6 | Aucun compte personnel | Éviter tout mélange de données personnelles |
| 7 | Contrôle strict des APK | Limiter les risques liés aux applications installées |
| 8 | Horodatage + captures | Assurer une traçabilité complète des étapes |

---

## 9. OWASP MASVS — 2 Exigences

**STORAGE-1** : Les données sensibles (API keys, mots de passe, tokens) doivent être stockées de manière sécurisée en utilisant des fonctions de chiffrement appropriées.

**NETWORK-1** : Les communications réseau doivent utiliser TLS avec une configuration correcte et vérifier les certificats pour éviter les attaques de type Man-in-the-Middle.

---

## 10. OWASP MASTG — 2 Idées de Tests

**Test 1** : Vérifier si les fichiers de préférences partagées contiennent des informations sensibles en clair en les examinant directement dans :
```bash
.\adb shell ls /data/data/com.example.pinktimerapp/shared_prefs/
.\adb shell cat /data/data/com.example.pinktimerapp/shared_prefs/[fichier].xml
```

**Test 2** : Analyser les logs de l'application pour détecter des fuites d'informations sensibles pendant l'exécution :
```bash
.\adb logcat -d | findstr "com.example.pinktimerapp"
```

---

## 11. Rappel des Commandes Clés

```bash
# Vérification ADB
adb devices

# Rooting
adb root
adb remount

# Vérifications
adb shell id
adb shell getprop ro.boot.verifiedbootstate
adb shell getprop ro.boot.veritymode
adb shell getprop ro.boot.vbmeta.device_state

# Logs
adb logcat -d > logcat_root_check.txt

# Reset AVD
adb emu avd wipe-data
```

---

```

📸 *[Insérer screenshot écran initial Android après reset — preuve de remise à zéro]*

---

*Rapport réalisé dans le cadre du cours Sécurité des applications mobiles — LAB 2 : Rooting Android*
