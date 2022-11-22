# Le cadre de test de s�curit� Web

## Aper�u

Cette section d�crit un cadre de test typique qui peut �tre d�velopp� au sein d'une organisation. Il peut �tre consid�r� comme un cadre de r�f�rence compos� de techniques et de t�ches appropri�es � diff�rentes phases du cycle de vie du d�veloppement logiciel (SDLC). Les entreprises et les �quipes de projet peuvent utiliser ce mod�le pour d�velopper leur propre cadre de test et pour d�finir les services de test des fournisseurs. Ce cadre ne doit pas �tre consid�r� comme normatif, mais comme une approche flexible qui peut �tre �tendue et fa�onn�e pour s'adapter au processus de d�veloppement et � la culture d'une organisation.

Cette section vise � aider les organisations � mettre en place un processus de test strat�gique complet et ne s'adresse pas aux consultants ou aux sous-traitants qui ont tendance � �tre engag�s dans des domaines de test plus tactiques et sp�cifiques.

Il est essentiel de comprendre pourquoi la cr�ation d'un cadre de test de bout en bout est cruciale pour �valuer et am�liorer la s�curit� des logiciels. Dans *Writing Secure Code*, Howard et LeBlanc notent que l'�mission d'un bulletin de s�curit� co�te � Microsoft au moins 100 000 $, et qu'il en co�te collectivement beaucoup plus � leurs clients pour mettre en �uvre les correctifs de s�curit�. Ils notent �galement que le [site Web CyberCrime] du gouvernement am�ricain (https://www.justice.gov/criminal-ccips) d�taille les affaires p�nales r�centes et les pertes subies par les organisations. Les pertes typiques d�passent de loin les 100 000 USD.

Avec une �conomie comme celle-ci, il n'est pas �tonnant que les �diteurs de logiciels passent de l'ex�cution de tests de s�curit� en bo�te noire uniquement, qui ne peuvent �tre effectu�s que sur des applications d�j� d�velopp�es, � une concentration sur les tests dans les premiers cycles de d�veloppement d'applications, comme pendant d�finition, conception et d�veloppement.

De nombreux praticiens de la s�curit� voient encore les tests de s�curit� dans le domaine des tests d'intrusion. Comme indiqu� dans le chapitre pr�c�dent, si les tests d'intrusion ont un r�le � jouer, ils sont g�n�ralement inefficaces pour trouver des bogues et d�pendent excessivement des comp�tences du testeur. Il ne doit �tre consid�r� que comme une technique de mise en �uvre, ou pour sensibiliser aux probl�mes de production. Pour am�liorer la s�curit� des applications, la qualit� de s�curit� des logiciels doit �tre am�lior�e. Cela signifie tester la s�curit� pendant les �tapes de d�finition, de conception, de d�veloppement, de d�ploiement et de maintenance, et ne pas compter sur la strat�gie co�teuse d'attendre que le code soit compl�tement construit.

Comme indiqu� dans l'introduction de ce document, il existe de nombreuses m�thodologies de d�veloppement, telles que le processus unifi� rationnel, le d�veloppement eXtreme et Agile et les m�thodologies traditionnelles en cascade. L'intention de ce guide n'est pas de sugg�rer une m�thodologie de d�veloppement particuli�re, ni de fournir des conseils sp�cifiques qui adh�rent � une m�thodologie particuli�re. Au lieu de cela, nous pr�sentons un mod�le de d�veloppement g�n�rique, et le lecteur doit le suivre en fonction du processus de son entreprise.

Ce cadre de test se compose d'activit�s qui doivent avoir lieu�:

- Avant le d�but du d�veloppement,
- Lors de la d�finition et de la conception,
- Au cours du d�veloppement,
- Pendant le d�ploiement, et
- Pendant la maintenance et les op�rations.

## Phase 1 avant le d�but du d�veloppement

### Phase 1.1 D�finir un SDLC

Avant le d�but du d�veloppement d'applications, un SDLC ad�quat doit �tre d�fini o� la s�curit� est inh�rente � chaque �tape.

### Phase 1.2 Examiner les politiques et les normes

Assurez-vous que des politiques, des normes et une documentation appropri�es sont en place. La documentation est extr�mement importante car elle donne aux �quipes de d�veloppement des directives et des politiques qu'elles peuvent suivre. Les gens ne peuvent faire la bonne chose que s'ils savent quelle est la bonne chose.

Si l'application doit �tre d�velopp�e en Java, il est indispensable qu'il existe un standard de codage s�curis� Java. Si l'application doit utiliser la cryptographie, il est essentiel qu'il existe une norme de cryptographie. Aucune politique ou norme ne peut couvrir toutes les situations auxquelles l'�quipe de d�veloppement sera confront�e. En documentant les probl�mes communs et pr�visibles, il y aura moins de d�cisions � prendre pendant le processus de d�veloppement.

### Phase 1.3 D�velopper des crit�res de mesure et de m�trique et assurer la tra�abilit�

Avant le d�but du d�veloppement, planifiez le programme de mesure. En d�finissant les crit�res qui doivent �tre mesur�s, il offre une visibilit� sur les d�fauts du processus et du produit. Il est essentiel de d�finir les m�triques avant le d�but du d�veloppement, car il peut �tre n�cessaire de modifier le processus afin de capturer les donn�es.

## Phase 2 pendant la d�finition et la conception

### Phase 2.1 Examen des exigences de s�curit�

Les exigences de s�curit� d�finissent le fonctionnement d'une application du point de vue de la s�curit�. Il est essentiel que les exigences de s�curit� soient test�es. Tester dans ce cas signifie tester les hypoth�ses qui sont faites dans les exigences et tester pour voir s'il y a des lacunes dans les d�finitions des exigences.

Par exemple, s'il existe une exigence de s�curit� qui stipule que les utilisateurs doivent �tre enregistr�s avant de pouvoir acc�der � la section des livres blancs d'un site Web, cela signifie-t-il que l'utilisateur doit �tre enregistr� aupr�s du syst�me ou doit-il �tre authentifi�? Assurez-vous que les exigences sont aussi claires que possible.

Lorsque vous recherchez des lacunes dans les exigences, envisagez d'examiner les m�canismes de s�curit� tels que�:

- Gestion des utilisateurs
- Authentification
- Autorisation
- Confidentialit� des donn�es
- Int�grit�
- Responsabilit�
- Gestion des sessions
- S�curit� des transports
- S�gr�gation du syst�me � plusieurs niveaux
- Conformit� aux lois et aux normes (y compris les normes de confidentialit�, gouvernementales et de l'industrie)

### Phase 2.2 Examen de la conception et de l'architecture

Les applications doivent avoir une conception et une architecture document�es. Cette documentation peut inclure des mod�les, des documents textuels et d'autres artefacts similaires. Il est essentiel de tester ces artefacts pour s'assurer que la conception et l'architecture appliquent le niveau de s�curit� appropri� tel que d�fini dans les exigences.

L'identification des failles de s�curit� dans la phase de conception n'est pas seulement l'un des endroits les plus rentables pour identifier les failles, mais peut �tre l'un des endroits les plus efficaces pour apporter des modifications. Par exemple, s'il est identifi� que la conception n�cessite que les d�cisions d'autorisation soient prises � plusieurs endroits, il peut �tre appropri� d'envisager un composant d'autorisation central. Si l'application effectue la validation des donn�es � plusieurs endroits, il peut �tre appropri� de d�velopper un cadre de validation central (c'est-�-dire que fixer la validation des entr�es � un seul endroit, plut�t qu'� des centaines d'endroits, est beaucoup moins cher).

Si des faiblesses sont d�couvertes, elles doivent �tre communiqu�es � l'architecte du syst�me pour des approches alternatives.

### Phase 2.3 Cr�er et r�viser des mod�les UML

Une fois la conception et l'architecture termin�es, cr�ez des mod�les UML (Unified Modeling Language) qui d�crivent le fonctionnement de l'application. Dans certains cas, ceux-ci peuvent d�j� �tre disponibles. Utilisez ces mod�les pour confirmer avec les concepteurs de syst�mes une compr�hension exacte du fonctionnement de l'application. Si des faiblesses sont d�couvertes, elles doivent �tre communiqu�es � l'architecte du syst�me pour des approches alternatives.

### Phase 2.4 Cr�er et r�viser des mod�les de menace

Arm� des revues de conception et d'architecture et des mod�les UML expliquant exactement comment le syst�me fonctionne, entreprenez un exercice de mod�lisation des menaces. �laborez des sc�narios de menace r�alistes. Analysez la conception et l'architecture pour vous assurer que ces menaces ont �t� att�nu�es, accept�es par l'entreprise ou attribu�es � un tiers, tel qu'une compagnie d'assurance. Lorsque les menaces identifi�es n'ont pas de strat�gie d'att�nuation, revoyez la conception et l'architecture avec l'architecte syst�me pour modifier la conception.

## Phase 3 pendant le d�veloppement

Th�oriquement, le d�veloppement est la mise en �uvre d'un design. Cependant, dans le monde r�el, de nombreuses d�cisions de conception sont prises lors du d�veloppement du code. Il s'agit souvent de d�cisions plus modestes qui �taient soit trop d�taill�es pour �tre d�crites dans la conception, soit de questions pour lesquelles aucune politique ou orientation standard n'a �t� propos�e. Si la conception et l'architecture n'�taient pas ad�quates, le d�veloppeur sera confront� � de nombreuses d�cisions. S'il n'y avait pas suffisamment de politiques et de normes, le d�veloppeur sera confront� � encore plus de d�cisions.

### Proc�dure pas � pas du code de la phase 3.1

L'�quipe de s�curit� doit effectuer une visite guid�e du code avec les d�veloppeurs et, dans certains cas, les architectes syst�me. Une proc�dure pas � pas de code est un examen de haut niveau du code au cours duquel les d�veloppeurs peuvent expliquer la logique et le flux du code impl�ment�. Cela permet � l'�quipe de r�vision du code d'obtenir une compr�hension g�n�rale du code et permet aux d�veloppeurs d'expliquer pourquoi certaines choses ont �t� d�velopp�es comme elles l'ont �t�.

Le but n'est pas d'effectuer une revue de code, mais de comprendre � un niveau �lev� le flux, la mise en page et la structure du code qui compose l'application.

### Examens du code de la phase 3.2

Arm� d'une bonne compr�hension de la structure du code et de la raison pour laquelle certaines choses ont �t� cod�es comme elles l'�taient, le testeur peut maintenant examiner le code r�el � la recherche de d�fauts de s�curit�.

Les revues de code statiques valident le code par rapport � un ensemble de listes de contr�le, notamment�:

- Exigences commerciales en mati�re de disponibilit�, de confidentialit� et d'int�grit�;
- Guide OWASP ou Top 10 des listes de contr�le pour les expositions techniques (selon la profondeur de l'examen);
- Probl�mes sp�cifiques li�s au langage ou au framework utilis�, tels que le papier Scarlet pour PHP ou [listes de contr�le Microsoft Secure Coding pour ASP.NET] (https://msdn.microsoft.com/en-us/library/ff648269.aspx ); et
- Toutes les exigences sp�cifiques � l'industrie, telles que Sarbanes-Oxley 404, COPPA, ISO/IEC 27002, APRA, HIPAA, directives Visa Merchant ou autres r�gimes r�glementaires.

En termes de retour sur les ressources investies (principalement en temps), les revues de code statiques produisent des retours de bien meilleure qualit� que toute autre m�thode de revue de s�curit� et reposent moins sur les comp�tences de l'examinateur. Cependant, ils ne sont pas une solution miracle et doivent �tre examin�s avec soin dans le cadre d'un r�gime de tests � spectre complet.

Pour plus de d�tails sur les listes de contr�le OWASP, veuillez vous r�f�rer � la derni�re �dition du [OWASP Top 10](https://owasp.org/www-project-top-ten/).

## Phase 4 pendant le d�ploiement

### Phase 4.1 Test de p�n�tration des applications

Apr�s avoir test� les exigences, analys� la conception et effectu� la r�vision du code, on peut supposer que tous les probl�mes ont �t� d�tect�s. Esp�rons que ce soit le cas, mais les tests de p�n�tration de l'application apr�s son d�ploiement fournissent une v�rification suppl�mentaire pour s'assurer que rien n'a �t� manqu�.

### Phase 4.2 Test de gestion de la configuration

Le test de p�n�tration de l'application doit inclure un examen de la fa�on dont l'infrastructure a �t� d�ploy�e et s�curis�e. Il est important de revoir les aspects de configuration, aussi petits soient-ils, pour s'assurer qu'aucun n'est laiss� � un param�tre par d�faut qui pourrait �tre vuln�rable � l'exploitation.

## Phase 5 pendant la maintenance et les op�rations

### Phase 5.1 Mener des revues de gestion op�rationnelle

Il doit y avoir un processus en place qui d�taille comment le c�t� op�rationnel de l'application et de l'infrastructure est g�r�.

### Phase 5.2 Effectuer des bilans de sant� p�riodiques

Des contr�les de sant� mensuels ou trimestriels doivent �tre effectu�s sur l'application et l'infrastructure pour s'assurer qu'aucun nouveau risque de s�curit� n'a �t� introduit et que le niveau de s�curit� est toujours intact.

### Phase 5.3 Assurer la v�rification des modifications

Une fois que chaque modification a �t� approuv�e et test�e dans l'environnement QA et d�ploy�e dans l'environnement de production, il est essentiel que la modification soit v�rifi�e pour s'assurer que le niveau de s�curit� n'a pas �t� affect� par la modification. Cela devrait �tre int�gr� dans le processus de gestion du changement.

## Un flux de travail de test SDLC typique

La figure suivante montre un flux de travail de test SDLC typique.

![Flux de travail de test SDLC typique](images/Typical_SDLC_Testing_Workflow.gif)\
 *Figure 3-1 : Flux de travail de test SDLC typique*
