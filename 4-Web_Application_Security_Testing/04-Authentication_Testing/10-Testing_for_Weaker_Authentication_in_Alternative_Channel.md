# Test d'authentification plus faible dans un canal alternatif

|ID          |
|------------|
|WSTG-ATHN-10|

## Sommaire

M�me si les m�canismes d'authentification primaires n'incluent aucune vuln�rabilit�, il se peut que des vuln�rabilit�s existent dans d'autres canaux d'utilisateurs l�gitimes d'authentification pour les m�mes comptes d'utilisateurs. Des tests doivent �tre entrepris pour identifier les canaux alternatifs et, sous r�serve de la port�e des tests, identifier les vuln�rabilit�s.

Les canaux d'interaction utilisateur alternatifs pourraient �tre utilis�s pour contourner le canal principal ou exposer des informations qui peuvent ensuite �tre utilis�es pour assister une attaque contre le canal principal. Certains de ces canaux peuvent eux-m�mes �tre des applications Web distinctes utilisant des noms d'h�te ou des chemins diff�rents. Par exemple:

- Site Web standard
- Mobile, ou appareil sp�cifique, site Web optimis�
- Site Web optimis� pour l'accessibilit�
- Sites Web alternatifs de pays et de langue
-�Sites Web parall�les qui utilisent les m�mes comptes d'utilisateurs (par exemple, un autre site Web offrant des fonctionnalit�s diff�rentes de la m�me organisation, un site Web partenaire avec lequel les comptes d'utilisateurs sont partag�s)
- D�veloppement, test, UAT et versions staging du site standard

Mais il peut �galement s'agir d'autres types d'applications ou de processus m�tier�:

- Application pour appareil mobile
- Application de bureau
- Op�rateurs de centres d'appels
- Syst�mes interactifs de r�ponse vocale ou d'arborescence t�l�phonique

Notez que ce test se concentre sur les canaux alternatifs�; certaines alternatives d'authentification peuvent appara�tre comme des contenus diff�rents diffus�s via le m�me site Web et seraient presque certainement susceptibles d'�tre test�es. Celles-ci ne sont pas d�crites plus en d�tail ici et auraient d� �tre identifi�es lors de la collecte d'informations et des tests d'authentification primaire. Par exemple:

- Enrichissement progressif et d�gradation gracieuse qui modifient les fonctionnalit�s
- Utilisation du site sans cookies
- Utilisation du site sans JavaScript
- Utilisation du site sans plugins comme pour Flash et Java

M�me si le p�rim�tre du test ne permet pas de tester les canaux alternatifs, leur existence doit �tre document�e. Ceux-ci peuvent miner le degr� d'assurance dans les m�canismes d'authentification et peuvent �tre un pr�curseur de tests suppl�mentaires.

## Exemple

Le site Web principal est `http://www.exemple.com` et les fonctions d'authentification ont toujours lieu sur les pages utilisant TLS `https://www.exemple.com/myaccount/`.

Cependant, il existe un site Web distinct optimis� pour les mobiles qui n'utilise pas du tout TLS et qui dispose d'un m�canisme de r�cup�ration de mot de passe plus faible `http://m.exemple.com/myaccount/`.

## Objectifs des tests

- Identifier les canaux d'authentification alternatifs.
- �valuer les mesures de s�curit� utilis�es et s'il existe des contournements sur les canaux alternatifs.

## Comment tester

### Comprendre le m�canisme principal

Testez enti�rement les principales fonctions d'authentification du site Web. Cela devrait identifier comment les comptes sont �mis, cr��s ou modifi�s et comment les mots de passe sont r�cup�r�s, r�initialis�s ou modifi�s. De plus, la connaissance de toute authentification � privil�ges �lev�s et des mesures de protection de l'authentification doit �tre connue. Ces pr�curseurs sont n�cessaires pour pouvoir comparer avec d'�ventuelles voies alternatives.

### Identifier d'autres cha�nes

D'autres canaux peuvent �tre trouv�s en utilisant les m�thodes suivantes�:

- Lire le contenu du site, en particulier la page d'accueil, nous contacter, les pages d'aide, les articles de support et les FAQ, les CGU, les avis de confidentialit�, le fichier robots.txt et tous les fichiers sitemap.xml.
- Recherche dans les journaux de proxy HTTP, enregistr�s lors de la collecte et des tests d'informations pr�c�dents, pour des cha�nes telles que "mobile", "android", blackberry", "ipad", "iphone", "application mobile", "e-reader", "sans fil ", "auth", "sso", "single sign on" dans les chemins d'URL et le contenu du corps.
- Utilisez les moteurs de recherche pour trouver diff�rents sites Web de la m�me organisation, ou utilisant le m�me nom de domaine, qui ont un contenu de page d'accueil similaire ou qui ont �galement des m�canismes d'authentification.

Pour chaque canal possible, confirmez si les comptes d'utilisateurs sont partag�s entre ceux-ci ou permettent d'acc�der � la m�me fonctionnalit� ou � une fonctionnalit� similaire.

### Enum�rer la fonctionnalit� d'authentification

Pour chaque canal alternatif o� les comptes d'utilisateurs ou les fonctionnalit�s sont partag�s, identifiez si toutes les fonctions d'authentification du canal principal sont disponibles et s'il existe des �l�ments suppl�mentaires. Il peut �tre utile de cr�er une grille comme celle ci-dessous�:

  |             Primaire             | Mobile | Centre d'appels | Site Web partenaire |
  |----------------------------------|--------|-----------------|---------------------|
  | Inscrivez-vous                   |  Oui   |        -        |          -          |
  | Se connecter                     |  Oui   |       Oui       |       Oui(SSO)      |
  | D�connectez-vous                 |   -    |        -        |          -          |
  | R�initialisation du mot de passe |  Oui   |       Oui       |          -          |
  | Changer le mot de passe          |        |        -        |          -          |

Dans cet exemple, le mobile a une fonction suppl�mentaire "changer le mot de passe" mais n'offre pas la "d�connexion". Un nombre limit� de t�ches est �galement possible en t�l�phonant au centre d'appels. Les centres d'appels peuvent �tre int�ressants, car leurs contr�les de confirmation d'identit� peuvent �tre plus faibles que ceux du site Web, ce qui permet d'utiliser ce canal pour faciliter une attaque contre le compte d'un utilisateur.

Tout en les �num�rant, il convient de prendre note de la fa�on dont la gestion des sessions est entreprise, en cas de chevauchement entre les canaux (par exemple, les cookies �tendus au m�me nom de domaine parent, les sessions simultan�es autoris�es sur les canaux, mais pas sur le m�me canal).

### Examen et test

Les canaux alternatifs doivent �tre mentionn�s dans le rapport de test, m�me s'ils sont marqu�s comme "informations uniquement" ou "hors champ". Dans certains cas, la port�e du test peut inclure le canal alternatif (par exemple, parce qu'il s'agit simplement d'un autre chemin sur le nom d'h�te cible), ou peut �tre ajout�e � la port�e apr�s discussion avec les propri�taires de tous les canaux. Si les tests sont autoris�s et autoris�s, tous les autres tests d'authentification de ce guide doivent alors �tre effectu�s et compar�s au canal principal.

## Cas de test associ�s

Les cas de test pour tous les autres tests d'authentification doivent �tre utilis�s.

## Correction

Assurez-vous qu'une politique d'authentification coh�rente est appliqu�e sur tous les canaux afin qu'ils soient �galement s�curis�s.
