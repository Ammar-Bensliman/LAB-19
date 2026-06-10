# LAB-19
markdown_content = """# LAB-19 : Exploitation SnakeYAML sur Android avec Patching Smali

> **Avertissement de sécurité :** Ce lab est réalisé dans un environnement de test contrôlé à des fins strictement pédagogiques. L'application cible est `Snake.apk`, un binaire conçu pour illustrer les vulnérabilités de désérialisation et les mécanismes de protection applicative.

## 📝 Contexte et Objectifs du Lab
Sous ses apparences d'application ordinaire, `Snake.apk` intègre des protections contre le rootage et une vulnérabilité critique de **désérialisation non sécurisée** liée à l'utilisation de la bibliothèque **SnakeYAML**. 

Le scénario complet de ce laboratoire consiste à :
1. **Analyser** la logique de détection de root dans la couche Java.
2. **Décompiler** l'APK et **patcher** son bytecode Smali pour neutraliser cette protection.
3. **Recompiler, signer et installer** l'application modifiée.
4. **Comprendre** le fonctionnement d'un "gadget" de désérialisation SnakeYAML via la classe `BigBoss`.
5. **Forger** un payload YAML malveillant et déclencher son exécution via un *Intent* Android ciblé.
6. **Intercepter** les logs système avec `logcat` pour extraire le flag final.

---

## 🛠️ Environnement et Boîte à Outils

| Outil / Élément | Rôle / Description |
| :--- | :--- |
| **Android Emulator** | Environnement d'exécution cible (Android Rooté). |
| **ADB (Android Debug Bridge)** | Communication, transfert de fichiers et gestion des permissions. |
| **Jadx-GUI** | Décompilateur pour l'analyse statique du code Java. |
| **Apktool** | Décompilation de l'APK en bytecode Smali et recompilation. |
| **Uber-APK-Signer** | Alignement et signature cryptographique de l'APK modifié (Debug certificate). |
| **VS Code / Éditeur** | Modification textuelle des fichiers Smali. |
| **PowerShell + Logcat** | Capture des flux de logs système en temps réel. |

---

## 🚀 Guide Opérationnel Étape par Étape

### Étape 1 — Analyse statique de la couche Java avec Jadx-GUI
Ouvrez `Snake.apk` dans Jadx-GUI. Le package racine est `com.pwnsec.snake`. Deux classes principales structurent l'application : `MainActivity` et `BigBoss`.

Dans `MainActivity`, on constate immédiatement l'intégration d'une bibliothèque native :
