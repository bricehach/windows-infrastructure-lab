# Analyser les BSOD de Windows 10 et Windows 11

> Guide pédagogique avancé : analyse des écrans bleus, codes Bug Check et dumps mémoire. Les commandes sont celles de l’invite de commandes ou de WinDbg ; aucun script PowerShell n’est requis.

## Objectifs

L’apprenant doit pouvoir collecter un code STOP, ses paramètres, un dump et le contexte de l’incident ; distinguer un indice d’une preuve ; exploiter WinDbg avec les symboles Microsoft ; comparer plusieurs dumps et rechercher une cause commune ; enfin choisir une réparation proportionnée et vérifier son effet.

Les TP, scénarios et évaluations sont à concevoir séparément.

## 1. Comprendre un Bug Check

Un BSOD survient lorsque Windows détecte une condition critique qui compromet la poursuite fiable du système. Le code Bug Check (ou STOP), ses paramètres, le processus en cours, la pile d’appels, les modules chargés et le dump forment un ensemble d’indices.

Un code ne désigne pas toujours directement la pièce défectueuse. Une corruption mémoire peut provenir d’un pilote, d’un défaut matériel, d’un firmware, d’une alimentation instable ou d’un autre composant. ntoskrnl.exe apparaît souvent parce que le noyau détecte ou traite l’état fatal ; sa présence seule n’accuse pas le noyau.

## 2. Collecter les éléments avant de réparer

Consigner :

- date, heure, fréquence et conditions de reproduction ;
- texte exact du BSOD, code hexadécimal et paramètres ;
- version Windows, build, modèle et périphériques concernés ;
- changements récents : pilote, mise à jour, logiciel de sécurité, matériel, firmware ou réglages mémoire/CPU ;
- actions déjà tentées et leurs résultats ;
- événements système proches de l’incident ;
- chemin, taille, date et type du dump.

Emplacements usuels :

- petits dumps : %SystemRoot%\Minidump\ ;
- dump mémoire : %SystemRoot%\MEMORY.DMP.

Préserver une copie de travail sans modifier l’original. Un dump peut contenir des données confidentielles ou des fragments de mémoire vive. Ne pas le joindre à un ticket public ou à GitHub sans examen de confidentialité et autorisation.

Vérifier la configuration dans **Paramètres système avancés → Démarrage et récupération**. L’absence de dump peut venir d’un arrêt brutal, d’un fichier d’échange mal configuré, d’un manque d’espace, d’un problème de stockage ou des paramètres de collecte.

## 3. Journaux et historique de fiabilité

Dans l’Observateur d’événements, examiner **Journaux Windows → Système** autour de l’heure. Kernel-Power 41 ou EventLog 6008 signalent un arrêt inattendu ; ils n’en donnent pas seuls la cause. Chercher les événements précédant le redémarrage : pilote, disque, WHEA, mise à jour ou service.

Depuis CMD, pour afficher les 100 événements les plus récents :

~~~cmd
wevtutil qe System /c:100 /rd:true /f:text
~~~

Pour conserver une preuve structurée, exporter le journal depuis l’Observateur au format EVTX. L’historique de fiabilité (perfmon /rel) aide à aligner les plantages avec les installations ou mises à jour ; il fournit une chronologie, pas une preuve de cause.

## 4. Outils d’analyse

### WinDbg

WinDbg est l’outil de référence pour examiner un dump noyau. Installer la version actuelle depuis la documentation Microsoft et ouvrir une copie du dump. Pour charger les symboles publics :

~~~text
.symfix
.reload
!analyze -v
.bugcheck
kv
lm
~~~

Commandes utiles :

| Commande WinDbg | Rôle |
|---|---|
| !analyze -v | Analyse détaillée du Bug Check et hypothèses du débogueur |
| .bugcheck | Code et paramètres de l’arrêt |
| kv | Pile d’appels avec les informations disponibles |
| lm | Modules chargés |
| lmvm nom_module | Version, chemin et métadonnées d’un module |
| !thread | Informations sur le thread courant, selon le contexte |
| !irp, !devstack | Analyse ciblée d’une requête ou pile de périphérique, selon le contexte |

Une commande peut ne pas produire d’information utile si le dump ne fournit pas le contexte nécessaire. Lire le code, les paramètres, la pile, le module et la chronologie ensemble. Les extensions et symboles doivent correspondre aux versions concernées.

### Outils simplifiés

BlueScreenView ou WhoCrashed peuvent fournir un premier résumé du code et des modules associés. Ils ne remplacent pas WinDbg pour conclure. Vérifier les versions, la source de téléchargement et les données transmises à ces outils. Les présenter comme aides d’orientation, pas comme verdicts.

## 5. Méthode d’analyse d’un dump

1. **Confirmer le dump** : bon fichier, date, format et version Windows.
2. **Charger les symboles** et relancer l’analyse.
3. **Lire les paramètres** du Bug Check et les interpréter dans la documentation officielle.
4. **Examiner la pile** : fonctions impliquées et thread actif.
5. **Examiner le module** avec lmvm : fournisseur, version, horodatage et chemin.
6. **Relier au contexte** : pilote ou périphérique récent, événement système, conditions de charge.
7. **Comparer plusieurs dumps** : la répétition d’un module ou d’une famille d’erreurs renforce une piste, sans constituer seule une preuve.
8. **Formuler une hypothèse testable**, appliquer une correction ciblée, puis observer la reproduction.
9. **Documenter le résultat**, y compris l’absence de changement.

La ligne « Probably caused by » de !analyze -v est une piste automatisée. Un module tiers peut être victime d’une corruption introduite ailleurs ; un module Microsoft peut apparaître dans la pile sans être fautif.

## 6. Bug Checks fréquents : premières pistes

| Code | Nom courant | Pistes de départ (non conclusives) |
|---|---|---|
| 0x0000007B | INACCESSIBLE_BOOT_DEVICE | contrôleur, pilote de stockage, mode SATA/RAID, volume ou chiffrement |
| 0x0000000A | IRQL_NOT_LESS_OR_EQUAL | accès mémoire invalide, parfois lié à un pilote |
| 0x000000D1 | DRIVER_IRQL_NOT_LESS_OR_EQUAL | pilote et adresse fautive ; examiner pile et module |
| 0x00000050 | PAGE_FAULT_IN_NONPAGED_AREA | mémoire invalide, corruption, pilote ou RAM |
| 0x0000001A | MEMORY_MANAGEMENT | corruption mémoire ; vérifier pilotes, RAM et stabilité |
| 0x0000003B | SYSTEM_SERVICE_EXCEPTION | exception en service système ; examiner paramètres et pile |
| 0x0000007E | SYSTEM_THREAD_EXCEPTION_NOT_HANDLED | exception non gérée ; identifier thread et module |
| 0x0000009F | DRIVER_POWER_STATE_FAILURE | transition d’alimentation et périphérique/pilote bloqué |
| 0x00000116 | VIDEO_TDR_FAILURE | chemin graphique, pilote, GPU, température et alimentation |
| 0x00000124 | WHEA_UNCORRECTABLE_ERROR | erreur matérielle rapportée par WHEA ; analyser l’enregistrement |
| 0x00000101 | CLOCK_WATCHDOG_TIMEOUT | processeur ou plateforme ; examiner stabilité et firmware |
| 0x00000133 | DPC_WATCHDOG_VIOLATION | latence DPC, stockage ou pilote ; analyser paramètres et pile |
| 0x00000139 | KERNEL_SECURITY_CHECK_FAILURE | corruption d’une structure noyau ; rechercher une corruption antérieure |
| 0x000000EF | CRITICAL_PROCESS_DIED | processus critique arrêté ; examiner le contexte |

Toujours consulter la référence Microsoft du Bug Check et ses paramètres. Cette table oriente l’analyse ; elle ne détermine pas quelle pièce remplacer.

## 7. Pilotes et matériel

### Pilotes et logiciels

Vérifier éditeur, version, matériel associé, date de mise à jour, compatibilité avec la build Windows et existence d’un correctif ou retour arrière documenté. Restaurer ou désinstaller uniquement un composant soutenu par les indices. Éviter les mises à jour massives de pilotes pendant l’enquête.

### Mémoire, stockage et stabilité

Si les dumps et journaux orientent vers le matériel :

- retirer provisoirement overclocking/XMP/EXPO et revenir aux réglages nominaux ;
- exécuter un diagnostic mémoire approprié ;
- examiner alertes SMART et journaux du contrôleur avec des outils constructeur ;
- vérifier températures, alimentation et connexions dans les limites du matériel ;
- sauvegarder avant les tests d’écriture ou de contrainte.

Un test réussi n’exclut pas une panne intermittente. Confirmer une hypothèse matérielle par plusieurs indices et tests reproductibles.

## 8. Driver Verifier : usage contrôlé

Driver Verifier impose des contrôles sur les pilotes et peut provoquer un Bug Check. C’est un outil de diagnostic actif, pas un scanner passif. Ne pas l’activer sur une machine critique ou sans sauvegarde et accès à WinRE.

Précautions :

- sauvegarder les données, connaître l’état BitLocker et disposer de la clé ;
- cibler uniquement des pilotes tiers précis ; éviter la sélection globale ;
- prévoir l’accès à WinRE et le retour arrière ;
- ne pas laisser Verifier activé après collecte.

Activation depuis CMD administrateur :

~~~cmd
verifier
~~~

Utiliser l’interface pour choisir contrôles et pilotes selon une procédure validée. Pour réinitialiser la configuration :

~~~cmd
verifier /reset
~~~

Redémarrer et vérifier que le contrôle est désactivé. Si Verifier provoque une boucle, démarrer en mode sans échec/WinRE puis réinitialiser. Consulter la documentation Microsoft avant de choisir les contrôles.

## 9. Réparations proportionnées

| Indice convergent | Action à évaluer | Validation |
|---|---|---|
| Début après mise à jour | désinstaller la mise à jour ciblée ou restaurer le système | stabilité sur plusieurs démarrages |
| Module tiers récurrent | mettre à jour, restaurer ou retirer le pilote identifié | nouveaux dumps et version du pilote |
| 0x7B après changement stockage | vérifier mode firmware/contrôleur et pilote de stockage | démarrage sans changer la configuration au hasard |
| 0x116 et contexte graphique | remettre le pilote officiel, vérifier GPU/température/alimentation | charge contrôlée et absence de nouveaux dumps |
| WHEA / 0x124 | analyser l’enregistrement et confirmer matériel/firmware | tests et journaux après correction |
| Corruption système suspectée | SFC puis DISM dans le bon contexte | résultats SFC/DISM et stabilité |
| Arrêts sans dump, erreurs disque | protéger les données et examiner stockage/alimentation | journaux et tests constructeur |

Ne pas lancer simultanément plusieurs réparations non ciblées : cela brouille le lien entre hypothèse et résultat.

## 10. Rapport d’intervention

Un rapport utile contient : appareil et build Windows ; heure et symptômes ; Bug Check et paramètres ; chemin et hash du dump si requis ; événements associés ; changements récents ; outils et résultats ; hypothèse et indices pour/contre ; action, risque et retour arrière ; validation et suivi.

Retirer identifiants, noms, données utilisateur et secrets des captures et exports. Les dumps mémoire et clés BitLocker ne doivent pas être publiés dans le repository.

## Références Microsoft

- [Bug Checks / écrans bleus](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-checks--blue-screens-)
- [Référence des codes Bug Check](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-code-reference2)
- [Analyser un dump noyau avec WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/analyzing-a-kernel-mode-dump-file-with-windbg)
- [Commande !analyze](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-analyze)
- [Installer WinDbg](https://learn.microsoft.com/fr-fr/windows-hardware/drivers/debugger/)
- [Types de dumps noyau](https://learn.microsoft.com/en-gb/windows-hardware/drivers/debugger/varieties-of-kernel-mode-dump-files)
- [Driver Verifier](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/verifier)
- [Dépannage du démarrage Windows](https://learn.microsoft.com/fr-fr/troubleshoot/windows-client/performance/windows-boot-issues-troubleshooting)

---
*Support pédagogique Windows 10/11. Les analyses restent des hypothèses jusqu’à leur validation par des éléments reproductibles.*
