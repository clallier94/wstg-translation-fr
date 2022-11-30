# Tester le t�l�chargement de types de fichiers inattendus

|ID          |
|------------|
|WSTG-BUSL-08|

## Sommaire

Les processus m�tier de nombreuses applications permettent le t�l�chargement et la manipulation de donn�es soumises via des fichiers. Mais le processus m�tier doit v�rifier les fichiers et n'autoriser que certains types de fichiers "approuv�s". D�cider quels fichiers sont "approuv�s" est d�termin� par la logique m�tier et est sp�cifique � l'application/au syst�me. Le risque qu'en autorisant les utilisateurs � t�l�charger des fichiers, les attaquants puissent soumettre un type de fichier inattendu qui pourrait �tre ex�cut� et avoir un impact n�gatif sur l'application ou le syst�me par le biais d'attaques susceptibles de d�grader le site Web, d'ex�cuter des commandes � distance, de parcourir les fichiers syst�me, de parcourir les ressources locales, attaquer d'autres serveurs ou exploiter les vuln�rabilit�s locales, pour n'en nommer que quelques-unes.

Les vuln�rabilit�s li�es au t�l�chargement de types de fichiers inattendus sont uniques en ce sens que le t�l�chargement devrait rapidement rejeter un fichier s'il n'a pas d'extension sp�cifique. De plus, cela diff�re du t�l�chargement de fichiers malveillants dans la mesure o�, dans la plupart des cas, un format de fichier incorrect peut ne pas �tre intrins�quement "malveillant" mais peut nuire aux donn�es enregistr�es. Par exemple, si une application accepte les fichiers Windows Excel, si un fichier de base de donn�es similaire est t�l�charg�, il peut �tre lu, mais les donn�es extraites peuvent �tre d�plac�es vers des emplacements incorrects.

L'application peut s'attendre � ce que seuls certains types de fichiers soient t�l�charg�s pour traitement, tels que les fichiers ".csv" ou ".txt". L'application peut ne pas valider le fichier t�l�charg� par extension (pour une validation de fichier � faible assurance) ou par son contenu (validation de fichier � haute assurance). Cela peut entra�ner des r�sultats syst�me ou de base de donn�es inattendus dans l'application/le syst�me ou donner aux attaquants des m�thodes suppl�mentaires pour exploiter l'application/le syst�me.

### Exemple

Supposons qu'une application de partage d'images permette aux utilisateurs de t�l�charger un fichier graphique `.gif` ou `.jpg` sur le site Web. Que se passe-t-il si un attaquant est capable de t�l�charger un fichier HTML contenant une balise `<script>` ou un fichier PHP�? Le syst�me peut d�placer le fichier d'un emplacement temporaire vers l'emplacement final o� le code PHP peut maintenant �tre ex�cut� sur l'application ou le syst�me.

## Objectifs des tests

- Examinez la documentation du projet pour les types de fichiers qui sont rejet�s par le syst�me.
- V�rifiez que les types de fichiers ind�sirables sont rejet�s et trait�s en toute s�curit�.
- V�rifiez que les t�l�chargements par lots de fichiers sont s�curis�s et n'autorisent aucun contournement des mesures de s�curit� d�finies.

## Comment tester

### M�thode de test sp�cifique

- �tudier les exigences logiques des applications.
- Pr�parez une biblioth�que de fichiers "non approuv�s" pour le t�l�chargement pouvant contenir des fichiers tels que�: jsp, exe ou des fichiers HTML contenant un script.
- Dans l'application, acc�dez au m�canisme de soumission ou de t�l�chargement de fichiers.
- Soumettez le fichier "non approuv�" pour le t�l�chargement et v�rifiez qu'il est correctement emp�ch� de le t�l�charger
- V�rifiez si le site Web v�rifie uniquement le type de fichier en JavaScript c�t� client
- V�rifiez si le site Web ne v�rifie que le type de fichier par "Content-Type" dans la requ�te HTTP.
- V�rifiez si le site Web ne v�rifie que l'extension de fichier.
- V�rifiez si d'autres fichiers t�l�charg�s sont accessibles directement par l'URL sp�cifi�e.
- V�rifiez si le fichier t�l�charg� peut inclure du code ou de l'injection de script.
- V�rifiez s'il existe un chemin de fichier v�rifiant les fichiers t�l�charg�s. En particulier, les pirates peuvent compresser les fichiers avec le chemin sp�cifi� dans ZIP afin que les fichiers d�compress�s puissent �tre t�l�charg�s vers le chemin pr�vu apr�s le t�l�chargement et la d�compression.

## Cas de test associ�s

- [Tester la gestion des extensions de fichiers pour les informations sensibles](../02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)
- [Tester le t�l�chargement de fichiers malveillants] (09-Test_Upload_of_Malicious_Files.md)

## Correction

Les applications doivent �tre d�velopp�es avec des m�canismes pour accepter et manipuler uniquement les fichiers "acceptables" que le reste de la fonctionnalit� de l'application est pr�te � g�rer et attend. Certains exemples sp�cifiques incluent�: refuser des listes ou autoriser des listes d'extensions de fichiers, utiliser "Content-Type" dans l'en-t�te ou utiliser un outil de reconnaissance de type de fichier, le tout pour autoriser uniquement les types de fichiers sp�cifi�s dans le syst�me.

## R�f�rences

- [OWASP - T�l�chargement de fichiers sans restriction](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Meilleures pratiques de s�curit� pour le t�l�chargement de fichiers�: bloquer un t�l�chargement de fichier malveillant](https://www.computerweekly.com/answer/File-upload-security-best-practices-Block-a-malicious-file-upload)
- [Emp�chez les gens de t�l�charger des fichiers PHP malveillants via des formulaires] (https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [CWE-434�: t�l�chargement illimit� de fichiers de type dangereux](https://cwe.mitre.org/data/definitions/434.html)

