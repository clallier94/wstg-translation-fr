# Test de la gestion incorrecte des erreurs

|ID          |
|------------|
|WSTG-ERRH-01|

## Sommaire

Tous les types d'applications (applications Web, serveurs Web, bases de donn�es, etc.) g�n�reront des erreurs pour diverses raisons. Les d�veloppeurs ignorent souvent la gestion de ces erreurs ou repoussent l'id�e qu'un utilisateur essaiera un jour de d�clencher une erreur � dessein (*par exemple* en envoyant une cha�ne l� o� un entier est attendu). Lorsque le d�veloppeur ne consid�re que le chemin heureux, il oublie toutes les autres entr�es utilisateur possibles que le code peut recevoir mais ne peut pas g�rer.

Les erreurs augmentent parfois comme�:

- traces de pile,
- les timeouts du r�seau,
- d�calage d'entr�e,
- et les vidages m�moire.

Une mauvaise gestion des erreurs peut permettre aux attaquants de�:

- Comprendre les API utilis�es en interne.
- Cartographier les diff�rents services s'int�grant les uns aux autres en obtenant un aper�u des syst�mes internes et des cadres utilis�s, ce qui ouvre la porte � l'encha�nement des attaques.
- Recueillir les versions et les types d'applications utilis�es.
- DoS le syst�me en for�ant le syst�me dans une impasse ou une exception non g�r�e qui envoie un signal de panique au moteur qui l'ex�cute.
- Contr�le le contournement lorsqu'une certaine exception n'est pas limit�e par la logique d�finie autour du chemin heureux.

## Objectifs des tests

- Identifier la sortie d'erreur existante.
- Analyser les diff�rentes sorties retourn�es.

## Comment tester

Les erreurs sont g�n�ralement consid�r�es comme b�nignes car elles fournissent des donn�es de diagnostic et des messages qui pourraient aider l'utilisateur � comprendre le probl�me en question ou permettre au d�veloppeur de d�boguer cette erreur.

En essayant d'envoyer des donn�es inattendues ou en for�ant le syst�me dans certains cas et sc�narios extr�mes, le syst�me ou l'application d�voilera la plupart du temps un peu ce qui se passe en interne, � moins que les d�veloppeurs ne d�sactivent toutes les erreurs possibles et ne renvoient un certain message personnalis�. .

### Serveurs Web

Toutes les applications Web s'ex�cutent sur un serveur Web, qu'il soit int�gr� ou � part enti�re. Les applications Web doivent g�rer et analyser les requ�tes HTTP, et pour cela, un serveur Web fait toujours partie de la pile. Certains des serveurs Web les plus connus sont NGINX, Apache et IIS.

Les serveurs Web ont des messages d'erreur et des formats connus. Si l'on n'est pas familier avec leur apparence, une recherche en ligne pour eux fournirait des exemples. Une autre fa�on serait de consulter leur documentation ou simplement de configurer un serveur localement et de d�couvrir les erreurs en parcourant les pages utilis�es par le serveur Web.

Pour d�clencher des messages d'erreur, un testeur doit�:

- Rechercher des fichiers et dossiers al�atoires qui ne seront pas trouv�s (404s).
- Essayez de demander des dossiers qui existent et voyez le comportement du serveur (403, page blanche ou liste de r�pertoires).
- Essayez d'envoyer une requ�te qui rompt le [HTTP RFC](https://tools.ietf.org/html/rfc7231). Un exemple serait d'envoyer un tr�s grand chemin, de casser le format des en-t�tes ou de changer la version HTTP.
    - M�me si les erreurs sont g�r�es au niveau de l'application, la rupture de la RFC HTTP peut faire appara�tre le serveur Web int�gr� puisqu'il doit g�rer la requ�te, et les d�veloppeurs oublient de remplacer ces erreurs.

### Applications

Les applications sont les plus susceptibles d'�mettre une grande vari�t� de messages d'erreur, notamment�: des traces de pile, des vidages m�moire, des exceptions mal g�r�es et des erreurs g�n�riques. Cela est d� au fait que les applications sont construites sur mesure la plupart du temps et que les d�veloppeurs doivent observer et g�rer tous les cas d'erreur possibles (ou disposer d'un m�canisme global de capture d'erreurs), et ces erreurs peuvent appara�tre � partir d'int�grations avec d'autres services.

Pour qu'une application g�n�re ces erreurs, un testeur doit�:

1. Identifiez les points d'entr�e possibles o� l'application attend des donn�es.
2. Analysez le type d'entr�e attendu (cha�nes, entiers, JSON, XML, etc.).
3. Fuzzez chaque point d'entr�e en fonction des �tapes pr�c�dentes pour avoir un sc�nario de test plus cibl�.
   - Fuzzer chaque entr�e avec toutes les injections possibles n'est pas la meilleure solution, sauf si vous disposez d'un temps de test illimit� et que l'application peut g�rer autant d'entr�es.
   - Si le fuzzing n'est pas une option, s�lectionnez manuellement les entr�es viables qui ont le plus de chances de casser un certain analyseur (*par exemple* un crochet fermant pour un corps JSON, un texte volumineux o� seuls quelques caract�res sont attendus, injection CLRF avec param�tres qui pourraient �tre analys�s par les serveurs et les contr�les de validation d'entr�e, les caract�res sp�ciaux qui ne s'appliquent pas aux noms de fichiers, etc.).
   - Le fuzzing avec des donn�es de jargon doit �tre ex�cut� pour chaque type, car parfois les interpr�teurs sortiront de la gestion des exceptions du d�veloppeur.
4. Comprenez le service qui r�pond avec le message d'erreur et essayez de cr�er une liste fuzz plus pr�cise pour faire ressortir plus d'informations ou de d�tails sur l'erreur de ce service (il peut s'agir d'une base de donn�es, d'un service autonome, etc.).

Les messages d'erreur sont parfois la principale faiblesse de la cartographie des syst�mes, en particulier sous une architecture de microservices. Si les services ne sont pas correctement configur�s pour g�rer les erreurs de mani�re g�n�rique et uniforme, les messages d'erreur permettent � un testeur d'identifier quel service g�re quelles requ�tes et permettent une attaque plus cibl�e par service.

> Le testeur doit garder un �il vigilant sur le type de r�ponse. Parfois, les erreurs sont renvoy�es comme un succ�s avec un corps d'erreur, masquent l'erreur dans un 302 ou simplement en ayant une mani�re personnalis�e de repr�senter cette erreur.

## Correction

Pour rem�dier aux probl�mes, consultez les [Contr�les proactifs C10](https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions) et la [Fiche de triche pour la gestion des erreurs](https:/ /cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html).

## Cours de r�cr�ation

- [Juice Shop - Gestion des erreurs](https://pwning.owasp-juice.shop/part2/security-misconfiguration.html#provoke-an-error-that-is-nither-very-gracefully-ni-consistently-handled )

## R�f�rences

- [WSTG : Annexe C - Vecteurs Fuzz](../../6-Appendix/C-Fuzz_Vectors.md)
- [Contr�les proactifs C10�: g�rer toutes les erreurs et exceptions] (https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions)
- [ASVS v4.1 v7.4�: gestion des erreurs](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x15-V7-Error-Logging.md#v74-error-handling)
- [CWE 728 - Traitement incorrect des erreurs](https://cwe.mitre.org/data/definitions/728.html)
- [Cheat Sheet Series�: Gestion des erreurs] (https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
