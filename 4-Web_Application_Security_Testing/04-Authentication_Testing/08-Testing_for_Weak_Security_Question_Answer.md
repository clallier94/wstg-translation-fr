# Test de réponse à la question de sécurité faible

|ID          |
|------------|
|WSTG-ATHN-08|

## Sommaire

Souvent appelées questions et réponses "secrètes", les questions et réponses de sécurité sont souvent utilisées pour récupérer les mots de passe oubliés (voir [Test des fonctionnalités de changement ou de réinitialisation de mot de passe faible](09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md), ou comme sécurité supplémentaire en plus du mot de passe).

Ils sont généralement générés lors de la création du compte et obligent l'utilisateur à sélectionner parmi certaines questions pré-générées et à fournir une réponse appropriée. Ils peuvent permettre à l'utilisateur de générer ses propres paires de questions et de réponses. Les deux méthodes sont sujettes à des incertitudes. Idéalement, les questions de sécurité devraient générer des réponses qui ne sont connues que de l'utilisateur, et qui ne peuvent être devinées ou découvertes par personne d'autre. C'est plus difficile qu'il n'y paraît.
Les questions et réponses de sécurité reposent sur le secret de la réponse. Les questions et les réponses doivent être choisies de manière à ce que les réponses ne soient connues que du titulaire du compte. Cependant, bien que de nombreuses réponses ne soient pas connues du public, la plupart des questions mises en œuvre par les sites Web promeuvent des réponses pseudo-privées.

### Questions pré-générées

La majorité des questions pré-générées sont de nature assez simpliste et peuvent conduire à des réponses peu sûres. Par exemple:

- Les réponses peuvent être connues des membres de la famille ou des amis proches de l'utilisateur, par ex. "Quel est le nom de jeune fille de votre mère ?", "Quelle est votre date de naissance ?"
- Les réponses peuvent être facilement devinables, par ex. "Quelle est ta couleur préférée ?", "Quelle est ton équipe de baseball préférée ?"
- Les réponses peuvent être brutales, par ex. "Quel est le prénom de votre professeur de lycée préféré ?" - la réponse se trouve probablement sur certaines listes facilement téléchargeables de prénoms populaires, et donc une simple attaque par force brute peut être scénarisée.
- Les réponses peuvent être accessibles au public, par ex. "Quel est ton film préféré?" - la réponse peut facilement être trouvée sur la page de profil de l'utilisateur sur les réseaux sociaux.

### Questions auto-générées

Le problème avec le fait que les utilisateurs génèrent leurs propres questions est que cela leur permet de générer des questions très peu sûres, voire de contourner l'intérêt d'avoir une question de sécurité en premier lieu. Voici quelques exemples concrets qui illustrent ce point :

- "Combien fait 1+1 ?"
- "Quel est votre nom d'utilisateur?"
- "Mon mot de passe est S3curIty !"

## Objectifs des tests

- Déterminer la complexité et la simplicité des questions.
- Évaluer les réponses possibles des utilisateurs et les capacités de force brute.

## Comment tester

### Test des questions pré-générées faibles

Essayez d'obtenir une liste de questions de sécurité en créant un nouveau compte ou en suivant le processus "Je ne me souviens pas de mon mot de passe". Essayez de générer autant de questions que possible pour avoir une bonne idée du type de questions de sécurité qui sont posées. Si l'une des questions de sécurité entre dans les catégories décrites ci-dessus, elle est susceptible d'être attaquée (devinée, forcée, disponible sur les réseaux sociaux, etc.).

### Test des questions auto-générées faibles

Essayez de créer des questions de sécurité en créant un nouveau compte ou en configurant les propriétés de récupération de mot de passe de votre compte existant. Si le système permet à l'utilisateur de générer ses propres questions de sécurité, il est vulnérable à la création de questions non sécurisées. Si le système utilise les questions de sécurité auto-générées pendant la fonctionnalité de mot de passe oublié et si les noms d'utilisateur peuvent être énumérés (voir [Test d'énumération de compte et de compte d'utilisateur devinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)), alors il devrait être facile pour le testeur d'énumérer un certain nombre de questions auto-générées. On devrait s'attendre à trouver plusieurs questions auto-générées faibles en utilisant cette méthode.

### Tester les réponses brutales

Utilisez les méthodes décrites dans [Test du mécanisme de verrouillage faible](03-Testing_for_Weak_Lock_Out_Mechanism.md) pour déterminer si un certain nombre de réponses de sécurité fournies de manière incorrecte déclenchent un mécanisme de verrouillage.

La première chose à prendre en considération lorsque vous essayez d'exploiter les questions de sécurité est le nombre de questions auxquelles il faut répondre. La majorité des applications n'ont besoin que de l'utilisateur pour répondre à une seule question, alors que certaines applications critiques peuvent exiger que l'utilisateur réponde à deux ou même plusieurs questions.

L'étape suivante consiste à évaluer la force des questions de sécurité. Les réponses pourraient-elles être obtenues par une simple recherche sur Google ou par une attaque d'ingénierie sociale ? En tant que testeur d'intrusion, voici une procédure pas à pas pour exploiter un schéma de questions de sécurité :

- L'application permet-elle à l'utilisateur final de choisir la question à laquelle il doit répondre ? Si oui, concentrez-vous sur les questions qui ont :

    - Une réponse « publique » ; par exemple, quelque chose qui pourrait être trouvé avec une simple requête de moteur de recherche.
    - Une réponse factuelle telle qu'une "première école" ou d'autres faits qui peuvent être recherchés.
    - Peu de réponses possibles, comme "quel modèle était votre première voiture". Ces questions présenteraient à l'attaquant une courte liste de réponses possibles et, sur la base de statistiques, l'attaquant pourrait classer les réponses de la plus probable à la moins probable.

- Déterminez combien de suppositions vous avez si possible.
    - La réinitialisation du mot de passe permet-elle des tentatives illimitées ?
    - Y a-t-il une période de blocage après X réponses incorrectes ? Gardez à l'esprit qu'un système de verrouillage peut être un problème de sécurité en soi, car il peut être exploité par un attaquant pour lancer un déni de service contre des utilisateurs légitimes.
    - Choisissez la question appropriée en fonction de l'analyse des points ci-dessus et faites des recherches pour déterminer les réponses les plus probables.

La clé pour exploiter et contourner avec succès un schéma de questions de sécurité faible est de trouver une question ou un ensemble de questions qui donne la possibilité de trouver facilement les réponses. Recherchez toujours les questions qui peuvent vous donner la plus grande chance statistique de deviner la bonne réponse, si vous n'êtes absolument pas sûr de l'une des réponses. En fin de compte, un schéma de questions de sécurité n'est aussi fort que la question la plus faible.

## Références

- [La malédiction de la question secrète](https://www.schneier.com/essay-081.html)
- [La feuille de triche des questions de sécurité OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.html)
