---
title: "Guide TOTP — FreeRADIUS, Google Authenticator et SSH"
lang: fr-FR
---

<style>
  /* ============================================================
     STYLE GLOBAL — Palette Vert Menthe Diginamic
     ============================================================ */

  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');

  :root {
    --primary: #2ED8A3;
    --primary-dark: #1EB88A;
    --primary-light: #E8FBF4;
    --primary-ultra-light: #F4FDF9;
    --text-dark: #1A1A2E;
    --text-medium: #3D3D5C;
    --text-light: #6B6B8D;
    --bg-white: #FFFFFF;
    --bg-gray: #F7F8FA;
    --border: #E5E7EB;
    --danger: #EF4444;
    --warning: #F59E0B;
    --success: #10B981;
    --info: #3B82F6;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    color: var(--text-dark);
    background: var(--bg-white);
    line-height: 1.7;
    font-size: 11pt;
  }

  /* ---- HERO SECTION ---- */
  .hero {
    background: linear-gradient(145deg, #FFFFFF 0%, var(--primary-ultra-light) 50%, var(--primary-light) 100%);
    color: var(--text-dark);
    border: 2px solid var(--primary);
    border-radius: 18px;
    padding: 30px;
    margin: 20px 0 30px 0;
    box-shadow: 0 10px 30px rgba(46, 216, 163, 0.15);
    position: relative;
    overflow: hidden;
  }

  .hero::before {
    content: '';
    position: absolute;
    top: -50px;
    right: -50px;
    width: 200px;
    height: 200px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(46,216,163,0.15) 0%, transparent 70%);
  }

  .hero h1 {
    margin: 0 0 12px 0;
    font-size: 1.8rem;
    font-weight: 800;
    color: var(--text-dark);
    line-height: 1.2;
  }

  .subtitle {
    color: var(--primary-dark);
    font-size: 1.05rem;
    margin: 0 0 10px 0;
    font-weight: 500;
  }

  .meta {
    color: var(--text-medium);
    margin: 0;
    font-size: 0.9em;
  }

  /* ---- ALERTES / CALLOUTS ---- */
  .callout {
    border-left: 5px solid var(--primary);
    background: var(--primary-ultra-light);
    padding: 16px 20px;
    border-radius: 8px;
    margin: 16px 0;
    color: var(--text-dark);
  }

  .callout.warn { border-left-color: var(--warning); background: #FFFBEB; color: #92400E; }
  .callout.bad { border-left-color: var(--danger); background: #FEF2F2; color: #991B1B; }
  .callout.ok { border-left-color: var(--success); background: #ECFDF5; color: #065F46; }

  /* ---- BADGES ---- */
  .badge {
    display: inline-block;
    padding: 3px 10px;
    border-radius: 50px;
    font-size: 0.78em;
    font-weight: 600;
  }

  .badge { background: var(--primary-light); color: var(--primary-dark); }
  .badge.warn { background: #FEF3C7; color: #92400E; }
  .badge.bad { background: #FEE2E2; color: #991B1B; }
  .badge.ok { background: #D1FAE5; color: #065F46; }

  /* ---- TABLEAUX ---- */
  .table-wrap table {
    width: 100%;
    border-collapse: collapse;
    margin: 20px 0;
    font-size: 0.9em;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 1px 4px rgba(0,0,0,0.06);
  }

  .table-wrap thead {
    background: linear-gradient(135deg, var(--primary-dark), var(--primary));
    color: white;
  }

  .table-wrap th {
    padding: 12px 16px;
    text-align: left;
    font-weight: 600;
    font-size: 0.9em;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .table-wrap td {
    padding: 10px 16px;
    border-bottom: 1px solid var(--border);
  }

  .table-wrap tbody tr:nth-child(even) { background: var(--primary-ultra-light); }
  .table-wrap tbody tr:hover { background: var(--primary-light); }

  /* ---- BLOCS CODE ---- */
  pre {
    background: #1A1A2E;
    color: #E8E8E8;
    padding: 20px 24px;
    border-radius: 8px;
    overflow-x: auto;
    margin: 16px 0;
    font-size: 0.85em;
    line-height: 1.6;
    border-left: 4px solid var(--primary);
  }

  code {
    font-family: 'JetBrains Mono', 'Fira Code', 'Courier New', monospace;
    background: var(--bg-gray);
    padding: 2px 6px;
    border-radius: 4px;
    font-size: 0.88em;
    color: var(--primary-dark);
  }

  pre code {
    background: none;
    color: inherit;
    padding: 0;
  }

  /* ---- TITRES ---- */
  h1 {
    font-size: 2em;
    font-weight: 800;
    color: var(--text-dark);
    border-bottom: 4px solid var(--primary);
    padding-bottom: 12px;
    margin: 50px 0 25px;
  }

  h2 {
    font-size: 1.5em;
    font-weight: 700;
    color: var(--primary-dark);
    margin: 35px 0 15px;
    padding-left: 16px;
    border-left: 5px solid var(--primary);
  }

  h3 {
    font-size: 1.2em;
    font-weight: 600;
    color: var(--text-dark);
    margin: 25px 0 12px;
  }

  h4 {
    font-size: 1.05em;
    font-weight: 600;
    color: var(--text-medium);
    margin: 18px 0 10px;
  }

  /* ---- PARAGRAPHES ---- */
  p { margin-bottom: 14px; text-align: justify; }

  /* ---- HR ---- */
  hr {
    border: 0;
    border-top: 2px solid var(--primary-light);
    margin: 28px 0;
  }

  /* ---- LISTS ---- */
  ul, ol {
    margin: 14px 0;
    padding-left: 24px;
  }

  li { margin-bottom: 8px; }

  /* ---- BLOCKQUOTE ---- */
  blockquote {
    border-left: 4px solid var(--primary);
    background: var(--primary-ultra-light);
    padding: 16px 20px;
    margin: 16px 0;
    border-radius: 8px;
    color: var(--text-medium);
  }
</style>

<div class="hero">
  <h1>Guide professionnel — Mise en place TOTP avec FreeRADIUS, Google Authenticator et SSH</h1>
  <p class="subtitle">Procédure complète, concrète et testée en laboratoire pour sécuriser un compte sensible avec MFA/TOTP</p>
  <p class="meta">Projet VulneZe — <a href="https://github.com/VulneZe/" target="_blank" style="color: var(--primary-dark); text-decoration: none; font-weight: 600;">https://github.com/VulneZe/</a></p>
</div>

<div class="callout ok">
<strong>💡 Alternative économique :</strong> Cette méthode basée sur TOTP (Time-based One-Time Password) constitue une excellente alternative pour les organisations et utilisateurs qui ne disposent pas du budget nécessaire pour l'acquisition de clés matérielles YubiKey ou d'autres solutions MFA payantes. Elle offre un niveau de sécurité comparable à un coût quasi nul, en utilisant simplement une application mobile gratuite et une configuration logicielle standard.
</div>

## 1. Objectif du document

Ce guide explique comment mettre en place **une authentification multifacteur TOTP** de manière **fonctionnelle, propre et professionnelle** avec :

- **FreeRADIUS** comme serveur AAA,
- **Google Authenticator** comme générateur de code TOTP,
- **PAM** comme mécanisme d'intégration système,
- **SSH** comme cas d'usage réel d'accès distant.

L'objectif est de montrer un **cas concret**, cohérent avec le scénario d'attaque du CHU Métropole :

- **T1078 — Valid Accounts** : usage frauduleux de comptes compromis,
- **T1021 — Remote Services** : accès distant aux systèmes via SSH/RDP/VPN.

<div class="callout ok">
<strong>Résultat attendu :</strong> un mot de passe seul ne suffit plus. L'utilisateur doit fournir <strong>son mot de passe + un code TOTP temporaire</strong> pour s'authentifier.
</div>

---

## 2. Architecture logique

```text
Utilisateur → SSH / VPN / WiFi sécurisé
              ↓
         FreeRADIUS
              ↓
            PAM
              ↓
   pam_google_authenticator
              ↓
   Vérification du code TOTP
              ↓
      Accept / Reject
```

### Rôle de chaque composant

<div class="table-wrap">
<table>
  <tr><th>Composant</th><th>Rôle</th></tr>
  <tr><td>FreeRADIUS</td><td>Serveur d'authentification centralisé. Reçoit les demandes d'accès et décide d'accepter ou non l'authentification.</td></tr>
  <tr><td>PAM</td><td>Couche intermédiaire utilisée par Linux pour brancher des mécanismes d'authentification.</td></tr>
  <tr><td>Google Authenticator</td><td>Génère un code TOTP à usage unique et à courte durée de vie.</td></tr>
  <tr><td>SSH</td><td>Cas d'usage réel pour sécuriser un accès administrateur distant.</td></tr>
</table>
</div>

---

## 3. Pré-requis

- Une machine Linux de lab (Debian/Ubuntu/Parrot).
- Accès root ou sudo.
- Le paquet SSH déjà présent.
- Une application mobile TOTP : Google Authenticator, Microsoft Authenticator, Aegis, etc.
- Un terminal de secours déjà ouvert lors des tests SSH pour éviter de se verrouiller.

<div class="callout warn">
<strong>Important :</strong> ce guide est prévu pour un <strong>environnement de TP/lab</strong>. En production, il faut ajouter gestion de secrets, supervision, journalisation centralisée, sauvegarde des fichiers de configuration et procédure de retour arrière.
</div>

---

## 4. Installation des paquets

### Commande

```bash
sudo apt update
sudo apt install -y freeradius freeradius-utils libpam-google-authenticator
```

### Explication

- `freeradius` : installe le serveur RADIUS.
- `freeradius-utils` : fournit des outils de test comme `radtest`.
- `libpam-google-authenticator` : permet à PAM d'utiliser les codes TOTP.

### Vérification

```bash
dpkg -l | grep -E 'freeradius|google-authenticator'
```

---

## 5. Démarrage et contrôle du service FreeRADIUS

### Commandes

```bash
sudo systemctl enable freeradius
sudo systemctl start freeradius
sudo systemctl status freeradius --no-pager
```

### Explication

- `enable` : démarre automatiquement au boot.
- `start` : lance le service immédiatement.
- `status` : vérifie si le service tourne.

### Résultat attendu

Le service doit être en état :

```text
active (running)
```

### Vérification approfondie de la configuration

Avant chaque redémarrage après modification, tester la validité de la conf :

```bash
sudo freeradius -CX
```

En cas de panne, utiliser le mode debug :

```bash
sudo systemctl stop freeradius
sudo freeradius -X
```

<div class="callout">
<strong>Pourquoi utiliser <code>-CX</code> ou <code>-X</code> ?</strong><br>
FreeRADIUS refuse de démarrer si un fichier est mal configuré. C'est la méthode la plus efficace pour repérer une erreur de syntaxe, un module non chargé ou un doublon de client.
</div>

---

## 6. Déclaration des utilisateurs RADIUS

### Fichier cible

```bash
sudo nano /etc/freeradius/3.0/users
```

### Ajouter les comptes de test

```conf
admin-chu Cleartext-Password := "AdminStrongPass!"
svc_backup Cleartext-Password := "BackupSecure123"
```

### Explication

Ces lignes déclarent deux utilisateurs connus par FreeRADIUS :

- `admin-chu` : compte d'administration,
- `svc_backup` : compte de service sensible.

`Cleartext-Password` indique le mot de passe attendu par le moteur d'authentification.

### Vérification

```bash
tail -n 10 /etc/freeradius/3.0/users
```

---

## 7. Déclaration du client RADIUS

### Fichier cible

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

### Client local minimal

```conf
client localhost {
    ipaddr = 127.0.0.1
    secret = testing123
    require_message_authenticator = no
}
```

### Explication

- `client localhost` : machine autorisée à parler au serveur RADIUS.
- `ipaddr` : adresse IP source autorisée.
- `secret` : secret partagé entre le client et le serveur.
- `require_message_authenticator = no` : acceptable en lab, à durcir en production.

<div class="callout bad">
<strong>Attention :</strong> il ne doit y avoir <strong>qu'un seul bloc <code>client localhost</code></strong>. Un doublon provoque l'erreur <code>Failed to add duplicate client localhost</code> et empêche le démarrage du service.
</div>

### Vérification des doublons

```bash
grep -n "client localhost" /etc/freeradius/3.0/clients.conf
```

Résultat attendu :

```text
49:client localhost {
317:client localhost_ipv6 {
```

---

## 8. Création des utilisateurs Linux

Le module `pam_google_authenticator` s'appuie sur un fichier par utilisateur. Les comptes doivent donc exister côté système.

### Commandes

```bash
sudo useradd -m admin-chu
sudo useradd -m svc_backup
sudo passwd admin-chu
sudo passwd svc_backup
```

### Mots de passe à définir

- `admin-chu` → `AdminStrongPass!`
- `svc_backup` → `BackupSecure123`

### Explication

- `useradd -m` crée le compte et son répertoire personnel.
- `passwd` définit le mot de passe du compte Linux.

### Vérification

```bash
getent passwd admin-chu
getent passwd svc_backup
```

---

## 9. Génération du secret TOTP par utilisateur

### Pour `admin-chu`

```bash
su - admin-chu
google-authenticator
```

### Réponses recommandées

```text
Do you want authentication tokens to be time-based? y
Do you want me to update your "/home/admin-chu/.google_authenticator" file? y
Do you want to disallow multiple uses of the same authentication token? y
Do you want to increase the original generation window? n
Do you want to enable rate-limiting? y
```

### Pour `svc_backup`

```bash
su - svc_backup
google-authenticator
```

### Explication

Cette commande :

- génère une **clé secrète TOTP**,
- affiche un **QR code** à scanner,
- crée le fichier `~/.google_authenticator`,
- propose des **codes de secours**.

### Vérification des fichiers générés

```bash
ls -l /home/admin-chu/.google_authenticator
ls -l /home/svc_backup/.google_authenticator
```

---

## 10. Activation de PAM dans FreeRADIUS

### Activer le module PAM

```bash
sudo ln -s /etc/freeradius/3.0/mods-available/pam /etc/freeradius/3.0/mods-enabled/
```

Si le lien existe déjà, la commande peut renvoyer une erreur bénigne. Vérifier avec :

```bash
ls -l /etc/freeradius/3.0/mods-enabled/pam
```

### Modifier le site `default`

```bash
sudo nano /etc/freeradius/3.0/sites-enabled/default
```

#### Dans le bloc `authorize { ... }`, ajouter :

```conf
pam
```

#### Dans le bloc `authenticate { ... }`, ajouter :

```conf
Auth-Type PAM {
    pam
}
```

### Explication

- Le bloc `authorize` dit à FreeRADIUS d'utiliser le module PAM pendant l'autorisation.
- Le bloc `authenticate` dit comment traiter les requêtes de type PAM.

### Vérification rapide

```bash
grep -n "authorize {" /etc/freeradius/3.0/sites-enabled/default
grep -n "authenticate {" /etc/freeradius/3.0/sites-enabled/default
```

---

## 11. Configuration PAM pour FreeRADIUS

### Fichier cible

```bash
sudo nano /etc/pam.d/radiusd
```

### Contenu recommandé pour le TP

```conf
auth required pam_google_authenticator.so
account required pam_permit.so
```

### Explication

- `pam_google_authenticator.so` : vérifie le code TOTP.
- `pam_permit.so` : accepte la partie `account` sans complexifier le TP.

<div class="callout warn">
<strong>Limite de cette implémentation de TP :</strong> ici, l'accent est mis sur la démonstration du MFA TOTP. En production, il faudrait un empilement PAM plus robuste, avec contrôle de compte, audit et politiques plus strictes.
</div>

---

## 12. Validation de la configuration et redémarrage

### Commandes

```bash
sudo freeradius -CX
sudo systemctl restart freeradius
sudo systemctl status freeradius --no-pager
```

### Résultat attendu

- Pas d'erreur dans `freeradius -CX`
- Service `active (running)`

### En cas d'erreur

```bash
journalctl -u freeradius.service -n 50 --no-pager
sudo freeradius -X
```

---

## 13. Tests RADIUS de validation

## 13.1 Test simple sans OTP

### Commande

```bash
radtest admin-chu AdminStrongPass! localhost 1812 testing123
```

### Objectif

Vérifier si l'authentification simple est encore acceptée.

### Résultat attendu après activation MFA

```text
Access-Reject
```

Si le résultat est `Access-Accept`, alors le MFA n'est pas encore réellement appliqué.

---

## 13.2 Test avec mauvais OTP

### Commande

```bash
radtest admin-chu 'AdminStrongPass!000000' localhost 1812 testing123
```

### Résultat attendu

```text
Access-Reject
```

### Interprétation

C'est un bon signe : cela prouve que le code TOTP est effectivement contrôlé.

---

## 13.3 Test avec bon OTP

### Commande

Remplacer `482913` par le code affiché sur l'application mobile au moment du test.

```bash
radtest admin-chu 'AdminStrongPass!482913' localhost 1812 testing123
```

### Résultat attendu

```text
Access-Accept
```

### Attention au caractère `!`

Le shell Bash interprète `!`. Pour éviter l'erreur `event not found`, il faut :

- soit utiliser des **apostrophes simples**,
- soit échapper le caractère avec `\!`.

Exemple valide :

```bash
radtest admin-chu 'AdminStrongPass!482913' localhost 1812 testing123
```

---

## 14. Test en mode debug pour prouver le fonctionnement réel

### Terminal 1

```bash
sudo systemctl stop freeradius
sudo freeradius -X
```

### Terminal 2

```bash
radtest admin-chu 'AdminStrongPass!482913' localhost 1812 testing123
```

### Ce qu'il faut observer

Dans le terminal debug, on doit voir :

- traitement de l'utilisateur `admin-chu`,
- passage par le module `pam`,
- décision finale `Access-Accept` ou `Access-Reject`.

### Intérêt

Ce mode permet de démontrer de façon professionnelle que :

- FreeRADIUS reçoit bien la requête,
- le module PAM est invoqué,
- le TOTP est réellement contrôlé.

---

## 15. Intégration réelle au service SSH

Cette partie transforme le MFA RADIUS/TOTP en **cas d'usage réel d'accès distant**.

<div class="callout bad">
<strong>Très important :</strong> garde toujours une session root ouverte avant de modifier SSH. Si la configuration est incorrecte, tu peux te bloquer hors du serveur.
</div>

### 15.1 Configurer PAM pour SSH

```bash
sudo nano /etc/pam.d/sshd
```

Ajouter en haut du fichier :

```conf
auth required pam_google_authenticator.so
```

### Explication

Cette ligne impose un TOTP lors de l'ouverture d'une session SSH.

---

### 15.2 Configurer le démon SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Paramètres minimums :

```conf
UsePAM yes
ChallengeResponseAuthentication yes
PasswordAuthentication yes
AuthenticationMethods password,keyboard-interactive
PermitRootLogin no
```

### Explication

- `UsePAM yes` : autorise PAM à intervenir.
- `ChallengeResponseAuthentication yes` : permet la demande du code TOTP.
- `PasswordAuthentication yes` : requis ici pour le test en lab.
- `AuthenticationMethods password,keyboard-interactive` : impose mot de passe + interaction PAM.
- `PermitRootLogin no` : sécurise l'accès SSH.

### Vérification de syntaxe

```bash
sudo sshd -t
```

### Redémarrage du service

```bash
sudo systemctl restart ssh
sudo systemctl status ssh --no-pager
```

---

### 15.3 Test réel de connexion SSH

```bash
ssh admin-chu@localhost
```

### Résultat attendu

Le serveur demande successivement :

```text
Verification code:
Password:
```

ou selon la version :

```text
Password:
Verification code:
```

### Interprétation

Si l'accès n'est accordé qu'avec un bon mot de passe **et** un bon TOTP, alors le MFA SSH est fonctionnel.

---

## 16. Vérifications complémentaires

### Vérifier que PAM est bien actif côté SSH

```bash
grep -E 'UsePAM|ChallengeResponseAuthentication|PasswordAuthentication|AuthenticationMethods' /etc/ssh/sshd_config
```

### Vérifier la présence du module Google Authenticator dans PAM

```bash
grep google-authenticator /etc/pam.d/sshd /etc/pam.d/radiusd
```

### Vérifier les utilisateurs déclarés dans FreeRADIUS

```bash
grep -E 'admin-chu|svc_backup' /etc/freeradius/3.0/users
```

---

## 17. Scénario de validation final

<div class="table-wrap">
<table>
  <tr><th>Test</th><th>Commande / action</th><th>Résultat attendu</th></tr>
  <tr><td>Mot de passe seul</td><td><code>radtest admin-chu AdminStrongPass! localhost 1812 testing123</code></td><td>Reject</td></tr>
  <tr><td>Mauvais OTP</td><td><code>radtest admin-chu 'AdminStrongPass!000000' localhost 1812 testing123</code></td><td>Reject</td></tr>
  <tr><td>Bon OTP</td><td><code>radtest admin-chu 'AdminStrongPass!&lt;OTP&gt;' localhost 1812 testing123</code></td><td>Accept</td></tr>
  <tr><td>Connexion SSH réelle</td><td><code>ssh admin-chu@localhost</code></td><td>Demande TOTP + mot de passe puis ouverture de session</td></tr>
</table>
</div>

---

## 18. Dépannage courant

### Problème 1 — FreeRADIUS ne démarre pas

#### Symptôme

```text
Failed to add duplicate client localhost
```

#### Cause

Un bloc `client localhost` a été ajouté en doublon dans `clients.conf`.

#### Vérification

```bash
grep -n "client localhost" /etc/freeradius/3.0/clients.conf
```

#### Correction

Conserver uniquement le bloc d'origine.

---

### Problème 2 — `bash: !123456: event not found`

#### Cause

Le caractère `!` du mot de passe est interprété par Bash.

#### Correction

Utiliser des apostrophes simples :

```bash
radtest admin-chu 'AdminStrongPass!123456' localhost 1812 testing123
```

---

### Problème 3 — OTP toujours rejeté

#### Vérifications

```bash
sudo freeradius -X
grep google-authenticator /etc/pam.d/radiusd
ls -l /home/admin-chu/.google_authenticator
```

#### Causes possibles

- TOTP non généré pour l'utilisateur,
- mauvais code OTP,
- module PAM non activé,
- fichier `radiusd` incorrect,
- `pam` non inséré dans `sites-enabled/default`.

---

### Problème 4 — SSH ne demande pas le code TOTP

#### Vérifications

```bash
grep -E 'UsePAM|ChallengeResponseAuthentication|AuthenticationMethods' /etc/ssh/sshd_config
grep google-authenticator /etc/pam.d/sshd
sudo sshd -t
```

---

## 19. Ce que cette mise en place protège réellement

### Menaces couvertes

- **T1078 — Valid Accounts** : un mot de passe volé n'est plus suffisant.
- **T1021 — Remote Services** : l'accès distant via SSH nécessite un facteur supplémentaire.

### Cas d'usage réels

- accès SSH administrateur,
- VPN d'entreprise,
- WiFi d'entreprise avec RADIUS,
- bastion d'administration.

---

## 20. Limites et professionnalisation

### Ce qui est correct pour le TP

- démonstration fonctionnelle du MFA,
- validation technique par `radtest`,
- intégration réelle à SSH.

### Ce qu'il faudrait ajouter en production

- secret RADIUS long et robuste,
- journalisation centralisée,
- sauvegarde sécurisée des secrets,
- comptes de secours,
- procédure de récupération MFA,
- supervision du service,
- intégration AD/LDAP/IdP,
- durcissement PAM plus complet,
- authentification par clé SSH + TOTP.

---

## 21. Conclusion prête à réutiliser dans le rapport

> Une solution d'authentification multifacteur TOTP a été déployée avec FreeRADIUS, PAM et Google Authenticator afin de sécuriser les comptes sensibles `admin-chu` et `svc_backup`. Les tests ont montré qu'un mot de passe seul ne permet plus l'accès, qu'un code OTP invalide est rejeté, et qu'un mot de passe combiné à un OTP valide est accepté. L'intégration à SSH permet de démontrer un cas d'usage réel d'accès distant sécurisé. Cette mesure réduit fortement le risque d'exploitation de comptes compromis (MITRE ATT&CK T1078) et limite les accès distants non autorisés (T1021).

---

## 22. Bloc de commandes final — version courte opérationnelle

```bash
# Installation
sudo apt update
sudo apt install -y freeradius freeradius-utils libpam-google-authenticator

# Utilisateurs Linux
sudo useradd -m admin-chu
sudo useradd -m svc_backup
sudo passwd admin-chu
sudo passwd svc_backup

# Utilisateurs FreeRADIUS
sudo nano /etc/freeradius/3.0/users
# Ajouter :
# admin-chu Cleartext-Password := "AdminStrongPass!"
# svc_backup Cleartext-Password := "BackupSecure123"

# Vérifier client localhost
sudo nano /etc/freeradius/3.0/clients.conf
# Garder un seul bloc client localhost

# Activer module PAM
sudo ln -s /etc/freeradius/3.0/mods-available/pam /etc/freeradius/3.0/mods-enabled/

# Activer pam dans le site default
sudo nano /etc/freeradius/3.0/sites-enabled/default
# Dans authorize { } : pam
# Dans authenticate { } :
# Auth-Type PAM {
#     pam
# }

# Config PAM RADIUS
sudo nano /etc/pam.d/radiusd
# Contenu :
# auth required pam_google_authenticator.so
# account required pam_permit.so

# Génération TOTP
su - admin-chu
google-authenticator
su - svc_backup
google-authenticator

# Vérification et redémarrage
sudo freeradius -CX
sudo systemctl restart freeradius
sudo systemctl status freeradius --no-pager

# Tests RADIUS
radtest admin-chu AdminStrongPass! localhost 1812 testing123
radtest admin-chu 'AdminStrongPass!000000' localhost 1812 testing123
radtest admin-chu 'AdminStrongPass!<OTP>' localhost 1812 testing123

# Intégration SSH
sudo nano /etc/pam.d/sshd
# Ajouter en haut : auth required pam_google_authenticator.so

sudo nano /etc/ssh/sshd_config
# Vérifier :
# UsePAM yes
# ChallengeResponseAuthentication yes
# PasswordAuthentication yes
# AuthenticationMethods password,keyboard-interactive
# PermitRootLogin no

sudo sshd -t
sudo systemctl restart ssh
ssh admin-chu@localhost
```
