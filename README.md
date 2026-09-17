# Windows Infrastructure Lab

## Administration • Sécurité • Dépannage • Documentation

Ce repository fait partie de **[Projet 00 — Portfolio professionnel LinkedIn + GitHub]**.

Il a pour objectif de transformer des travaux réels d'administration, de dépannage et de sécurisation Windows en une base de connaissances **pratique, vérifiable et réutilisable**.

> **Comprendre avant de modifier. Vérifier après chaque changement. Documenter le retour arrière.**

---

## Objectif

Ce dépôt ne cherche pas à constituer une simple collection de commandes ou de scripts. Chaque ressource doit permettre de comprendre :

- ce que fait Windows ;
- pourquoi une modification est envisagée ;
- quels sont les risques ou effets secondaires possibles ;
- comment vérifier le résultat ;
- comment revenir en arrière lorsque cela est possible.

Les sujets couverts concernent principalement **Windows 10 et Windows 11 Pro**, l'administration poste de travail, le hardening, la maintenance, le diagnostic et le troubleshooting.

---

## Méthode documentaire

```text
Contexte
   ↓
Objectif
   ↓
Prérequis
   ↓
Comprendre le mécanisme
   ↓
Procédure
   ↓
Vérification
   ↓
Risques / limites
   ↓
Retour arrière
   ↓
Troubleshooting
   ↓
Ce qu'il faut retenir
```

Les contenus seront classés selon quatre formats :

| Type | Utilité |
|---|---|
| **GUIDE** | Réaliser une tâche de manière reproductible |
| **REFERENCE** | Retrouver rapidement commandes, paramètres et informations |
| **LAB** | Reproduire un diagnostic ou une expérience |
| **EXPLAINED** | Comprendre un mécanisme, ses effets et ses limites |

---

## Structure du repository

| Domaine | Contenu |
|---|---|
| [`installation/`](installation/) | Installation propre, compte local, post-installation |
| [`privacy-telemetry/`](privacy-telemetry/) | Confidentialité, télémétrie, collecte et transparence |
| [`winget/`](winget/) | Maintenance et mise à jour des applications |
| [`drivers-bios/`](drivers-bios/) | Pilotes, BIOS, compatibilité et diagnostic |
| [`boot-repair/`](boot-repair/) | Réparation du démarrage Windows |
| [`bsod/`](bsod/) | BSOD, Bug Check, dump et diagnostic |
| [`event-logs/`](event-logs/) | Journaux Windows et Event ID utiles |
| [`defender-hardening/`](defender-hardening/) | Microsoft Defender, GPO et hardening |
| [`troubleshooting/`](troubleshooting/) | Cas de dépannage documentés |

---

## Principes de sécurité et de fiabilité

Avant publication, chaque procédure doit être relue selon les critères suivants :

1. **Utilité** — apporte-t-elle une solution ou une compréhension concrète ?
2. **Exactitude** — commandes, paramètres et effets ont-ils été vérifiés ?
3. **Contexte** — versions Windows, éditions et prérequis sont-ils précisés ?
4. **Sécurité** — aucun secret, donnée privée ou information client n'est publié.
5. **Réversibilité** — le retour arrière est documenté lorsqu'il existe.
6. **Validation** — la procédure indique comment vérifier son résultat.
7. **Sources** — les éléments évolutifs sont reliés à des sources fiables lorsque nécessaire.

---

## Positionnement dans le portfolio

Ce repository complète les autres axes du portfolio :

- infrastructure et systèmes Windows ;
- cybersécurité / SOC ;
- DFIR et analyse de logs ;
- réseau et Linux ;
- automatisation ;
- IA appliquée à l'IT et à la cybersécurité.

L'objectif est de montrer non seulement **ce qui fonctionne**, mais aussi **comment diagnostiquer, sécuriser, vérifier et documenter un environnement Windows**.

---

## Statut

Repository actif et en construction progressive.

Les premiers contenus prévus concernent :

- l'installation de Windows 11 Pro ;
- le compte administrateur local ;
- la confidentialité et la télémétrie ;
- Winget ;
- la réparation du démarrage ;
- les BSOD ;
- les pilotes et le BIOS ;
- les Event ID de stabilité ;
- Microsoft Defender et le hardening ;
- le troubleshooting transversal.
