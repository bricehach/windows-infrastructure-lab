# Réparation du démarrage Windows

Guide avancé pour le diagnostic et la réparation raisonnée du démarrage sous **Windows 10 et Windows 11**.

## Guide

- [Réparer le démarrage de Windows 10 et Windows 11](guide-avance-demarrage-windows-10-11.md)

## Contenu

- Chaînes UEFI/GPT et BIOS/MBR
- Triage, WinRE, lettres de volumes et BitLocker
- Inventaire des disques/partitions avec DiskPart
- Réparation automatisée, BCDBoot et portée de BOOTREC
- Vérification BCD, SFC, DISM et CHKDSK
- Codes d’erreur fréquents, validation et documentation

Les exemples doivent être adaptés à la machine observée. Le guide distingue les opérations de lecture des commandes qui écrivent sur le disque. Les TP, scénarios et évaluations sont séparés.

## Principe

Identifier le mode de démarrage, le bon disque et les bons volumes avant toute réparation. Préserver les données et la clé BitLocker. Vérifier chaque résultat.

[Retour à l’index du repository](../README.md)
