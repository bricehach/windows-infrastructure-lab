# Installation, mise à niveau et dépannage de Windows 10/11

> Notes de cours pour techniciens et étudiants. Les commandes sont exécutées dans l’invite de commandes (CMD), jamais dans PowerShell. Chaque procédure précise le contexte, le résultat attendu et les points d’arrêt.

## Objectifs pédagogiques

À la fin de ce cours, l’apprenant doit pouvoir :

- préparer une installation propre sans perdre les données par erreur ;
- distinguer installation propre, mise à niveau sur place et déploiement ;
- contrôler la compatibilité matérielle, le média, la langue et l’édition ;
- diagnostiquer un échec de Windows Setup à partir des journaux ;
- reconnaître les limites et risques d’une installation sur matériel non pris en charge ;
- vérifier le poste après l’installation et documenter les écarts.

Ce document contient des notes et procédures de référence. Les scénarios, TP et évaluations sont prévus séparément.

## Repère de cycle de vie

**Windows 10 — contexte hérité :** la prise en charge standard de Windows 10 22H2 Home/Pro a pris fin le 14 octobre 2025. Des règles distinctes existent notamment pour certaines éditions LTSC et pour le programme ESU. Toute intervention sur Windows 10 doit commencer par identifier précisément l’édition, la version et son statut de support.

**Windows 11 — contexte courant :** la configuration requise, les éditions disponibles et les versions proposées évoluent. Vérifier les informations Microsoft au moment de préparer le poste ; ne pas figer une liste de processeurs ou une version cible dans une procédure qui serait réutilisée plusieurs années.

## 1. Choisir le type d’opération

| Opération | Effet principal | Quand la choisir |
|---|---|---|
| **Mise à niveau sur place** | Remplace la version de Windows en conservant éventuellement fichiers, paramètres et applications | Migration d’un poste fonctionnel vers une version compatible |
| **Installation propre** | Installe Windows sur une partition/disque ciblé ; peut supprimer les données existantes | Nouveau poste, réinstallation planifiée ou système à repartir de zéro |
| **Réinitialisation / récupération** | Réinstalle Windows selon les options de récupération | Dépannage local après vérification des conséquences |
| **Déploiement géré** | Installe/configure plusieurs appareils avec une méthode standardisée | Parc d’entreprise, outillage de déploiement disponible |

La conservation des fichiers et applications dépend de la compatibilité de la version, de l’édition, de la langue, de l’architecture et du média. Une option « conserver » absente ou grisée est une information de diagnostic : ne pas forcer la suite en espérant que l’option apparaisse.

## 2. Préparation commune

Avant une mise à niveau ou une installation :

1. **Identifier le poste** : modèle générique, version et édition de Windows, architecture, stockage, mode de démarrage et périphériques essentiels.
2. **Sauvegarder et vérifier la restauration** des données nécessaires. Une sauvegarde non testée n’est pas une garantie.
3. **Vérifier l’activation et la licence** de l’édition qui sera installée.
4. **Contrôler l’état du disque** et l’espace disponible ; arrêter en cas d’erreurs d’E/S ou de signes de panne.
5. **Vérifier BitLocker** et s’assurer que la clé de récupération est disponible dans un emplacement autorisé.
6. **Déconnecter les périphériques non indispensables** à l’installation et garder l’alimentation secteur.
7. **Télécharger le média depuis une source Microsoft** et relever la version, l’architecture, la langue et l’édition.
8. **Noter la configuration de départ** pour comparer après l’opération.

CMD en administrateur :

~~~cmd
winver
systeminfo
dism /online /get-intl
manage-bde -status
~~~

**Résultats à relever :**

- winver ouvre une fenêtre avec version et build ;
- systeminfo affiche notamment le système, le type du système et les informations de démarrage ;
- DISM /Get-Intl affiche la langue système/locale active ; comparer avec la langue du média ;
- manage-bde -status indique les volumes chiffrés, leur méthode et leur état de protection.

Ces commandes fournissent des indices, pas un bilan complet de compatibilité. Les informations TPM et firmware peuvent aussi être consultées dans **Sécurité Windows**, **tpm.msc** et **msinfo32**. Les outils graphiques sont des compléments ; ce cours garde les commandes d’administration dans CMD.

## 3. Vérifier l’éligibilité de Windows 11

Avant une mise à niveau prise en charge, vérifier les exigences Microsoft actuelles, notamment processeur pris en charge, TPM 2.0, firmware UEFI, capacité Secure Boot, mémoire, stockage et compatibilité des pilotes. Ne pas conclure qu’un PC est éligible uniquement parce qu’il démarre l’installateur.

Pour contrôler la présence du TPM depuis CMD :

~~~cmd
tpm.msc
~~~

Cette commande ouvre la console de gestion TPM ; relever la version de spécification et l’état du module. Si la console indique qu’aucun TPM compatible n’est trouvé, il peut être absent ou désactivé dans le firmware ; vérifier les spécifications du fabricant avant de modifier l’UEFI.

Pour examiner les informations de firmware et Secure Boot :

~~~cmd
msinfo32
~~~

Dans Informations système, relever **Mode BIOS** et **État du démarrage sécurisé**. Ne pas changer Legacy/UEFI, Secure Boot, TPM ou mode SATA à l’aveugle : ces paramètres peuvent empêcher le démarrage ou déclencher BitLocker.

Les listes de processeurs compatibles sont maintenues par version de Windows 11. Confirmer la version cible sur la documentation Microsoft correspondante, et non sur une liste copiée dans une ancienne fiche de cours.

## 4. Vérifier le média et la langue

Un média doit correspondre à l’opération visée : architecture, langue, édition et version. Pour un parcours de conservation des applications, une différence de langue ou d’édition peut rendre l’option de conservation indisponible.

Sur le système installé :

~~~cmd
dism /online /get-intl
~~~

Sur un ISO, vérifier également la langue déclarée et l’image/édition contenue. Pour afficher les images présentes, repérer le lecteur monté et le fichier d’installation :

~~~cmd
dir E:\sources\install.*
dism /Get-ImageInfo /ImageFile:E:\sources\install.wim
~~~

Si le média contient install.esd, remplacer le nom du fichier par install.esd. Adapter E: à la lettre réelle. DISM affiche les index et les éditions du média ; comparer ces informations au plan d’installation. Ne pas deviner à partir du seul nom du fichier.

Pour une mise à niveau sur place, l’architecture, l’édition et la langue doivent être compatibles avec le Windows installé. Certaines options d’installation de Windows 11 22H2 et ultérieur exigent la même langue d’interface par défaut que la cible. Si le média provient d’un outil tiers, documenter l’outil et sa version, vérifier la provenance et ne pas mélanger ses modifications avec celles de Windows Setup dans un diagnostic initial.

**Rufus** peut servir à créer un média USB à partir d’un ISO ; c’est un outil tiers, distinct de Windows Setup. Choisir explicitement le schéma de partition et la cible firmware correspondant au poste. Une clé USB bootable ne rend pas éligible un PC qui ne respecte pas les exigences de Windows 11. Vérifier la documentation et les options de la version utilisée.

## 5. Mise à niveau sur place prise en charge

Pour migrer un poste Windows 10 éligible :

1. vérifier les exigences de Windows 11 et le statut proposé par Windows Update ;
2. sauvegarder, contrôler BitLocker, l’espace disque et les applications/pilotes critiques ;
3. installer les mises à jour préalables et redémarrer ;
4. utiliser Windows Update ou un média Microsoft compatible ;
5. lancer Setup depuis Windows si l’objectif est une mise à niveau sur place ;
6. lire l’écran de résumé et confirmer explicitement ce qui sera conservé ;
7. interrompre l’opération si l’édition, la langue ou les options de conservation ne correspondent pas au plan ;
8. après redémarrage, vérifier activation, édition, données, pilotes, chiffrement et événements.

Démarrer sur une clé USB lance généralement une installation/récupération depuis l’environnement de démarrage ; ce n’est pas la même opération que lancer une mise à niveau depuis la session Windows. Toujours confirmer la méthode avant de poursuivre.

## 6. Installation propre

1. Télécharger un ISO ou média officiel et confirmer sa version/édition/langue.
2. Créer le support d’installation et vérifier qu’il démarre en UEFI si le poste est prévu pour UEFI.
3. Démarrer sur le support et choisir les paramètres régionaux.
4. Vérifier l’édition et l’activation attendues.
5. À l’écran de sélection des disques, identifier sans ambiguïté le disque cible par sa capacité et son contenu attendu.
6. Ne supprimer ou formater des partitions que lorsque la réinstallation destructive est planifiée, que la sauvegarde a été vérifiée et que le disque cible est confirmé.
7. Terminer l’installation et noter les éventuels codes d’erreur.

**Point d’arrêt :** si plusieurs disques sont connectés et que le disque cible n’est pas certain, arrêter avant toute suppression. Ne pas enseigner « supprimer toutes les partitions » comme recette générale.

### USB et partitions

- UEFI utilise normalement un disque GPT et une partition système EFI.
- Les anciens environnements BIOS/MBR diffèrent dans leur chaîne de démarrage.
- Un réglage firmware incohérent avec le schéma du disque peut empêcher le démarrage.
- Ne pas convertir, nettoyer ou reformater un disque dans le but d’essayer au hasard.

Pour les détails de partitionnement et de chaîne d’amorçage, consulter le [guide de réparation du démarrage](../boot-repair/guide-avance-demarrage-windows-10-11.md).

## 7. Matériel non éligible : expérience séparée et non prise en charge

Microsoft ne recommande pas l’installation de Windows 11 sur un appareil qui ne satisfait pas aux exigences minimales. L’appareil peut ne pas recevoir de support Microsoft, rencontrer des problèmes de compatibilité et ne pas être garanti pour les mises à jour. Cette section sert à expliquer et documenter un essai de laboratoire ; elle ne constitue pas une procédure de déploiement recommandée.

### Vérifications complémentaires avec Coreinfo

Coreinfo est un utilitaire Microsoft Sysinternals. Depuis le dossier où l’outil a été téléchargé et extrait :

~~~cmd
coreinfo64.exe -f
~~~

Dans la liste des fonctionnalités, rechercher **SSE4.2** et **POPCNT**. L’astérisque indique que la fonction est présente sur le processeur ; un tiret indique qu’elle n’est pas disponible. La présence de ces deux fonctions ne valide pas l’ensemble des exigences ni la compatibilité d’une version donnée. La liste officielle des processeurs pris en charge et les exigences complètes restent la référence.

### Retour d’expérience de test

Un test antérieur a utilisé un ISO de même architecture et de même langue, vérifié la langue avec DISM, puis lancé une variante de Setup depuis les fichiers extraits. La commande observée était :

~~~cmd
setupprep.exe /product server
~~~

Cette option n’est pas présentée ici comme un chemin Microsoft pris en charge. Le résultat dépend de l’ISO, de la version, de la machine et de Setup ; l’écran de conservation peut changer ou disparaître. **Arrêter si l’option de conservation attendue est absente.** Ne pas accepter une installation qui supprime fichiers ou applications au lieu de l’opération prévue. Avant tout essai, sauvegarder, conserver la clé BitLocker et noter la méthode exacte pour pouvoir expliquer les écarts.

Ne pas généraliser ce résultat à d’autres builds, ni promettre qu’un processeur compatible SSE4.2/POPCNT rendra l’appareil conforme. Garder le parcours officiellement pris en charge et l’essai non pris en charge dans des sections distinctes.

## 8. Dépanner un échec de Setup

### Ordre de diagnostic

1. relever le code d’erreur complet, l’étape et le pourcentage ;
2. noter si l’échec se produit pendant téléchargement, préparation, premier redémarrage, migration ou retour arrière ;
3. ne pas relancer immédiatement la mise à niveau sans lire les journaux ;
4. examiner l’espace disque, les périphériques, le média, la langue, l’édition et les pilotes ;
5. analyser les journaux avec SetupDiag ;
6. ne changer qu’un facteur à la fois et conserver les résultats.

### SetupDiag depuis CMD

Télécharger la version courante depuis Microsoft, ouvrir CMD en administrateur, puis exécuter depuis le dossier où se trouve le fichier :

~~~cmd
SetupDiag.exe /Output:C:\SetupDiag\Results.log
~~~

SetupDiag analyse les journaux de Setup/Upgrade et écrit un rapport. Le chemin de sortie doit exister ou être adapté ; lire le rapport et identifier la règle associée avant d’appliquer un correctif.

Pour analyser hors ligne un dossier de journaux copié :

~~~cmd
SetupDiag.exe /Output:C:\SetupDiag\Results.log /LogsPath:D:\Temp\Logs
~~~

Adapter tous les chemins aux lettres réellement présentes. Dans WinRE, les lettres de lecteur peuvent différer de celles du Windows installé.

### Journaux à conserver

Selon la phase et la version, examiner notamment :

- C:\$Windows.~BT\Sources\Panther\setupact.log
- C:\$Windows.~BT\Sources\Panther\setuperr.log
- C:\$Windows.~BT\Sources\Rollback\
- C:\Windows\Panther\

Les journaux Setup sont souvent volumineux et peuvent révéler des chemins, noms d’appareils ou informations système. Les assainir avant partage public.

## 9. Installation et vérification des pilotes

Après l’installation :

1. terminer Windows Update et redémarrer ;
2. ouvrir le Gestionnaire de périphériques et relever les périphériques inconnus ou en erreur ;
3. rechercher les pilotes auprès du fabricant du PC ou du composant ;
4. installer un pilote à la fois et noter la version ;
5. éviter les utilitaires génériques d’« update driver » qui installent plusieurs composants sans diagnostic ;
6. conserver le pilote précédent ou le moyen de retour arrière lorsqu’il est disponible.

Une mise à niveau BIOS/UEFI est une opération séparée : vérifier le modèle exact, la version, les prérequis du fabricant, l’alimentation et le chiffrement avant de la planifier. Ne jamais interrompre une mise à jour firmware.

## 10. Contrôles post-installation

Après une installation ou mise à niveau, valider :

- version/build avec winver ;
- édition et activation ;
- ouverture de session, réseau, audio, affichage, veille et périphériques essentiels ;
- Gestionnaire de périphériques sans erreurs non expliquées ;
- état Windows Update ;
- état BitLocker et présence de la clé de récupération ;
- espace disque, sauvegarde, restauration et données conservées ;
- journaux autour de l’installation et absence de redémarrages inattendus.

CMD :

~~~cmd
winver
systeminfo
manage-bde -status
wevtutil qe System /c:50 /rd:true /f:text
~~~

Le résultat doit être comparé aux notes prises avant intervention. Si des données ou applications sont manquantes, arrêter les nettoyages (par exemple suppression de Windows.old) avant d’évaluer le retour arrière.

## 11. Documentation minimale de l’intervention

Consigner :

- date, appareil et statut de support ;
- version/source/architecture/langue/édition du média ;
- méthode : mise à niveau, installation propre, média USB ou essai hors prise en charge ;
- contrôles de compatibilité et état BitLocker ;
- code d’erreur, journaux et rapport SetupDiag ;
- données sauvegardées et vérification de la restauration ;
- changements de firmware, pilotes et partitions ;
- contrôles post-installation, réserves et retour arrière.

Ne pas publier de clé BitLocker, numéro de série, nom d’utilisateur, dump ou journal contenant des informations identifiantes.

## Sources officielles et outils

- [Configuration requise pour Windows 11](https://learn.microsoft.com/en-us/windows/whats-new/windows-11-requirements)
- [Méthodes d’installation de Windows 11](https://support.microsoft.com/fr-fr/windows/deployment/install-upgrade/ways-to-install-windows-11)
- [Installation de Windows 11 sur un appareil non conforme](https://support.microsoft.com/fr-fr/windows/experience/compatibility/windows-11-on-devices-that-don-t-meet-minimum-system-requirements)
- [Chemins de mise à niveau Windows](https://learn.microsoft.com/fr-fr/windows/deployment/upgrade/windows-upgrade-paths)
- [Options de ligne de commande de Windows Setup](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-command-line-options?view=windows-10)
- [Informations sur les images avec DISM](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/dism-image-management-command-line-options-s14?view=windows-11)
- [SetupDiag](https://learn.microsoft.com/fr-fr/windows/deployment/upgrade/setupdiag)
- [Journaux de mise à niveau Windows](https://learn.microsoft.com/en-us/windows/deployment/upgrade/log-files)
- [Fin du support de Windows 10](https://learn.microsoft.com/en-us/lifecycle/products/windows-10-home-and-pro)
- [Coreinfo — Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/coreinfo)
- [Créer un support d’installation Windows](https://support.microsoft.com/fr-fr/windows/experience/install-windows/create-installation-media-for-windows)

---
*Notes de cours. Vérifier les versions et consignes du fabricant avant chaque intervention. Windows 10 est explicitement traité comme un système hérité ; les expériences non prises en charge doivent rester identifiées comme telles.*
