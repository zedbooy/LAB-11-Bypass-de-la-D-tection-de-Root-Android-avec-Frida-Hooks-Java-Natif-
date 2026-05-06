# LAB-11-Bypass-de-la-D-tection-de-Root-Android-avec-Frida-Hooks-Java-Natif-
<img width="1209" height="570" alt="image" src="https://github.com/user-attachments/assets/34de71ab-7f8b-4854-a5f9-324fca4b9991" />

## Description
Ce laboratoire porte sur l'utilisation de Frida pour neutraliser dynamiquement les mécanismes de détection de root (Root Detection) au sein d'une application Android, en ciblant à la fois la couche Java et la couche native (C/C++).

## Fichiers du Projet
- `bypass_root.js` : Script Frida pour intercepter les vérifications Java (Build.TAGS, RootBeer, File.exists, Runtime.exec).
- `bypass_native.js` : Script Frida pour neutraliser les appels système natifs (open, access, stat, etc.) via le NDK.
- `anti_frida.js` : Script Frida pour masquer la présence de Frida lui-même (variables d'environnement, ports).

## Contenu des Scripts Frida

### 1. bypass_root.js (Couche Java)
Le script cible quatre vecteurs principaux de détection Java :
- **Build.TAGS** : Force la valeur à `release-keys`.
- **RootBeer** : Hook les méthodes `isRooted` et `isRootedWithBusyBoxCheck`.
- **File.exists** : Intercepte les accès aux fichiers suspects (binaires `su`, `busybox`).
- **Runtime.exec** : Bloque l'exécution de commandes liées au root.

### 2. bypass_native.js (Couche Native)
Le script cible les appels système NDK :
- **open/openat** : Empêche l'ouverture de fichiers système sensibles.
- **access/stat/lstat** : Falsifie les tests d'existence et de permissions.
- **Filtre /proc/mounts** : Masque les partitions montées en RW.

### 3. anti_frida.js (Masquage Frida)
- **Variables d'environnement** : Masque les variables contenant "frida".
- **Détection de ports** : Bloque les connexions aux ports 27042/27043.

## Utilisation

### Lancement Combiné (Recommandé)
```bash
frida -U -f com.example.rootcheck -l bypass_root.js -l bypass_native.js -l anti_frida.js --no-pause
```

### Méthodes Alternatives
- **Injection dès le démarrage** : `frida -U -f com.package.name -l script.js --no-pause`
- **Attachement** : `frida -U -n "NomDuProcessus" -l script.js`
- **Traçage** : `frida-trace -U -i open -i access com.package.name`
