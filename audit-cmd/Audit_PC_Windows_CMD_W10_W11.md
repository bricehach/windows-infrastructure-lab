# Audit et maintenance d’un PC Windows avec l’Invite de commandes

## Cours de référence — Windows 11 Pro et commandes compatibles Windows 10

**Public visé :** techniciens IT et étudiants en progression vers un niveau professionnel  
**Environnement :** Windows 11 Pro ; les commandes compatibles Windows 10 sont signalées  
**Shell étudié :** `cmd.exe` (Invite de commandes), sans PowerShell  
**Date de révision :** 10 octobre 2026

---

## Sommaire

1. But, périmètre et limites
2. Préparer un audit fiable
3. Comprendre les résultats et les codes de retour
4. Collecter les informations de base
5. Auditer le stockage et le système de fichiers
6. Auditer Windows, les mises à jour et l’intégrité
7. Auditer les comptes, les groupes et les privilèges
8. Auditer les services, processus, pilotes et démarrage
9. Auditer le réseau et le pare-feu
10. Auditer les journaux d’événements
11. Réparer de façon contrôlée
12. Tableau de décision rapide
13. Aide-mémoire des commandes
14. Sources Microsoft

---

## 1. But, périmètre et limites

Un audit avec `cmd.exe` consiste à collecter des éléments vérifiables sur l’état d’un poste : configuration matérielle et logicielle, stockage, comptes, services, pilotes, réseau, pare-feu, journaux et intégrité de Windows. Il sert à établir un état initial, à orienter un diagnostic et à vérifier une réparation.

Il ne constitue pas à lui seul un audit de sécurité complet. Certaines vérifications demandent des outils spécialisés, des journaux centralisés ou une inspection hors ligne. Une commande peut aussi ne rien afficher parce que l’accès est refusé, qu’un composant est absent ou que la configuration ne s’applique pas à ce poste.

### Ce que signifie « commande Windows » dans ce cours

- **Commande interne :** mot-clé interprété par `cmd.exe`, par exemple `dir`, `if` ou `set`.
- **Exécutable :** programme externe lancé depuis l’invite, par exemple `ipconfig.exe`, `sfc.exe` ou `diskpart.exe`.
- **Sous-commande :** action propre à un exécutable, par exemple `reg query` ou `netsh advfirewall`.
- **Option :** paramètre qui modifie le comportement d’une commande, souvent précédé de `/`, par exemple `chkdsk C: /scan`.

`help` liste les commandes reconnues par l’interpréteur. `nom_commande /?` affiche l’aide disponible pour de nombreux outils. La documentation Microsoft A–Z décrit des commandes Windows 11, mais inclut aussi des outils Windows Server, historiques ou conditionnels ; leur présence sur un poste client n’est donc pas garantie.

### Repères de compatibilité utilisés

- **[W10/W11]** : commande courante sur Windows 10 et Windows 11, avec possibles différences selon la version ou les composants installés.
- **[W11]** : commande ou option visant Windows 11 ; vérifier la présence et le comportement dans la version concernée.
- **[OPTIONNEL]** : outil dépendant d’une fonctionnalité, d’un rôle, d’un package ou d’une installation supplémentaire.
- **[DÉPRÉCIÉ]** : outil conservé pour compatibilité, mais que Microsoft ne recommande plus pour les nouveaux usages.

Une commande marquée [W10/W11] peut avoir des options qui ne sont pas disponibles sur toutes les versions. Vérifier `commande /?` sur le poste audité avant d’utiliser une option peu connue.

---

## 2. Préparer un audit fiable

### 2.1 Définir le cadre

Avant de lancer les commandes, noter :

- le nom du poste, son utilisateur et son emplacement logique (poste personnel, entreprise, laboratoire) ;
- la date et l’heure de début ;
- le symptôme signalé, sans le transformer en conclusion ;
- les changements récents connus : mise à jour, pilote, logiciel, panne électrique, changement réseau ;
- si l’audit est en lecture seule ou comprend des opérations de réparation ;
- si l’opérateur dispose d’une session administrateur autorisée.

Ne pas lancer de commande de modification avant d’avoir identifié le poste, compris le résultat attendu, vérifié les sauvegardes utiles et obtenu l’autorisation requise.

### 2.2 Ouvrir une invite adaptée

Pour un audit simple, une invite standard suffit souvent. Les contrôles système, la lecture de certaines informations de sécurité et la majorité des réparations nécessitent une invite élevée.

1. Ouvrir le menu Démarrer.
2. Saisir `cmd`.
3. Choisir **Exécuter en tant qu’administrateur** si le contrôle le requiert.
4. Vérifier le contexte :

```cmd
whoami
whoami /groups
```

Une invite élevée est soumise au contrôle de compte utilisateur (UAC). La présence du groupe Administrateurs dans `whoami /groups` ne prouve pas à elle seule que le processus courant est élevé : les jetons UAC peuvent être filtrés.

### 2.3 Créer un dossier de collecte

Les commandes suivantes créent un dossier et des fichiers de rapport. Elles ne réparent pas Windows, mais écrivent sur le disque.

```cmd
set "AUDIT=%USERPROFILE%\Desktop\Audit_PC_%COMPUTERNAME%"
mkdir "%AUDIT%"
echo Poste=%COMPUTERNAME%> "%AUDIT%\contexte.txt"
echo Utilisateur=%USERNAME%>> "%AUDIT%\contexte.txt"
echo Date=%DATE% %TIME%>> "%AUDIT%\contexte.txt"
```

Dans `cmd.exe`, `>` crée ou remplace le fichier ; `>>` ajoute à la fin. Ne pas réutiliser un dossier qui contient des résultats d’un audit précédent sans distinguer les dates.

Exemple d’enregistrement de sortie :

```cmd
systeminfo > "%AUDIT%\systeminfo.txt" 2>&1
```

`2>&1` envoie aussi les messages d’erreur dans le fichier. Une sortie vide ou incomplète doit être interprétée avec le code de retour et les droits de la session.

### 2.4 Collecter proprement

- Conserver la commande exacte, le contexte (standard ou administrateur), la date et la sortie.
- Ne pas publier de rapport brut contenant noms d’utilisateurs, noms de machines, adresses IP, noms Wi-Fi, numéros de série, clés, chemins personnels ou informations d’entreprise.
- Masquer les données sensibles avant de joindre une capture ou de déposer le document dans un dépôt public.
- Conserver les originaux dans un emplacement maîtrisé et travailler sur une copie pour l’analyse.
- Ne pas confondre une sortie d’audit et une preuve complète : une commande locale peut être altérée par un système compromis.

---

## 3. Comprendre les résultats et les codes de retour

Après une commande, `echo %ERRORLEVEL%` affiche le code de retour du dernier programme exécuté :

```cmd
sfc /verifyonly
echo CodeRetour=%ERRORLEVEL%
```

Le sens exact dépend du programme. En général, `0` indique que l’opération s’est terminée sans erreur signalée ; un code non nul indique un état particulier ou une erreur. Toujours consulter l’aide et la documentation du programme avant de traduire un code en conclusion.

### Distinguer observation et interprétation

| Observation | Interprétation prudente |
|---|---|
| La commande répond « Accès refusé » | Le jeton ou les autorisations ne permettent pas cette action ; cela ne prouve pas une corruption. |
| Un nom de commande n’est pas reconnu | L’outil peut être absent, non accessible via `PATH`, ou le nom peut être incorrect. |
| Un service est `STOPPED` | Il est arrêté au moment du contrôle ; ce n’est pas forcément une anomalie. |
| Un port apparaît dans `netstat` | Une socket est en écoute ou une connexion est présente ; il faut identifier le PID et le processus. |
| `sfc` ne trouve aucune violation | Le contrôle n’a pas détecté de violation protégée ; il ne prouve pas l’absence de tout problème Windows. |
| `chkdsk /scan` ne trouve pas d’erreur | Le système de fichiers n’a pas signalé d’erreur lors de ce contrôle ; cela ne certifie pas l’état physique du disque. |

Une commande de contrôle donne un résultat à un instant donné. Si un état change, répéter le contrôle après avoir noté le changement.

---

## 4. Collecter les informations de base

| Commande | Usage et interprétation |
|---|---|
| `ver` **[W10/W11]** | Version du noyau affichée par l’invite. Elle ne donne pas toujours le numéro de version convivial complet. |
| `systeminfo` **[W10/W11]** | Résumé OS, matériel, démarrage, correctifs et domaine. Sortie longue ; peut prendre quelques secondes. Enregistrer avec `systeminfo > fichier.txt`. |
| `hostname` **[W10/W11]** | Nom réseau local du poste. |
| `whoami` **[W10/W11]** | Identité de l’utilisateur courant. `whoami /all` affiche identifiants, groupes et privilèges du jeton. À protéger comme donnée sensible. |
| `driverquery /v /fo table` **[W10/W11]** | Liste les pilotes installés et des détails. Un pilote ancien ou arrêté n’est pas automatiquement vulnérable ou fautif. |
| `msinfo32` **[W10/W11]** | Ouvre l’interface Informations système. Pour enregistrer depuis la ligne de commande : `msinfo32 /report "%AUDIT%\msinfo.txt"`. Vérifier `msinfo32 /?` selon la version. |
| `dxdiag /t "%AUDIT%\dxdiag.txt"` **[W10/W11]** | Collecte des informations DirectX et périphériques. Certaines données peuvent être absentes si le rapport n’a pas fini de s’écrire. |
| `set` **[interne]** | Affiche les variables d’environnement de la session. Des variables peuvent contenir des chemins personnels ou des secrets d’outils ; ne pas publier sans revue. |

Commandes de collecte :

```cmd
ver > "%AUDIT%\version.txt"
systeminfo > "%AUDIT%\systeminfo.txt" 2>&1
hostname > "%AUDIT%\hostname.txt"
whoami /all > "%AUDIT%\identite_jeton.txt" 2>&1
driverquery /v /fo csv > "%AUDIT%\pilotes.csv" 2>&1
msinfo32 /report "%AUDIT%\msinfo.txt"
```

Le format CSV facilite le tri dans un tableur, mais le fichier reste à traiter comme une donnée potentiellement sensible.

---

## 5. Auditer le stockage et le système de fichiers

### 5.1 Espace disponible et volumes

```cmd
wmic logicaldisk get caption,filesystem,freespace,size
```

**[DÉPRÉCIÉ / disponibilité variable]** `wmic` est déprécié et peut avoir été retiré ou désactivé dans certaines installations. Ne pas en faire une dépendance d’un guide moderne. À défaut :

```cmd
fsutil volume diskfree C:
vol C:
```

`fsutil` peut exiger une invite administrateur selon l’opération et le volume. `vol` affiche l’étiquette et le numéro de série logique, pas l’état de santé physique.

### 5.2 Vérification du système de fichiers

```cmd
chkdsk C: /scan
```

**[W10/W11]** Analyse en ligne du volume NTFS et recherche de problèmes sans planifier la réparation hors ligne. Utiliser la lettre du volume concerné ; ne pas supposer que Windows est toujours sur `C:`.

Options courantes :

- `chkdsk C:` : vérifie et affiche l’état sans demander une réparation automatique.
- `chkdsk C: /scan` : analyse en ligne pour NTFS.
- `chkdsk C: /f` : corrige les erreurs logiques ; le volume système peut nécessiter une planification au redémarrage.
- `chkdsk C: /r` : recherche aussi les secteurs illisibles et tente de récupérer les informations lisibles ; opération longue, à réserver à un diagnostic justifié et avec une sauvegarde adaptée.

Ne pas lancer `/f` ou `/r` comme première étape si le disque présente des signes de panne physique ou contient des données irremplaçables non sauvegardées. Une réparation du système de fichiers n’est pas une réparation matérielle.

### 5.3 Vérification des fichiers et dossiers

```cmd
dir C:\ /a
chkdsk C: /scan
```

`dir /a` affiche aussi les fichiers cachés et système. Une entrée inconnue ne suffit pas à conclure à une infection. Noter le chemin, la taille, la date et l’éditeur avant toute action.

### 5.4 BitLocker

```cmd
manage-bde -status
```

**[W10/W11 Pro]** Affiche les volumes BitLocker, l’état de chiffrement et la protection. Ne jamais publier une clé de récupération. La présence d’un volume chiffré ne prouve pas que la clé de récupération est correctement sauvegardée.

### 5.5 Contrôle SMART et santé physique

Les commandes de base de `cmd.exe` ne fournissent pas un diagnostic matériel complet et universel de l’état SMART des disques. `chkdsk` vérifie principalement la cohérence du système de fichiers. Pour une évaluation matérielle, utiliser les diagnostics du fabricant, l’outil de gestion approuvé par l’organisation, ou le BIOS/UEFI, puis comparer avec les symptômes et les journaux.

---

## 6. Auditer Windows, les mises à jour et l’intégrité

### 6.1 Version et correctifs

```cmd
systeminfo
```

La section « Hotfix(s) » peut fournir une vue partielle des mises à jour installées. Elle ne remplace pas l’historique Windows Update ou un outil de gestion centralisé. Les versions de Windows 10 et 11 évoluent ; consigner l’édition, la version et le numéro de build.

### 6.2 Vérifier l’image Windows avec DISM

```cmd
dism /online /cleanup-image /checkhealth
dism /online /cleanup-image /scanhealth
```

**[W10/W11]** Ces commandes vérifient l’image Windows en cours d’exécution. `/CheckHealth` lit l’état déjà connu ; `/ScanHealth` effectue une analyse plus approfondie. Elles ne sont pas équivalentes à un contrôle de tous les fichiers personnels ou de toutes les applications.

### 6.3 Vérifier les fichiers système avec SFC

```cmd
sfc /verifyonly
```

**[W10/W11]** Vérifie les fichiers système protégés sans tenter de les réparer. Si des violations sont signalées, conserver le résultat et passer à l’étape de réparation uniquement après avoir évalué le contexte.

Les journaux SFC sont écrits dans `%windir%\Logs\CBS\CBS.log`. Une copie filtrée peut être créée ainsi :

```cmd
findstr /c:"[SR]" %windir%\Logs\CBS\CBS.log > "%AUDIT%\sfc-details.txt"
```

Le journal peut contenir des informations système ; le protéger.

### 6.4 Activation et composants facultatifs

```cmd
slmgr /xpr
dism /online /get-features /format:table
```

`slmgr /xpr` affiche l’état d’activation dans une fenêtre. `DISM /Get-Features` liste les fonctionnalités de Windows et leur état ; les fonctionnalités visibles varient avec l’édition et les packages installés.

---

## 7. Auditer les comptes, les groupes et les privilèges

### 7.1 Identité et groupes locaux

```cmd
whoami /all
net user
net localgroup
net localgroup Administrateurs
```

Selon la langue d’installation, le nom du groupe local peut être traduit. Si `Administrateurs` n’est pas reconnu, exécuter `net localgroup` et reprendre le nom exact affiché. Sur un poste joint à un domaine, les comptes et stratégies du domaine doivent aussi être considérés.

Pour examiner un compte local précis :

```cmd
net user NOM_UTILISATEUR
```

Cette commande expose des informations de compte. Éviter de collecter ou publier des données qui ne sont pas nécessaires. L’existence d’un compte désactivé ou d’un compte intégré n’est pas en soi une anomalie ; comparer avec la politique de l’organisation.

### 7.2 Stratégies appliquées

```cmd
gpresult /r
gpresult /h "%AUDIT%\strategies.html"
```

**[W10/W11 Pro]** Résume les stratégies de groupe appliquées au poste et à l’utilisateur. `/h` génère un rapport HTML. L’absence de stratégie de domaine peut être normale sur un poste autonome ou géré par un autre système.

### 7.3 Stratégie d’audit locale

```cmd
auditpol /get /category:*
```

Affiche la stratégie d’audit effective accessible à l’utilisateur. Une catégorie non configurée ne prouve pas un défaut : les exigences dépendent du rôle du poste et de la politique de sécurité applicable.

### 7.4 Partages et sessions réseau

```cmd
net share
net use
net session
```

`net session` peut exiger des droits élevés et concerne les sessions entrantes vers le poste. Examiner chaque partage selon son usage, ses autorisations et son exposition réseau. Ne pas supprimer un partage de manière automatique.

---

## 8. Auditer les services, processus, pilotes et démarrage

### 8.1 Processus en cours

```cmd
tasklist /v
tasklist /svc
```

`tasklist /v` présente des détails de processus ; `tasklist /svc` associe certains processus aux services hébergés. Un nom inconnu ou un chemin inhabituel doit être investigué, mais le nom seul ne permet pas de conclure qu’un programme est malveillant.

Pour relier un port réseau au PID d’un processus, utiliser `netstat -ano`, puis :

```cmd
tasklist /fi "PID eq 1234" /v
```

Remplacer `1234` par le PID constaté.

### 8.2 Services

```cmd
sc query type= service state= all
sc query WinDefend
sc qc WinDefend
```

**[W10/W11]** `sc query` interroge l’état ; `sc qc` affiche la configuration du service. Dans `sc query type= service`, l’espace après `=` est requis par la syntaxe de `sc`.

Les services peuvent être déclenchés à la demande, arrêtés quand ils ne sont pas nécessaires ou protégés par une stratégie. Comparer le nom du service, son chemin binaire, son compte d’exécution, sa configuration et son rôle avant de le modifier.

### 8.3 Tâches planifiées

```cmd
schtasks /query /fo LIST /v
```

Affiche les tâches planifiées en format détaillé. La sortie peut être volumineuse et contenir des chemins ou noms d’utilisateurs. Une tâche inconnue doit être vérifiée : éditeur, emplacement, déclencheur, action, signature et lien avec un logiciel installé.

### 8.4 Pilotes

```cmd
driverquery /v /fo csv
pnputil /enum-drivers
```

`driverquery` liste les pilotes chargés/connus. `pnputil /enum-drivers` énumère les packages de pilotes du magasin de pilotes. Un pilote ancien n’est pas automatiquement incompatible. Avant toute suppression ou remplacement, identifier le matériel, le fournisseur, la version, la signature, la source du package et la procédure de retour arrière.

### 8.5 Clés de démarrage courantes

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"
```

Ces commandes lisent quelques emplacements de démarrage automatique. Elles ne constituent pas un inventaire complet de l’autostart Windows. Ne pas supprimer une valeur simplement parce qu’elle est inconnue ; valider le chemin et l’éditeur, puis comparer à une source fiable.

---

## 9. Auditer le réseau et le pare-feu

### 9.1 Configuration IP

```cmd
ipconfig /all
```

**[W10/W11]** Affiche interfaces, adresses, passerelles, DNS, DHCP et informations connexes. Masquer les adresses et noms d’hôtes avant toute publication.

### 9.2 Connectivité et résolution DNS

```cmd
ping -n 4 127.0.0.1
ping -n 4 ADRESSE_PASSERELLE
ping -n 4 NOM_DNS
nslookup NOM_DNS
tracert NOM_DNS
pathping NOM_DNS
```

- `127.0.0.1` vérifie la pile IP locale, pas l’accès au réseau physique.
- Une absence de réponse à `ping` peut venir d’un filtrage ICMP ; elle ne prouve pas que la machine est hors ligne.
- `tracert` et `pathping` dépendent du comportement des routeurs et des règles de filtrage.
- `nslookup` interroge le DNS ; comparer le serveur utilisé et la réponse attendue.

### 9.3 Interfaces, routes et connexions

```cmd
getmac /v
route print
arp -a
netstat -ano
```

`netstat -ano` montre les connexions et ports avec le PID. « LISTENING » ne suffit pas à déterminer si un service doit être accessible ; vérifier le processus, l’interface locale, le pare-feu, le profil réseau et le besoin métier.

### 9.4 Pare-feu Windows

```cmd
netsh advfirewall show allprofiles
netsh advfirewall firewall show rule name=all verbose
```

La seconde commande peut générer beaucoup de texte. Vérifier les profils Domaine, Privé et Public, l’état du pare-feu et les règles pertinentes. Un profil actif dépend de la classification du réseau ; ne pas désactiver le pare-feu pour « tester » sans autorisation et mesure compensatoire.

### 9.5 Wi-Fi

```cmd
netsh wlan show interfaces
netsh wlan show profiles
```

Affiche les interfaces sans fil et les profils mémorisés. Les noms de réseaux sont des informations sensibles. Ne pas demander ou afficher les clés Wi-Fi avec `netsh wlan ... key=clear` dans un audit général ; cette opération révèle des secrets.

### 9.6 Remise à zéro réseau : action de réparation, pas diagnostic

Les commandes ci-dessous modifient la configuration réseau et peuvent interrompre des connexions, VPN, paramètres statiques ou accès distants. Les exécuter uniquement après avoir enregistré la configuration et obtenu l’autorisation.

```cmd
ipconfig /flushdns
netsh winsock reset
netsh int ip reset "%AUDIT%\ip-reset.log"
```

`ipconfig /flushdns` vide le cache DNS local. Les deux commandes `netsh` réinitialisent des éléments réseau et peuvent nécessiter un redémarrage. Ne pas enchaîner ces commandes comme rituel systématique : choisir l’action en fonction du symptôme.

---

## 10. Auditer les journaux d’événements

### 10.1 Lister les journaux

```cmd
wevtutil el
```

Liste les journaux enregistrés. Les noms exacts varient selon les fonctionnalités et les logiciels installés.

### 10.2 Lire les événements récents

```cmd
wevtutil qe System /c:50 /rd:true /f:text
wevtutil qe Application /c:50 /rd:true /f:text
```

Affiche les 50 événements les plus récents du journal concerné, en ordre inverse (`/rd:true`). Un événement isolé n’explique pas nécessairement le symptôme. Corréler l’heure, la source, l’ID, la sévérité, la répétition et les changements observés.

Pour exporter un journal entier vers un fichier EVTX :

```cmd
wevtutil epl System "%AUDIT%\System.evtx"
```

Cette commande écrit un fichier potentiellement volumineux et sensible. Protéger l’accès et respecter la durée de conservation. L’Observateur d’événements peut aussi être ouvert par `eventvwr.msc`.

### 10.3 Journaux de réparation

- DISM : `%windir%\Logs\DISM\dism.log`
- SFC / CBS : `%windir%\Logs\CBS\CBS.log`
- Démarrage et maintenance : journaux spécifiques selon l’opération et la version.

Ne pas conclure à une défaillance à partir d’une ligne d’erreur isolée. Examiner l’heure, l’opération en cours et les lignes adjacentes.

---

## 11. Réparer de façon contrôlée

### 11.1 Principes de réparation

1. Reproduire ou préciser le symptôme.
2. Capturer l’état avant intervention.
3. Vérifier l’espace disque, l’alimentation et la connectivité requise.
4. Sauvegarder les données importantes et noter les modifications récentes.
5. Effectuer une seule intervention à la fois.
6. Redémarrer si l’outil le demande.
7. Refaire le contrôle concerné et confirmer que le symptôme a disparu.
8. Noter le résultat, le code de retour, la durée et les effets secondaires.

### 11.2 Réparer les fichiers système avec SFC

À lancer dans une invite administrateur :

```cmd
sfc /scannow
```

Cette commande vérifie les fichiers système protégés et tente de réparer les problèmes détectés. Attendre la fin complète ; ne pas fermer la fenêtre pendant l’opération. Consulter ensuite le message final et, si nécessaire, le journal CBS.

### 11.3 Réparer l’image Windows avec DISM

Dans une invite administrateur :

```cmd
dism /online /cleanup-image /restorehealth
```

Cette commande recherche et répare la corruption du magasin de composants de l’installation Windows en cours. Elle peut utiliser Windows Update comme source et prendre du temps. Si elle échoue, conserver le code et le journal DISM avant de répéter avec une source d’installation adaptée.

Ordre généralement utilisé lorsque l’intégrité système est en cause :

```cmd
dism /online /cleanup-image /restorehealth
sfc /scannow
```

Effectuer ensuite un redémarrage si demandé et répéter les vérifications nécessaires.

### 11.4 Réparer le système de fichiers

Après sauvegarde et analyse :

```cmd
chkdsk C: /f
```

Si le volume est utilisé, Windows peut proposer de planifier l’opération au prochain démarrage. Vérifier qu’un redémarrage est possible et informer l’utilisateur. Utiliser `/r` uniquement si le besoin est établi ; cette option peut être très longue.

### 11.5 Nettoyage du magasin de composants

```cmd
dism /online /cleanup-image /startcomponentcleanup
```

Cette opération de maintenance modifie le magasin de composants et peut réduire la possibilité de désinstaller certaines mises à jour remplacées. Ne pas l’utiliser comme première réponse à un problème d’espace sans évaluer le besoin et les options de récupération.

### 11.6 Opérations à fort impact

Les actions suivantes ne sont pas des vérifications passives :

- `diskpart clean`, `format`, `convert`, suppression ou création de partitions ;
- `bcdedit` ou `bcdboot` pour modifier la configuration de démarrage ;
- suppression de valeurs avec `reg delete` ;
- arrêt/désactivation de services avec `sc stop` ou `sc config` ;
- réinitialisation réseau ;
- suppression de pilotes ou packages avec `pnputil` ;
- réinitialisation, réinstallation ou réinitialisation usine de Windows.

Une erreur de cible ou de syntaxe peut provoquer une perte de données, une indisponibilité ou un démarrage impossible. Documenter l’état initial et disposer d’un plan de retour arrière avant toute action.

---

## 12. Tableau de décision rapide

| Symptôme ou question | Vérifications initiales | Réparation possible après validation |
|---|---|---|
| Windows ou une application est instable | `systeminfo`, `wevtutil`, `sfc /verifyonly`, `dism ... /scanhealth` | DISM `/RestoreHealth`, puis `sfc /scannow` si justifié |
| Manque d’espace disque | `fsutil volume diskfree C:`, `dir`, `dism /Get-Features` | Nettoyage ciblé ; ne pas supprimer manuellement les dossiers système |
| Erreurs de fichiers ou volume | `chkdsk C:`, `chkdsk C: /scan`, journaux système | `/f` après sauvegarde ; `/r` seulement si pertinent |
| Pas d’accès réseau | `ipconfig /all`, `ping`, `nslookup`, `route print`, `netsh advfirewall show allprofiles` | Renouvellement/réinitialisation uniquement après diagnostic |
| Port inattendu | `netstat -ano`, `tasklist /fi "PID eq ..."`, règles pare-feu | Désactivation ou règle de blocage selon politique et preuve |
| Démarrage lent | `tasklist`, `schtasks /query`, événements, pilotes | Désactiver une entrée uniquement après validation de son rôle |
| Suspicion de droits excessifs | `whoami /all`, `net localgroup`, `gpresult /r` | Ajuster via procédure contrôlée et politique d’entreprise |
| Dysfonctionnement d’un pilote | `driverquery`, `pnputil /enum-drivers`, événements | Mise à jour/retour arrière depuis source validée |

Ce tableau oriente les vérifications ; il ne remplace pas l’analyse du poste et de son contexte.

---

## 13. Aide-mémoire des commandes

### Aide et contexte

```cmd
help
commande /?
ver
systeminfo
hostname
whoami /all
```

### Stockage et intégrité

```cmd
fsutil volume diskfree C:
chkdsk C: /scan
manage-bde -status
dism /online /cleanup-image /scanhealth
sfc /verifyonly
```

### Comptes, services et tâches

```cmd
net user
net localgroup
gpresult /r
auditpol /get /category:*
sc query type= service state= all
tasklist /svc
schtasks /query /fo LIST /v
```

### Pilotes et réseau

```cmd
driverquery /v /fo csv
pnputil /enum-drivers
ipconfig /all
route print
netstat -ano
netsh advfirewall show allprofiles
```

### Journaux

```cmd
wevtutil el
wevtutil qe System /c:50 /rd:true /f:text
wevtutil qe Application /c:50 /rd:true /f:text
```

### Réparations principales — invite administrateur

```cmd
dism /online /cleanup-image /restorehealth
sfc /scannow
chkdsk C: /f
```

Toujours vérifier la cible, l’aide locale, les privilèges et les effets avant les opérations de réparation.

---

## 14. Sources Microsoft

Les références ci-dessous sont des sources primaires. Les pages de référence peuvent être mises à jour ; vérifier la documentation de la version de Windows et l’aide locale avant une opération de maintenance.

- [Référence A–Z des commandes Windows](https://learn.microsoft.com/windows-server/administration/windows-commands/windows-commands) — couvre Windows 11 et Windows 10, avec aussi des commandes Server et conditionnelles.
- [`cmd`](https://learn.microsoft.com/windows-server/administration/windows-commands/cmd) — paramètres de l’interpréteur de commandes.
- [`help`](https://learn.microsoft.com/windows-server/administration/windows-commands/help) — aide disponible dans l’invite.
- [`chkdsk`](https://learn.microsoft.com/windows-server/administration/windows-commands/chkdsk) — vérification du système de fichiers.
- [`sfc`](https://learn.microsoft.com/windows-server/administration/windows-commands/sfc) — vérificateur des fichiers système.
- [`DISM`](https://learn.microsoft.com/windows-hardware/manufacture/desktop/what-is-dism) — gestion et maintenance des images Windows.
- [`ipconfig`](https://learn.microsoft.com/windows-server/administration/windows-commands/ipconfig) — configuration IP et cache DNS.
- [`netsh`](https://learn.microsoft.com/windows-server/administration/windows-commands/netsh) — commandes de configuration réseau.
- [`wevtutil`](https://learn.microsoft.com/windows-server/administration/windows-commands/wevtutil) — gestion des journaux d’événements.
- [`wmic`](https://learn.microsoft.com/windows-server/administration/windows-commands/wmic) — utilitaire WMIC déprécié.

---

## Note de publication GitHub

Avant de publier un rapport d’audit ou des captures dans un dépôt public, retirer les noms de machines et comptes, adresses IP publiques ou internes, identifiants, clés, numéros de série, noms Wi-Fi, chemins personnels et détails de l’infrastructure. Ce support de cours ne contient pas de données provenant d’un poste réel.
