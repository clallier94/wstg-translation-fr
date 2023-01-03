# Tester le téléchargement de types de fichiers inattendus

|ID          |
|------------|
|WSTG-BUSL-08|

## Sommaire

Les processus métier de nombreuses applications permettent le téléchargement et la manipulation de données soumises via des fichiers. Mais le processus métier doit vérifier les fichiers et n'autoriser que certains types de fichiers "approuvés". Décider quels fichiers sont "approuvés" est déterminé par la logique métier et est spécifique à l'application/au système. Le risque qu'en autorisant les utilisateurs à télécharger des fichiers, les attaquants puissent soumettre un type de fichier inattendu qui pourrait être exécuté et avoir un impact négatif sur l'application ou le système par le biais d'attaques susceptibles de dégrader le site Web, d'exécuter des commandes à distance, de parcourir les fichiers système, de parcourir les ressources locales, attaquer d'autres serveurs ou exploiter les vulnérabilités locales, pour n'en nommer que quelques-unes.

Les vulnérabilités liées au téléchargement de types de fichiers inattendus sont uniques en ce sens que le téléchargement devrait rapidement rejeter un fichier s'il n'a pas d'extension spécifique. De plus, cela diffère du téléchargement de fichiers malveillants dans la mesure où, dans la plupart des cas, un format de fichier incorrect peut ne pas être intrinsèquement "malveillant" mais peut nuire aux données enregistrées. Par exemple, si une application accepte les fichiers Windows Excel, si un fichier de base de données similaire est téléchargé, il peut être lu, mais les données extraites peuvent être déplacées vers des emplacements incorrects.

L'application peut s'attendre à ce que seuls certains types de fichiers soient téléchargés pour traitement, tels que les fichiers ".csv" ou ".txt". L'application peut ne pas valider le fichier téléchargé par extension (pour une validation de fichier à faible assurance) ou par son contenu (validation de fichier à haute assurance). Cela peut entraîner des résultats système ou de base de données inattendus dans l'application/le système ou donner aux attaquants des méthodes supplémentaires pour exploiter l'application/le système.

### Exemple

Supposons qu'une application de partage d'images permette aux utilisateurs de télécharger un fichier graphique `.gif` ou `.jpg` sur le site Web. Que se passe-t-il si un attaquant est capable de télécharger un fichier HTML contenant une balise `<script>` ou un fichier PHP ? Le système peut déplacer le fichier d'un emplacement temporaire vers l'emplacement final où le code PHP peut maintenant être exécuté sur l'application ou le système.

## Objectifs des tests

- Examinez la documentation du projet pour les types de fichiers qui sont rejetés par le système.
- Vérifiez que les types de fichiers indésirables sont rejetés et traités en toute sécurité.
- Vérifiez que les téléchargements par lots de fichiers sont sécurisés et n'autorisent aucun contournement des mesures de sécurité définies.

## Comment tester

### Méthode de test spécifique

- Étudier les exigences logiques des applications.
- Préparez une bibliothèque de fichiers "non approuvés" pour le téléchargement pouvant contenir des fichiers tels que : jsp, exe ou des fichiers HTML contenant un script.
- Dans l'application, accédez au mécanisme de soumission ou de téléchargement de fichiers.
- Soumettez le fichier "non approuvé" pour le téléchargement et vérifiez qu'il est correctement empêché de le télécharger
- Vérifiez si le site Web vérifie uniquement le type de fichier en JavaScript côté client
- Vérifiez si le site Web ne vérifie que le type de fichier par "Content-Type" dans la requête HTTP.
- Vérifiez si le site Web ne vérifie que l'extension de fichier.
- Vérifiez si d'autres fichiers téléchargés sont accessibles directement par l'URL spécifiée.
- Vérifiez si le fichier téléchargé peut inclure du code ou de l'injection de script.
- Vérifiez s'il existe un chemin de fichier vérifiant les fichiers téléchargés. En particulier, les pirates peuvent compresser les fichiers avec le chemin spécifié dans ZIP afin que les fichiers décompressés puissent être téléchargés vers le chemin prévu après le téléchargement et la décompression.

## Cas de test associés

- [Tester la gestion des extensions de fichiers pour les informations sensibles](../02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)
- [Tester le téléchargement de fichiers malveillants](09-Test_Upload_of_Malicious_Files.md)

## Correction

Les applications doivent être développées avec des mécanismes pour accepter et manipuler uniquement les fichiers "acceptables" que le reste de la fonctionnalité de l'application est prête à gérer et attend. Certains exemples spécifiques incluent : refuser des listes ou autoriser des listes d'extensions de fichiers, utiliser "Content-Type" dans l'en-tête ou utiliser un outil de reconnaissance de type de fichier, le tout pour autoriser uniquement les types de fichiers spécifiés dans le système.

## Références

- [OWASP - Téléchargement de fichiers sans restriction](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Meilleures pratiques de sécurité pour le téléchargement de fichiers : bloquer un téléchargement de fichier malveillant](https://www.computerweekly.com/answer/File-upload-security-best-practices-Block-a-malicious-file-upload)
- [Empêchez les gens de télécharger des fichiers PHP malveillants via des formulaires](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [CWE-434 : téléchargement illimité de fichiers de type dangereux](https://cwe.mitre.org/data/definitions/434.html)

