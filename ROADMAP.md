# Roadmap — Windows Infrastructure Lab

Ce document organise la transformation progressive des sujets déjà travaillés en ressources GitHub utiles, vérifiables et réutilisables.

## Priorités de publication

1. Compte administrateur local et vérifications post-installation
2. Confidentialité, télémétrie et transparence des données
3. Winget et maintenance applicative
4. Pilotes, BIOS et firmware
5. Journaux Windows et Event ID de stabilité
6. Microsoft Defender, GPO et hardening
7. Troubleshooting transversal

## Déjà publié

- [Audit et maintenance Windows 10/11 avec CMD](audit-cmd/) — guide pédagogique en Markdown et Word.
- [Installation, mise à niveau et dépannage Windows 10/11](installation/) — notes de cours en Markdown.
- [Réparation avancée du démarrage Windows 10/11](boot-repair/) — guide pédagogique en Markdown.
- [Analyse avancée des BSOD Windows 10/11](bsod/) — guide pédagogique en Markdown.

Le guide d’installation sépare le parcours pris en charge des essais sur matériel non éligible. Les exercices, TP et évaluations restent à créer dans des ressources distinctes.

## Règle documentaire

Chaque ressource doit idéalement suivre cette logique :

~~~text
Contexte
   ↓
Objectif
   ↓
Prérequis
   ↓
Comprendre
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
À retenir
~~~

## Statuts

- **Prévu** : sujet identifié
- **En rédaction** : contenu en cours de structuration
- **Publié** : ressource disponible dans le repository
- **À revoir** : contenu à actualiser ou à compléter

## Objectif à moyen terme

Construire une base Windows qui puisse servir à la fois de :

- support d'apprentissage ;
- référence rapide ;
- guide de dépannage ;
- documentation de sécurité ;
- preuve technique dans le portfolio professionnel.
