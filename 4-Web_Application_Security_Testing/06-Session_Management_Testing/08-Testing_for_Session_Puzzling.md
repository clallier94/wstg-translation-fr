# Test de Session Puzzling

|ID          |
|------------|
|WSTG-SESS-08|

## Sommaire

La surcharge de variable de session (également connue sous le nom de Session Puzzling) est une vulnérabilité au niveau de l'application qui peut permettre à un attaquant d'effectuer diverses actions malveillantes, y compris, mais sans s'y limiter :

- Contourner les mécanismes efficaces d'application de l'authentification et usurper l'identité des utilisateurs légitimes.
- Élever les privilèges d'un compte utilisateur malveillant, dans un environnement qui serait autrement considéré comme infaillible.
- Ignorez les phases de qualification dans les processus multiphases, même si le processus inclut toutes les restrictions de niveau de code couramment recommandées.
- Manipulez les valeurs côté serveur dans des méthodes indirectes qui ne peuvent pas être prédites ou détectées.
- Exécutez des attaques traditionnelles dans des endroits auparavant inaccessibles, voire considérés comme sûrs.

Cette vulnérabilité se produit lorsqu'une application utilise la même variable de session à plusieurs fins. Un attaquant peut potentiellement accéder aux pages dans un ordre non prévu par les développeurs afin que la variable de session soit définie dans un contexte puis utilisée dans un autre.

Par exemple, un attaquant pourrait utiliser la surcharge des variables de session pour contourner les mécanismes d'application de l'authentification des applications qui appliquent l'authentification en validant l'existence de variables de session contenant des valeurs liées à l'identité, qui sont généralement stockées dans la session après un processus d'authentification réussi. Cela signifie qu'un attaquant accède d'abord à un emplacement de l'application qui définit le contexte de session, puis accède à des emplacements privilégiés qui examinent ce contexte.

Par exemple, un vecteur d'attaque de contournement d'authentification peut être exécuté en accédant à un point d'entrée accessible au public (par exemple, une page de récupération de mot de passe) qui remplit la session avec une variable de session identique, basée sur des valeurs fixes ou sur l'entrée de l'utilisateur.

## Objectifs des tests

- Identifier toutes les variables de session.
- Rompre le flux logique de génération de session.

## Comment tester

### Test de la boîte noire

Cette vulnérabilité peut être détectée et exploitée en énumérant toutes les variables de session utilisées par l'application et dans quel contexte elles sont valides. Ceci est notamment possible en accédant à une séquence de points d'entrée puis en examinant les points de sortie. En cas de test en boîte noire, cette procédure est difficile et nécessite un peu de chance car chaque séquence différente peut conduire à un résultat différent.

#### exemples

Un exemple très simple pourrait être la fonctionnalité de réinitialisation du mot de passe qui, dans le point d'entrée, pourrait demander à l'utilisateur de fournir des informations d'identification telles que le nom d'utilisateur ou l'adresse e-mail. Cette page peut ensuite remplir la session avec ces valeurs d'identification, qui sont reçues directement du côté client, ou obtenues à partir de requêtes ou de calculs basés sur l'entrée reçue. À ce stade, certaines pages de l'application peuvent afficher des données privées basées sur cet objet de session. De cette manière, l'attaquant pourrait contourner le processus d'authentification.

### Test de la boîte grise

Le moyen le plus efficace de détecter ces vulnérabilités consiste à passer en revue le code source.

## Correction

Les variables de session ne doivent être utilisées que dans un seul but cohérent.

## Références

- [Session Puzzling](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/puzzlemall/Session%20Puzzles%20-%20Indirect%20Application%20Attack%20Vectors%20 -%20May%202011%20-%20Whitepaper.pdf)
- [Session Puzzling et Session Race Conditions] (http://sectooladdict.blogspot.com/2011/09/session-puzzling-and-session-race.html)
