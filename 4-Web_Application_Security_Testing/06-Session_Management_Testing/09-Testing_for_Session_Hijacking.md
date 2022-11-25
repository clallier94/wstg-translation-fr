# Test de piratage de session

|ID          |
|------------|
|WSTG-SESS-09|

## Sommaire

Un attaquant qui acc�de aux cookies de session utilisateur peut se faire passer pour eux en pr�sentant ces cookies. Cette attaque est connue sous le nom de d�tournement de session. Lorsque l'on consid�re les attaquants de r�seau, c'est-�-dire les attaquants qui contr�lent le r�seau utilis� par la victime, les cookies de session peuvent �tre ind�ment expos�s � l'attaquant via HTTP. Pour �viter cela, les cookies de session doivent �tre marqu�s avec l'attribut "S�curis�" afin qu'ils ne soient communiqu�s que via HTTPS.

Notez que l'attribut `Secure` doit �galement �tre utilis� lorsque l'application Web est enti�rement d�ploy�e sur HTTPS, sinon l'attaque de vol de cookie suivante est possible. Supposons que `exemple.com` est enti�rement d�ploy� sur HTTPS, mais ne marque pas ses cookies de session comme `Secure`. Les �tapes d'attaque suivantes sont possibles�:

1. La victime envoie une requ�te � `http://another-site.com`.
2. L'attaquant corrompt la r�ponse correspondante pour qu'elle d�clenche une requ�te vers `http://exemple.com`.
3. Le navigateur essaie maintenant d'acc�der � `http://exemple.com`.
4. Bien que la demande �choue, les cookies de session sont divulgu�s en clair via HTTP.

Alternativement, le piratage de session peut �tre emp�ch� en interdisant l'utilisation de HTTP � l'aide de [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security). Notez qu'il y a ici une subtilit� li�e � la port�e des cookies. En particulier, l'adoption compl�te du HSTS est requise lorsque les cookies de session sont �mis avec l'ensemble d'attributs "Domaine".

L'adoption compl�te du HSTS est d�crite dans un article intitul� *Testing for Integrity Flaws in Web Sessions* par Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo et Michele Bugliesi. L'adoption compl�te du HSTS se produit lorsqu'un h�te active le HSTS pour lui-m�me et tous ses sous-domaines. L'adoption partielle du HSTS se produit lorsqu'un h�te active le HSTS uniquement pour lui-m�me.

Avec l'attribut `Domain` d�fini, les cookies de session peuvent �tre partag�s entre les sous-domaines. L'utilisation de HTTP avec des sous-domaines doit �tre �vit�e pour emp�cher la divulgation de cookies non chiffr�s envoy�s via HTTP. Pour illustrer cette faille de s�curit�, supposons que le site Web `exemple.com` active HSTS sans l'option `includeSubDomains`. Le site Web �met des cookies de session avec l'attribut `Domain` d�fini sur `exemple.com`. L'attaque suivante est possible :

1. La victime envoie une requ�te � `http://another-site.com`.
2. L'attaquant corrompt la r�ponse correspondante pour qu'elle d�clenche une requ�te vers `http://fake.exemple.com`.
3. Le navigateur essaie maintenant d'acc�der � `http://fake.exemple.com`, ce qui est autoris� par la configuration HSTS.
4. �tant donn� que la requ�te est envoy�e � un sous-domaine de `exemple.com` avec l'attribut `Domain` d�fini, elle inclut les cookies de session, qui sont divulgu�s en clair via HTTP.

Le HSTS complet doit �tre activ� sur le domaine apex pour emp�cher cette attaque.

## Objectifs des tests

- Identifier les cookies de session vuln�rables.
- D�tourner les cookies vuln�rables et �valuer le niveau de risque.

## Comment tester

La strat�gie de test est cibl�e sur les attaquants du r�seau, elle ne doit donc �tre appliqu�e qu'aux sites sans adoption compl�te du HSTS (les sites avec une adoption compl�te du HSTS sont s�curis�s, car leurs cookies ne sont pas communiqu�s via HTTP). Nous supposons que nous avons deux comptes de test sur le site Web test�, un pour agir en tant que victime et un pour agir en tant qu'attaquant. Nous simulons un sc�nario dans lequel l'attaquant vole tous les cookies qui ne sont pas prot�g�s contre la divulgation via HTTP et les pr�sente au site Web pour acc�der au compte de la victime. Si ces cookies sont suffisants pour agir au nom de la victime, le d�tournement de session est possible.

Voici les �tapes pour ex�cuter ce test :

1. Connectez-vous au site en tant que victime et acc�dez � toute page proposant une fonction s�curis�e n�cessitant une authentification.
2. Supprimez de la bo�te � cookies tous les cookies qui satisfont � l'une des conditions suivantes.
    - s'il n'y a pas d'adoption du HSTS�: l'attribut "S�curis�" est d�fini.
    - en cas d'adoption partielle du HSTS�: l'attribut `Secure` est d�fini ou l'attribut `Domain` n'est pas d�fini.
3. Enregistrez un instantan� de la bo�te � biscuits.
4. D�clenchez la fonction s�curis�e identifi�e � l'�tape 1.
5. Observez si l'op�ration � l'�tape 4 a �t� effectu�e avec succ�s. Si c'est le cas, l'attaque a r�ussi.
6. Effacez la bo�te � cookies, connectez-vous en tant qu'attaquant et acc�dez � la page � l'�tape 1.
7. �crivez dans la bo�te � biscuits, un par un, les cookies enregistr�s � l'�tape 3.
8. D�clenchez � nouveau la fonction s�curis�e identifi�e � l'�tape 1.
9. Videz la bo�te � cookies et reconnectez-vous en tant que victime.
10. Observez si l'op�ration � l'�tape 8 a �t� effectu�e avec succ�s dans le compte de la victime. Si tel est le cas, l'attaque a r�ussi; sinon, le site est s�curis� contre le d�tournement de session.

Nous recommandons d'utiliser deux machines ou navigateurs diff�rents pour la victime et l'attaquant. Cela vous permet de r�duire le nombre de faux positifs si l'application Web prend des empreintes digitales pour v�rifier l'acc�s activ� � partir d'un cookie donn�. Une variante plus courte mais moins pr�cise de la strat�gie de test ne n�cessite qu'un seul compte de test. Il suit le m�me sch�ma, mais il s'arr�te � l'�tape 5 (notez que cela rend l'�tape 3 inutile).

## Outils

- [OWASP ZAP] (https://www.zaproxy.org)
- [JHijack - un outil de piratage de session num�rique](https://sourceforge.net/projects/jhijack/)
