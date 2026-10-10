# Réparer le démarrage de Windows 10 et Windows 11

> Guide pédagogique avancé. Les commandes s’exécutent dans l’invite de commandes de WinRE ou dans un terminal administrateur, selon la section. Windows 10 et Windows 11 sont couverts ; les limites de contexte sont précisées.

## Objectifs

À la fin de ce guide, l’apprenant doit pouvoir situer une panne dans la chaîne de démarrage, distinguer un incident firmware/disque/partition/BCD/système, identifier les volumes depuis WinRE, protéger les données et BitLocker, puis choisir une action minimale et vérifier son résultat.

Les TP, scénarios et évaluations sont à concevoir séparément.

## 1. Comprendre la chaîne de démarrage

### UEFI / GPT

1. Le firmware UEFI initialise le matériel et lit une entrée de démarrage NVRAM.
2. Windows Boot Manager démarre depuis la partition système EFI (ESP), généralement FAT32 : \EFI\Microsoft\Boot\bootmgfw.efi.
3. Le magasin BCD fournit les paramètres de démarrage.
4. winload.efi charge le noyau, HAL et pilotes de démarrage.
5. Windows initialise les services et ouvre la session.

### BIOS hérité / MBR

Le BIOS passe le contrôle au code d’amorçage du disque. Le code MBR puis celui de la partition système conduisent à Windows Boot Manager (bootmgr), au BCD et à winload.exe.

Une réparation de fichiers EFI ne corrige pas un SSD absent du firmware. Une réparation du BCD ne corrige pas un volume chiffré et verrouillé. Il faut d’abord localiser l’étape qui échoue.

## 2. Triage avant réparation

Avant toute écriture sur un disque, relever :

- l’étape de l’échec : aucun POST, firmware, logo Windows, écran de connexion, redémarrage ou BSOD ;
- le texte et le code exacts, photographiés si possible ;
- les changements récents : mise à jour, pilote, BIOS/UEFI, clonage, ajout matériel ou configuration de stockage ;
- le modèle du disque, sa détection dans le firmware et les signes de panne physique ;
- le mode firmware (UEFI ou Legacy/CSM), le mode SATA/RAID et l’ordre de démarrage ;
- l’état BitLocker et la disponibilité de la clé de récupération ;
- les sauvegardes, journaux et dumps disponibles.

Si le disque claque, disparaît ou renvoie des erreurs d’E/S répétées, limiter les écritures et privilégier la sauvegarde ou une imagerie spécialisée. Ne pas lancer des réparations en boucle sur un support suspect.

### WinRE et lettres de lecteur

Les lettres peuvent changer dans WinRE. L’installation Windows qui est sur C: en fonctionnement peut être sur D: ou une autre lettre ici. X: est généralement le disque RAM de WinRE, pas l’installation.

Dans l’invite de commandes de WinRE :

~~~cmd
diskpart
list volume
exit
~~~

Repérer les volumes et confirmer le dossier Windows :

~~~cmd
dir D:\Windows
~~~

Essayer d’autres lettres et vérifier que le volume contient bien l’installation attendue. Noter la lettre utilisée ; ne pas supposer que les exemples D: et S: correspondent à chaque machine.

## 3. BitLocker : vérifier avant l’accès hors ligne

Un volume verrouillé peut sembler vide ou inaccessible. Dans WinRE :

~~~cmd
manage-bde -status
~~~

Si nécessaire, déverrouiller uniquement le volume concerné avec la clé autorisée :

~~~cmd
manage-bde -unlock D: -RecoveryPassword <clé-de-récupération-48-chiffres>
~~~

Ne jamais publier la clé de récupération dans un rapport, un dépôt GitHub, une capture ou une conversation non sécurisée. Des changements firmware/TPM peuvent déclencher une demande de récupération : éviter les bascules répétées de Secure Boot, TPM, mode SATA ou ordre de démarrage.

## 4. Observer disques et partitions avec DiskPart

DiskPart permet d’inventorier disques, partitions et volumes. Au début, rester en lecture :

~~~cmd
diskpart
list disk
select disk 0
detail disk
list partition
list volume
exit
~~~

Adapter le numéro après vérification. Pour examiner un volume :

~~~cmd
diskpart
select volume <n>
detail volume
exit
~~~

Sur UEFI/GPT, l’ESP est une petite partition système, souvent FAT32, sans lettre. Pour l’inspecter temporairement, après avoir confirmé son identité :

~~~cmd
diskpart
list volume
select volume <n>
assign letter=S
exit
dir S:\EFI\Microsoft\Boot
~~~

Après la procédure, retirer uniquement la lettre temporaire :

~~~cmd
diskpart
select volume <n>
remove letter=S
exit
~~~

**Commandes destructives à ne pas employer comme diagnostic :** clean, format, delete partition, convert. La commande active concerne le démarrage BIOS/MBR et ne doit pas servir à réparer une installation UEFI/GPT. Vérifier le disque et les données avant toute écriture.

## 5. Réparation du démarrage automatisée

Dans WinRE, commencer par **Dépannage → Options avancées → Outil de redémarrage système**. L’outil peut corriger certains problèmes détectés automatiquement. Son échec ne prouve pas que Windows ou le disque est irrécupérable.

S’il échoue, conserver le journal lorsqu’il est disponible, par exemple SrtTrail.txt, puis établir un diagnostic avant de répéter ou d’empiler les réparations.

## 6. Réparer les fichiers UEFI avec BCDBoot

À utiliser seulement après avoir confirmé : démarrage UEFI, disque présent et lisible, volume Windows déverrouillé, lettre de l’installation et identité de l’ESP FAT32 du bon disque.

Exemple où Windows est sur D: et l’ESP sur S: :

~~~cmd
dir D:\Windows
dir S:\EFI
bcdboot D:\Windows /s S: /f UEFI
~~~

Le message de réussite confirme la copie des fichiers d’environnement de démarrage, pas le fonctionnement complet du système. Vérifier le firmware, l’entrée Windows Boot Manager et le démarrage réel. S’il existe plusieurs disques/ESP, sélectionner le bon disque avec soin.

Si S: a été assignée temporairement, retirer cette lettre via DiskPart après vérification. Ne pas formater l’ESP pour « repartir proprement » sans procédure validée et sauvegarde.

## 7. BOOTREC : portée et limites

Depuis WinRE, les commandes suivantes peuvent aider dans des contextes BIOS/MBR ou pour rechercher une installation :

~~~cmd
bootrec /scanos
bootrec /rebuildbcd
~~~

Les options /fixmbr et /fixboot sont historiquement associées au dépannage BIOS/MBR ; elles ne remplacent pas l’identification correcte de l’ESP et l’usage de BCDBoot en UEFI/GPT. Un résultat /scanos égal à zéro ne prouve pas que Windows a disparu : vérifier les lettres, BitLocker, l’état des partitions et le chemin Windows.

Éviter les recettes qui enchaînent toutes les options BOOTREC sans déterminer le mode de démarrage.

## 8. Vérifier le BCD et le mode de démarrage

Depuis Windows démarré, ou depuis un environnement où l’outil cible bien le magasin voulu :

~~~cmd
bcdedit /enum
bcdedit /enum all
~~~

BCDEdit est puissant : une modification erronée peut rendre le système non amorçable. N’utiliser /store que si le magasin ciblé est connu. Ne pas copier un identifiant ou un chemin d’un autre PC sans validation.

Dans DiskPart, la colonne GPT de list disk aide à vérifier le partitionnement. L’absence d’astérisque indique généralement un disque MBR ; cette observation ne valide pas à elle seule le mode réellement utilisé au démarrage.

## 9. Réparer Windows hors ligne : SFC et DISM

Les vérifications de fichiers système interviennent après identification de la bonne installation. En Windows démarré, invite administrateur :

~~~cmd
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
~~~

DISM répare le magasin de composants utilisé par SFC ; il ne recrée pas l’ESP et ne remplace pas une réparation de la chaîne d’amorçage.

Dans WinRE, utiliser les chemins réels. Exemple à adapter :

~~~cmd
sfc /scannow /offbootdir=D:\ /offwindir=D:\Windows
~~~

La source et les répertoires peuvent dépendre de l’image, de la version et de la configuration. Si SFC ou DISM échoue, conserver le message et le journal. Une source de réparation doit correspondre à l’édition, la langue et une version compatible.

## 10. CHKDSK : choisir selon les indices

Pour un examen du système de fichiers :

~~~cmd
chkdsk D:
~~~

L’option /f modifie le système de fichiers ; /r recherche les secteurs illisibles et inclut la réparation logique. Ces options peuvent être longues et écrire sur le disque. En cas de suspicion de panne physique, protéger les données avant CHKDSK et suivre une stratégie de récupération appropriée. CHKDSK ne répare pas le matériel.

## 11. Mise à jour, pilote et configuration

Si la panne suit une mise à jour ou un pilote, privilégier les options WinRE adaptées : désinstallation de la dernière mise à jour, restauration système ou mode sans échec si disponible. Garder une hypothèse vérifiable et ne faire qu’une modification à la fois.

Ne pas désactiver globalement les contrôles de sécurité ni forcer un pilote incompatible « pour tester ». PnPUtil peut inventorier ou gérer les packages de pilotes depuis Windows ; toute suppression doit être ciblée, précédée d’un relevé du fournisseur, de la classe et du nom publié.

## 12. Codes et indices fréquents

| Indice | Piste à examiner |
|---|---|
| 0xc000000e, 0xc0000225 | BCD, périphérique de démarrage, partition ou chemin manquant |
| BOOTMGR is missing | Chemin BIOS/MBR ou partition système ; distinguer le firmware utilisé |
| INACCESSIBLE_BOOT_DEVICE / 0x7B | Contrôleur, mode AHCI/RAID, pilote de stockage, partition ou chiffrement |
| Boucle de réparation automatique | Lire le journal, vérifier disque, mise à jour, pilotes et fichiers système |
| Disque absent du firmware | Détection matérielle, alimentation, connexion, firmware ou panne du support |
| BSOD après le logo | Voir le [guide d’analyse des BSOD](../bsod/guide-analyse-bsod-windows-10-11.md) |

Un code oriente l’enquête ; il ne désigne pas toujours la cause racine.

## 13. Vérification après réparation

Après une action :

1. consigner commande, contexte, lettres de lecteur et résultat exact ;
2. redémarrer à froid si pertinent et confirmer le bon Windows Boot Manager ;
3. observer plusieurs démarrages, connexion et fonctionnement des périphériques essentiels ;
4. vérifier l’état BitLocker, les événements système et l’absence de nouvelle boucle ;
5. confirmer la présence des partitions et données ;
6. retirer les lettres temporaires et supports d’installation ;
7. noter les changements, incertitudes et plan de retour arrière.

## Références Microsoft

- [Résolution des problèmes de démarrage Windows](https://learn.microsoft.com/fr-fr/troubleshoot/windows-client/performance/windows-boot-issues-troubleshooting)
- [BCDBoot](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/bcdboot) et [options de ligne de commande](https://learn.microsoft.com/fr-fr/windows-hardware/manufacture/desktop/bcdboot-command-line-options-techref-di?view=windows-11)
- [Options BCDEdit](https://learn.microsoft.com/fr-fr/windows-hardware/manufacture/desktop/bcdedit-command-line-options?view=windows-11)
- [manage-bde -status](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/manage-bde-status) et [manage-bde -unlock](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/manage-bde-unlock)
- [SFC](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/sfc), [réparer une image Windows avec DISM](https://learn.microsoft.com/fr-fr/windows-hardware/manufacture/desktop/repair-a-windows-image?view=windows-11), [CHKDSK](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/chkdsk)
- [Options de récupération Windows](https://support.microsoft.com/fr-fr/windows/experience/backup-recovery/recovery-options-in-windows), [Réparation du démarrage](https://support.microsoft.com/fr-fr/windows/experience/startup-boot/startup-repair)
- [Erreur 7B / Inaccessible Boot Device](https://learn.microsoft.com/fr-fr/troubleshoot/windows-client/performance/stop-error-7b-or-inaccessible-boot-device-troubleshooting)

---
*Support pédagogique : adapter les exemples au poste observé. Toute opération d’écriture doit être précédée de l’identification du disque, du volume, du mode de démarrage et du chiffrement.*
