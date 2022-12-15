# Le cadre de test de sécurité Web

## Aperçu

Cette section décrit un cadre de test typique qui peut être développé au sein d'une organisation. Il peut être considéré comme un cadre de référence composé de techniques et de tâches appropriées à différentes phases du cycle de vie du développement logiciel (SDLC). Les entreprises et les équipes de projet peuvent utiliser ce modèle pour développer leur propre cadre de test et pour définir les services de test des fournisseurs. Ce cadre ne doit pas être considéré comme normatif, mais comme une approche flexible qui peut être étendue et façonnée pour s'adapter au processus de développement et à la culture d'une organisation.

Cette section vise à aider les organisations à mettre en place un processus de test stratégique complet et ne s'adresse pas aux consultants ou aux sous-traitants qui ont tendance à être engagés dans des domaines de test plus tactiques et spécifiques.

Il est essentiel de comprendre pourquoi la création d'un cadre de test de bout en bout est cruciale pour évaluer et améliorer la sécurité des logiciels. Dans *Writing Secure Code*, Howard et LeBlanc notent que l'émission d'un bulletin de sécurité coûte à Microsoft au moins 100 000 $, et qu'il en coûte collectivement beaucoup plus à leurs clients pour mettre en œuvre les correctifs de sécurité. Ils notent également que le [site Web CyberCrime](https://www.justice.gov/criminal-ccips) du gouvernement américain  détaille les affaires pénales récentes et les pertes subies par les organisations. Les pertes typiques dépassent de loin les 100 000 USD.

Avec une économie comme celle-ci, il n'est pas étonnant que les éditeurs de logiciels passent de l'exécution de tests de sécurité en boîte noire uniquement, qui ne peuvent être effectués que sur des applications déjà développées, à une concentration sur les tests dans les premiers cycles de développement d'applications, comme pendant définition, conception et développement.

De nombreux praticiens de la sécurité voient encore les tests de sécurité dans le domaine des tests d'intrusion. Comme indiqué dans le chapitre précédent, si les tests d'intrusion ont un rôle à jouer, ils sont généralement inefficaces pour trouver des bogues et dépendent excessivement des compétences du testeur. Il ne doit être considéré que comme une technique de mise en œuvre, ou pour sensibiliser aux problèmes de production. Pour améliorer la sécurité des applications, la qualité de sécurité des logiciels doit être améliorée. Cela signifie tester la sécurité pendant les étapes de définition, de conception, de développement, de déploiement et de maintenance, et ne pas compter sur la stratégie coûteuse d'attendre que le code soit complètement construit.

Comme indiqué dans l'introduction de ce document, il existe de nombreuses méthodologies de développement, telles que le processus unifié rationnel, le développement eXtreme et Agile et les méthodologies traditionnelles en cascade. L'intention de ce guide n'est pas de suggérer une méthodologie de développement particulière, ni de fournir des conseils spécifiques qui adhèrent à une méthodologie particulière. Au lieu de cela, nous présentons un modèle de développement générique, et le lecteur doit le suivre en fonction du processus de son entreprise.

Ce cadre de test se compose d'activités qui doivent avoir lieu :

- Avant le début du développement,
- Lors de la définition et de la conception,
- Au cours du développement,
- Pendant le déploiement, et
- Pendant la maintenance et les opérations.

## Phase 1 avant le début du développement

### Phase 1.1 Définir un SDLC

Avant le début du développement d'applications, un SDLC adéquat doit être défini où la sécurité est inhérente à chaque étape.

### Phase 1.2 Examiner les politiques et les normes

Assurez-vous que des politiques, des normes et une documentation appropriées sont en place. La documentation est extrêmement importante car elle donne aux équipes de développement des directives et des politiques qu'elles peuvent suivre. Les gens ne peuvent faire la bonne chose que s'ils savent quelle est la bonne chose.

Si l'application doit être développée en Java, il est indispensable qu'il existe un standard de codage sécurisé Java. Si l'application doit utiliser la cryptographie, il est essentiel qu'il existe une norme de cryptographie. Aucune politique ou norme ne peut couvrir toutes les situations auxquelles l'équipe de développement sera confrontée. En documentant les problèmes communs et prévisibles, il y aura moins de décisions à prendre pendant le processus de développement.

### Phase 1.3 Développer des critères de mesure et de métrique et assurer la traçabilité

Avant le début du développement, planifiez le programme de mesure. En définissant les critères qui doivent être mesurés, il offre une visibilité sur les défauts du processus et du produit. Il est essentiel de définir les métriques avant le début du développement, car il peut être nécessaire de modifier le processus afin de capturer les données.

## Phase 2 pendant la définition et la conception

### Phase 2.1 Examen des exigences de sécurité

Les exigences de sécurité définissent le fonctionnement d'une application du point de vue de la sécurité. Il est essentiel que les exigences de sécurité soient testées. Tester dans ce cas signifie tester les hypothèses qui sont faites dans les exigences et tester pour voir s'il y a des lacunes dans les définitions des exigences.

Par exemple, s'il existe une exigence de sécurité qui stipule que les utilisateurs doivent être enregistrés avant de pouvoir accéder à la section des livres blancs d'un site Web, cela signifie-t-il que l'utilisateur doit être enregistré auprès du système ou doit-il être authentifié ? Assurez-vous que les exigences sont aussi claires que possible.

Lorsque vous recherchez des lacunes dans les exigences, envisagez d'examiner les mécanismes de sécurité tels que :

- Gestion des utilisateurs
- Authentification
- Autorisation
- Confidentialité des données
- Intégrité
- Responsabilité
- Gestion des sessions
- Sécurité des transports
- Ségrégation du système à plusieurs niveaux
- Conformité aux lois et aux normes (y compris les normes de confidentialité, gouvernementales et de l'industrie)

### Phase 2.2 Examen de la conception et de l'architecture

Les applications doivent avoir une conception et une architecture documentées. Cette documentation peut inclure des modèles, des documents textuels et d'autres artefacts similaires. Il est essentiel de tester ces artefacts pour s'assurer que la conception et l'architecture appliquent le niveau de sécurité approprié tel que défini dans les exigences.

L'identification des failles de sécurité dans la phase de conception n'est pas seulement l'un des endroits les plus rentables pour identifier les failles, mais peut être l'un des endroits les plus efficaces pour apporter des modifications. Par exemple, s'il est identifié que la conception nécessite que les décisions d'autorisation soient prises à plusieurs endroits, il peut être approprié d'envisager un composant d'autorisation central. Si l'application effectue la validation des données à plusieurs endroits, il peut être approprié de développer un cadre de validation central (c'est-à-dire que fixer la validation des entrées à un seul endroit, plutôt qu'à des centaines d'endroits, est beaucoup moins cher).

Si des faiblesses sont découvertes, elles doivent être communiquées à l'architecte du système pour des approches alternatives.

### Phase 2.3 Créer et réviser des modèles UML

Une fois la conception et l'architecture terminées, créez des modèles UML (Unified Modeling Language) qui décrivent le fonctionnement de l'application. Dans certains cas, ceux-ci peuvent déjà être disponibles. Utilisez ces modèles pour confirmer avec les concepteurs de systèmes une compréhension exacte du fonctionnement de l'application. Si des faiblesses sont découvertes, elles doivent être communiquées à l'architecte du système pour des approches alternatives.

### Phase 2.4 Créer et réviser des modèles de menace

Armé des revues de conception et d'architecture et des modèles UML expliquant exactement comment le système fonctionne, entreprenez un exercice de modélisation des menaces. Élaborez des scénarios de menace réalistes. Analysez la conception et l'architecture pour vous assurer que ces menaces ont été atténuées, acceptées par l'entreprise ou attribuées à un tiers, tel qu'une compagnie d'assurance. Lorsque les menaces identifiées n'ont pas de stratégie d'atténuation, revoyez la conception et l'architecture avec l'architecte système pour modifier la conception.

## Phase 3 pendant le développement

Théoriquement, le développement est la mise en œuvre d'un design. Cependant, dans le monde réel, de nombreuses décisions de conception sont prises lors du développement du code. Il s'agit souvent de décisions plus modestes qui étaient soit trop détaillées pour être décrites dans la conception, soit de questions pour lesquelles aucune politique ou orientation standard n'a été proposée. Si la conception et l'architecture n'étaient pas adéquates, le développeur sera confronté à de nombreuses décisions. S'il n'y avait pas suffisamment de politiques et de normes, le développeur sera confronté à encore plus de décisions.

### Procédure pas à pas du code de la phase 3.1

L'équipe de sécurité doit effectuer une visite guidée du code avec les développeurs et, dans certains cas, les architectes système. Une procédure pas à pas de code est un examen de haut niveau du code au cours duquel les développeurs peuvent expliquer la logique et le flux du code implémenté. Cela permet à l'équipe de révision du code d'obtenir une compréhension générale du code et permet aux développeurs d'expliquer pourquoi certaines choses ont été développées comme elles l'ont été.

Le but n'est pas d'effectuer une revue de code, mais de comprendre à un niveau élevé le flux, la mise en page et la structure du code qui compose l'application.

### Examens du code de la phase 3.2

Armé d'une bonne compréhension de la structure du code et de la raison pour laquelle certaines choses ont été codées comme elles l'étaient, le testeur peut maintenant examiner le code réel à la recherche de défauts de sécurité.

Les revues de code statiques valident le code par rapport à un ensemble de listes de contrôle, notamment :

- Exigences commerciales en matière de disponibilité, de confidentialité et d'intégrité ;
- Guide OWASP ou Top 10 des listes de contrôle pour les expositions techniques (selon la profondeur de l'examen);
- Problèmes spécifiques liés au langage ou au framework utilisé, tels que le papier Scarlet pour PHP ou les [listes de contrôle Microsoft Secure Coding pour ASP.NET](https://msdn.microsoft.com/en-us/library/ff648269.aspx); et
- Toutes les exigences spécifiques à l'industrie, telles que Sarbanes-Oxley 404, COPPA, ISO/IEC 27002, APRA, HIPAA, directives Visa Merchant ou autres régimes réglementaires.

En termes de retour sur les ressources investies (principalement en temps), les revues de code statiques produisent des retours de bien meilleure qualité que toute autre méthode de revue de sécurité et reposent moins sur les compétences de l'examinateur. Cependant, ils ne sont pas une solution miracle et doivent être examinés avec soin dans le cadre d'un régime de tests à spectre complet.

Pour plus de détails sur les listes de contrôle OWASP, veuillez vous référer à la dernière édition du [OWASP Top 10](https://owasp.org/www-project-top-ten/).

## Phase 4 pendant le déploiement

### Phase 4.1 Test de pénétration des applications

Après avoir testé les exigences, analysé la conception et effectué la révision du code, on peut supposer que tous les problèmes ont été détectés. Espérons que ce soit le cas, mais les tests de pénétration de l'application après son déploiement fournissent une vérification supplémentaire pour s'assurer que rien n'a été manqué.

### Phase 4.2 Test de gestion de la configuration

Le test de pénétration de l'application doit inclure un examen de la façon dont l'infrastructure a été déployée et sécurisée. Il est important de revoir les aspects de configuration, aussi petits soient-ils, pour s'assurer qu'aucun n'est laissé à un paramètre par défaut qui pourrait être vulnérable à l'exploitation.

## Phase 5 pendant la maintenance et les opérations

### Phase 5.1 Mener des revues de gestion opérationnelle

Il doit y avoir un processus en place qui détaille comment le côté opérationnel de l'application et de l'infrastructure est géré.

### Phase 5.2 Effectuer des bilans de santé périodiques

Des contrôles de santé mensuels ou trimestriels doivent être effectués sur l'application et l'infrastructure pour s'assurer qu'aucun nouveau risque de sécurité n'a été introduit et que le niveau de sécurité est toujours intact.

### Phase 5.3 Assurer la vérification des modifications

Une fois que chaque modification a été approuvée et testée dans l'environnement QA et déployée dans l'environnement de production, il est essentiel que la modification soit vérifiée pour s'assurer que le niveau de sécurité n'a pas été affecté par la modification. Cela devrait être intégré dans le processus de gestion du changement.

## Un flux de travail de test SDLC typique

La figure suivante montre un flux de travail de test SDLC typique.

![Flux de travail de test SDLC typique](images/Typical_SDLC_Testing_Workflow.gif)\
 *Figure 3-1 : Flux de travail de test SDLC typique*
