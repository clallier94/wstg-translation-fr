# Rapports

La réalisation de l'aspect technique de l'évaluation ne représente que la moitié du processus d'évaluation global. Le produit final est la production d'un rapport bien écrit et informatif. Un rapport doit être facile à comprendre et doit mettre en évidence tous les risques constatés lors de la phase d'évaluation. Le rapport devrait plaire à la fois à la direction générale et au personnel technique.

## À propos de cette section

Ce guide ne fournit que des suggestions sur une approche possible en matière de signalement et ne doit pas être considéré comme des règles strictes à suivre. Lorsque vous envisagez l'une des recommandations ci-dessous, demandez-vous toujours si la recommandation améliorerait votre rapport.

Ce guide de reporting est le mieux adapté aux rapports basés sur des conseils. Cela peut être exagéré pour les rapports internes ou de prime de bogue.

Quelle que soit l'audience, il est conseillé de sécuriser le rapport et de le crypter pour s'assurer que seul le destinataire est en mesure de l'utiliser.

Un bon rapport aide votre client à comprendre vos conclusions et met en évidence la qualité de vos tests techniques. La qualité des tests techniques est complètement hors de propos si le client ne peut pas comprendre vos conclusions.

## 1. Introduction

### 1.1 Contrôle des versions

Définit les modifications du rapport, principalement présentées sous forme de tableau comme ci-dessous.

| version | Descriptif | Rendez-vous | Auteur |
|:------- :|-------------|------|--------|
| 1.0 | Rapport initial | JJ/MM/AAAA | J. Doe |

### 1.2 Table des matières

Une page de table des matières pour le document.

### 1.3 L'équipe

Une liste des membres de l'équipe détaillant leur expertise et leurs qualifications.

### 1.4 Champ d'application

Les limites et les besoins de l'engagement convenus avec l'organisation.

### 1.5 Limites

Les limites peuvent être :

- Zones interdites par rapport aux tests.
- Fonctionnalité cassée.
- Manque de coopération.
- Manque de temps.
- Manque d'accès ou d'informations d'identification.

### 1.6 Chronologie

La durée de l'engagement.

### 1.7 Clause de non-responsabilité

Vous voudrez peut-être fournir une clause de non-responsabilité pour votre service. Consultez toujours un professionnel du droit afin de créer un document juridiquement contraignant.

L'exemple suivant est fourni à titre indicatif uniquement. Il ne doit pas être utilisé tel quel et ne constitue pas un avis juridique.

*Ce test est une évaluation "à un moment donné" et, en tant que tel, l'environnement peut avoir changé depuis que le test a été exécuté. Il n'y a aucune garantie que tous les problèmes de sécurité possibles ont été identifiés et que de nouvelles vulnérabilités ont pu être découvertes depuis l'exécution des tests. En tant que tel, ce rapport sert de document d'orientation et non de garantie que le rapport fournit une représentation complète des risques menaçant les systèmes en question.*

## 2. Résumé analytique

C'est comme l'elevator pitch du rapport, il vise à fournir aux dirigeants :

- L'objectif du test.
    - Décrire le besoin métier derrière le test de sécurité.
    - Décrivez comment les tests ont aidé l'organisation à comprendre ses systèmes.
- Principaux résultats dans un contexte commercial, tels que les éventuels problèmes de conformité, les atteintes à la réputation, etc. Concentrez-vous sur l'impact commercial et laissez de côté les détails techniques pour le moment.
- Les recommandations stratégiques sur la manière dont l'entreprise peut empêcher que les problèmes ne se reproduisent. Décrivez-les dans un contexte non technique et laissez de côté les recommandations techniques spécifiques pour le moment.

Le résumé doit être constructif et significatif. Évitez le jargon et les spéculations négatives. Si des figures, des graphiques ou des illustrations sont utilisés, assurez-vous qu'ils aident à transmettre un message d'une manière plus claire que le texte.

## 3. Résultats

Cette section est destinée à l'équipe technique. Il doit inclure toutes les informations nécessaires pour comprendre la vulnérabilité, la répliquer et la résoudre. La séparation logique peut aider à améliorer la lisibilité du rapport. Par exemple, vous pouvez avoir des sections distinctes intitulées "Accès externe" et "Accès interne".

S'il s'agit d'un nouveau test, vous pouvez créer une sous-section qui résume les résultats du test précédent, l'état mis à jour des vulnérabilités précédemment identifiées et toute référence croisée avec le test actuel.

### 3.1 Résumé des résultats

Une liste des résultats avec leur niveau de risque. Une table peut être utilisée pour faciliter l'utilisation par les deux équipes.

| Réf. identifiant | Titre | Niveau de risque |
|:-----------:|--------|------------|
| 1 | Contournement de l'authentification utilisateur | Haut |

### 3.2 Détails des résultats

Chaque constatation doit être détaillée avec les informations suivantes :

- ID de référence, qui peut être utilisé pour la communication entre les parties et pour les références croisées dans le rapport.
- Le titre de la vulnérabilité, tel que "Contournement de l'authentification de l'utilisateur".
- La probabilité ou l'exploitabilité du problème, en fonction de divers facteurs tels que :
    - Comme il est facile à exploiter.
    - S'il existe un code d'exploitation fonctionnel pour cela.
    - Le niveau d'accès requis.
    - Motivation de l'attaquant pour l'exploiter.
- L'impact de la vulnérabilité sur le système.
- Risque de vulnérabilité sur l'application.
    - Certaines valeurs suggérées sont : Informationnel, Faible, Moyen, Élevé et Critique. Assurez-vous de détailler les valeurs que vous décidez d'utiliser dans une annexe. Cela permet au lecteur de comprendre comment chaque score est déterminé.
    - Sur certains engagements, il est nécessaire d'avoir un score [CVSS](https://www.first.org/cvss/). S'il n'est pas nécessaire, il est parfois bon de l'avoir, et d'autres fois, cela ne fait qu'ajouter de la complexité au rapport.
- Description détaillée de ce qu'est la vulnérabilité, comment l'exploiter et les dommages qui peuvent résulter de son exploitation. Toutes les données éventuellement sensibles doivent être masquées, par exemple, les mots de passe, les informations personnelles ou les détails de la carte de crédit.
- Étapes détaillées sur la façon de remédier à la vulnérabilité, améliorations possibles qui pourraient aider à renforcer la posture de sécurité et pratiques de sécurité manquantes.
- Des ressources supplémentaires qui pourraient aider le lecteur à comprendre la vulnérabilité, comme une image, une vidéo, un CVE, un guide externe, etc.

Formatez cette section de manière à transmettre au mieux votre message.

Assurez-vous toujours que vos descriptions fournissent suffisamment d'informations pour que l'ingénieur lisant ce rapport puisse prendre des mesures en fonction de celles-ci. Expliquez soigneusement la constatation et fournissez autant de détails techniques que nécessaire pour y remédier.

## Annexes

Plusieurs annexes peuvent être ajoutées, telles que :

- Méthodologie de test utilisée.
- Explications sur la gravité et la cote de risque.
- Rendement pertinent des outils utilisés.
    - Assurez-vous de nettoyer la sortie et de ne pas simplement la vider.
- Une checklist de tous les tests effectués, comme la [liste de contrôle WSTG](https://github.com/OWASP/wstg/tree/master/checklists). Ceux-ci peuvent être fournis en pièces jointes au rapport.

## Références

Cette section ne fait pas partie du format de rapport suggéré. Les liens ci-dessous fournissent plus de conseils pour rédiger vos rapports.

- [SANS : Conseils pour créer un rapport d'évaluation de la cybersécurité solide](https://www.sans.org/blog/tips-for-creating-a-strong-cybersecurity-assessment-report/)
- [SANS : Rédaction d'un rapport de test d'intrusion](https://www.sans.org/reading-room/whitepapers/bestprac/paper/33343)
- [Institut Infosec : l'art de rédiger des rapports de test d'intrusion](https://resources.infosecinstitute.com/topic/writing-penetration-testing-reports/)
- [Pour les nuls : Comment structurer un rapport de test d'intrusion] (https://www.dummies.com/computers/macs/security/how-to-structure-a-pen-test-report/)
- [Rhino Security Labs : quatre choses que chaque rapport de test d'intrusion devrait avoir] (https://rhinosecuritylabs.com/penetration-testing/four-things-every-penetration-test-report/)
