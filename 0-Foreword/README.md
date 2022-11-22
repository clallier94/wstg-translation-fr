# Avant-propos par Eoin Keary

Le problème des logiciels non sécurisés est peut-être le défi technique le plus important de notre époque. L'essor spectaculaire des applications Web permettant les affaires, les réseaux sociaux, etc. n'a fait qu'aggraver les exigences pour établir une approche robuste de l'écriture et de la sécurisation de notre Internet, des applications Web et des données.

À l'Open Web Application Security Project® (OWASP®), nous essayons de faire du monde un endroit où les logiciels non sécurisés sont l'anomalie, pas la norme. Le Guide de test OWASP a un rôle important à jouer dans la résolution de ce grave problème. Il est extrêmement important que notre approche de test des logiciels pour les problèmes de sécurité soit basée sur les principes de l'ingénierie et de la science. Nous avons besoin d'une approche cohérente, reproductible et définie pour tester les applications Web. Un monde sans normes minimales en termes d'ingénierie et de technologie est un monde chaotique.

Il va sans dire que vous ne pouvez pas créer une application sécurisée sans effectuer des tests de sécurité dessus. Les tests font partie d'une approche plus large pour construire un système sécurisé. De nombreuses organisations de développement de logiciels n'incluent pas les tests de sécurité dans leur processus de développement de logiciels standard. Ce qui est encore pire, c'est que de nombreux fournisseurs de sécurité proposent des tests avec différents degrés de qualité et de rigueur.

Les tests de sécurité, en eux-mêmes, ne sont pas une mesure autonome particulièrement bonne du degré de sécurité d'une application, car il existe un nombre infini de façons dont un attaquant pourrait être en mesure de casser une application, et il n'est tout simplement pas possible de toutes les tester. Nous ne pouvons pas nous "pirater en toute sécurité" car nous n'avons qu'un temps limité pour tester et défendre là où un attaquant n'a pas de telles contraintes.

En conjonction avec d'autres projets OWASP tels que le "Code Review Guide", le "Development Guide" et des outils tels que [OWASP ZAP](https://www.zaproxy.org/), c'est un bon début pour créer et maintenir des applications sécurisées. Ce guide de test vous montrera comment vérifier la sécurité de votre application en cours d'exécution. Je recommande fortement d'utiliser ces guides dans le cadre de vos initiatives de sécurité des applications.

## Pourquoi OWASP?

La création d'un guide comme celui-ci est une entreprise colossale, nécessitant l'expertise de centaines de personnes à travers le monde. Il existe de nombreuses façons différentes de tester les failles de sécurité et ce guide reflète le consensus des principaux experts sur la façon d'effectuer ces tests rapidement, avec précision et efficacité. L'OWASP donne aux spécialistes de la sécurité partageant les mêmes idées la possibilité de travailler ensemble et de former une approche de pratique de pointe pour un problème de sécurité.

L'importance d'avoir ce guide disponible de manière entièrement gratuite et ouverte est importante pour la mission de la fondation. Il donne à chacun la possibilité de comprendre les techniques utilisées pour tester les problèmes de sécurité courants. La sécurité ne devrait pas être un art noir ou un secret fermé que seuls quelques-uns peuvent pratiquer. Il doit être ouvert à tous et non exclusif aux praticiens de la sécurité mais aussi à l'assurance qualité, aux développeurs et aux responsables techniques. Le projet de construction de ce guide garde cette expertise entre les mains des personnes qui en ont besoin - vous, moi et toute personne impliquée dans la construction de logiciels.

Ce guide doit faire son chemin entre les mains des développeurs et des testeurs de logiciels. Il n'y a pas assez d'experts en sécurité des applications dans le monde pour faire une brèche significative dans le problème global. La responsabilité initiale de la sécurité des applications doit incomber aux développeurs car ils écrivent le code. Il ne devrait pas être surprenant que les développeurs ne produisent pas de code sécurisé s'ils ne le testent pas ou ne considèrent pas les types de bogues qui introduisent une vulnérabilité.

La mise à jour de ces informations est un aspect essentiel de ce projet de guide. En adoptant l'approche wiki, la communauté OWASP peut faire évoluer et étendre les informations contenues dans ce guide pour suivre le rythme de l'évolution rapide du paysage des menaces de sécurité des applications.

Ce guide est un excellent témoignage de la passion et de l'énergie que nos membres et nos volontaires de projet ont pour ce sujet. Cela aidera certainement à changer le monde une ligne de code à la fois.

## Adaptation et priorisation

Vous devriez adopter ce guide dans votre organisation. Vous devrez peut-être adapter les informations pour qu'elles correspondent aux technologies, aux processus et à la structure organisationnelle de votre organisation.

En général, plusieurs rôles différents au sein des organisations peuvent utiliser ce guide :

- Les développeurs doivent utiliser ce guide pour s'assurer qu'ils produisent un code sécurisé. Ces tests doivent faire partie des procédures normales de test de code et d'unité.
- Les testeurs de logiciels et l'assurance qualité doivent utiliser ce guide pour étendre l'ensemble de cas de test qu'ils appliquent aux applications. La détection précoce de ces vulnérabilités permet d'économiser beaucoup de temps et d'efforts par la suite.
- Les spécialistes de la sécurité doivent utiliser ce guide en combinaison avec d'autres techniques comme un moyen de vérifier qu'aucune faille de sécurité n'a été manquée dans une application.
- Les chefs de projet doivent tenir compte de la raison pour laquelle ce guide existe et du fait que les problèmes de sécurité se manifestent par des bogues dans le code et la conception.

La chose la plus importante à retenir lors de l'exécution des tests de sécurité est de redéfinir continuellement les priorités. Il existe un nombre infini de façons possibles qu'une application puisse échouer, et les organisations ont toujours un temps et des ressources de test limités. Assurez-vous que le temps et les ressources sont dépensés à bon escient. Essayez de vous concentrer sur les failles de sécurité qui représentent un risque réel pour votre entreprise. Essayez de contextualiser le risque en termes d'application et de ses cas d'utilisation.

Ce guide est mieux considéré comme un ensemble de techniques que vous pouvez utiliser pour trouver différents types de failles de sécurité. Mais toutes les techniques n'ont pas la même importance. Essayez d'éviter d'utiliser le guide comme une liste de contrôle, de nouvelles vulnérabilités se manifestent toujours et aucun guide ne peut être une liste exhaustive de "choses à tester", mais plutôt un bon point de départ.

## Le rôle des outils automatisés

Il existe un certain nombre d'entreprises vendant des outils d'analyse et de test de sécurité automatisés. N'oubliez pas les limites de ces outils afin que vous puissiez les utiliser pour ce qu'ils font de mieux. Comme Michael Howard l'a dit lors de la conférence OWASP AppSec de 2006 à Seattle, "Les outils ne sécurisent pas les logiciels ! Ils aident à faire évoluer le processus et à faire appliquer la politique."

Plus important encore, ces outils sont génériques, ce qui signifie qu'ils ne sont pas conçus pour votre code personnalisé, mais pour les applications en général. Cela signifie que même s'ils peuvent trouver des problèmes génériques, ils n'ont pas une connaissance suffisante de votre application pour leur permettre de détecter la plupart des défauts. D'après mon expérience, les problèmes de sécurité les plus graves sont ceux qui ne sont pas génériques, mais profondément liés à votre logique métier et à la conception d'applications personnalisées.

Ces outils peuvent également être très utiles, car ils détectent de nombreux problèmes potentiels. Bien que l'exécution des outils ne prenne pas beaucoup de temps, chacun des problèmes potentiels prend du temps à étudier et à vérifier. Si l'objectif est de trouver et d'éliminer les défauts les plus graves le plus rapidement possible, demandez-vous s'il est préférable de passer votre temps avec des outils automatisés ou avec les techniques décrites dans ce guide. Pourtant, ces outils font certainement partie d'un programme de sécurité des applications bien équilibré. Utilisés à bon escient, ils peuvent prendre en charge vos processus globaux pour produire un code plus sécurisé.

## Appel à l'action

Si vous construisez, concevez ou testez un logiciel, je vous encourage vivement à vous familiariser avec les conseils de test de sécurité contenus dans ce document. C'est une excellente feuille de route pour tester les problèmes les plus courants auxquels les applications sont confrontées aujourd'hui, mais elle n'est pas exhaustive. Si vous trouvez des erreurs, veuillez ajouter une note à la page de discussion ou apporter la modification vous-même. Vous aiderez des milliers d'autres personnes qui utilisent ce guide.

Veuillez envisager de [nous rejoindre](https://owasp.org/membership/) en tant que membre individuel ou corporatif afin que nous puissions continuer à produire des documents comme ce guide de test et tous les autres grands projets de l'OWASP.

Merci à tous les contributeurs passés et futurs de ce guide, votre travail contribuera à rendre les applications plus sûres dans le monde entier.

--Eoin Keary, membre du conseil d'administration de l'OWASP, 19 avril 2013

Open Web Application Security Project et OWASP sont des marques déposées de OWASP Foundation, Inc.
