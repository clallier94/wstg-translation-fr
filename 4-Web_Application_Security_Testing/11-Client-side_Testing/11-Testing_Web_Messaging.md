# Tester la messagerie Web

|ID          |
|------------|
|WSTG-CLNT-11|

## Sommaire

La messagerie Web (également connue sous le nom de [messagerie croisée](https://html.spec.whatwg.org/multipage/web-messaging.html#web-messaging)) permet aux applications exécutées sur différents domaines de communiquer de manière sécurisée. Avant l'introduction de la messagerie Web, la communication d'origines différentes (entre les iframes, les onglets et les fenêtres) était restreinte par la même politique d'origine et imposée par le navigateur. Les développeurs ont utilisé plusieurs hacks pour accomplir ces tâches, et la plupart d'entre eux étaient principalement non sécurisés.

Cette restriction dans le navigateur est en place pour empêcher un site Web malveillant de lire des données confidentielles à partir d'autres iframes, onglets, etc. ; cependant, il existe des cas légitimes où deux sites Web de confiance doivent échanger des données entre eux. Pour répondre à ce besoin, la messagerie interdocuments a été introduite dans le projet de spécification [WHATWG HTML5](https://html.spec.whatwg.org/multipage/) et a été implémentée dans tous les principaux navigateurs. Il permet des communications sécurisées entre plusieurs origines à travers des iframes, des onglets et des fenêtres.

L'API de messagerie a introduit la [méthode `postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage), avec laquelle des messages en texte brut peuvent être envoyés origine. Il se compose de deux paramètres : message et domaine.

Il existe des problèmes de sécurité lors de l'utilisation de `*` comme domaine dont nous parlerons ci-dessous. Pour recevoir des messages, le site Web destinataire doit ajouter un nouveau gestionnaire d'événements, qui possède les attributs suivants :

- Données, le contenu du message entrant ;
- Origine du document expéditeur ; et
- Source, la fenêtre source.

Voici un exemple de l'API de messagerie utilisée. Pour envoyer un message :

```js
iframe1.contentWindow.postMessage("Hello world","http://www.exemple.com");
```

Pour recevoir un message :

```js
window.addEventListener("message", handler, true);
function handler(event) {
    if(event.origin === 'chat.exemple.com') {
        /* process message (event.data) */
    } else {
        /* ignore messages from untrusted domains */
    }
}
```

### Sécurité d'origine

L'origine est composée d'un schéma, d'un nom d'hôte et d'un port. Il identifie de manière unique le domaine qui envoie ou reçoit le message et n'inclut pas le chemin ou la partie fragment de l'URL. Par exemple, `https://exemple.com` sera considéré comme différent de `http://exemple.com` car le schéma du premier est `https`, tandis que le second est `http`. Cela s'applique également aux serveurs Web exécutés dans le même domaine mais sur des ports différents.

## Objectifs des tests

- Évaluer la sécurité de l'origine du message.
- Validez qu'il utilise des méthodes sûres et validez son entrée.

## Comment tester

### Examiner la sécurité de l'origine

Les testeurs doivent vérifier si le code de l'application filtre et traite les messages provenant de domaines de confiance. Dans le domaine d'envoi, assurez-vous également que le domaine de réception est explicitement indiqué et que `*` n'est pas utilisé comme deuxième argument de `postMessage()`. Cette pratique pourrait introduire des problèmes de sécurité et pourrait conduire, dans le cas d'une redirection ou si l'origine change par d'autres moyens, à ce que le site Web envoie des données à des hôtes inconnus, et donc à des fuites de données confidentielles vers des serveurs malveillants.

Si le site Web ne parvient pas à ajouter des contrôles de sécurité pour restreindre les domaines ou les origines autorisés à envoyer des messages à un site Web, il est susceptible d'introduire un risque de sécurité. Les testeurs doivent examiner le code des écouteurs d'événements de message et obtenir la fonction de rappel de la méthode "addEventListener" pour une analyse plus approfondie. Les domaines doivent toujours être vérifiés avant la manipulation des données.

### Examiner la validation des entrées

Bien que le site Web n'accepte théoriquement que les messages provenant de domaines de confiance, les données doivent toujours être traitées comme des données non fiables de source externe et traitées avec les contrôles de sécurité appropriés. Les testeurs doivent analyser le code et rechercher des méthodes non sécurisées, en particulier lorsque les données sont évaluées via `eval()` ou insérées dans le DOM via la propriété `innerHTML`, ce qui peut créer des vulnérabilités XSS basées sur DOM.

### Analyse de code statique

Le code JavaScript doit être analysé pour déterminer comment la messagerie Web est implémentée. En particulier, les testeurs doivent s'intéresser à la manière dont le site Web limite les messages provenant de domaines non approuvés et à la manière dont les données sont gérées même pour les domaines de confiance.

Dans cet exemple, un accès est nécessaire pour chaque sous-domaine (www, chat, forums, ...) du domaine owasp.org. Le code essaie d'accepter n'importe quel domaine avec `.owasp.org` :

```js
window.addEventListener("message", callback, true);

function callback(e) {
    if(e.origin.indexOf(".owasp.org")!=-1) {
        /* process message (e.data) */
    }
}
```

L'intention est d'autoriser des sous-domaines tels que :

- `www.owasp.org`
- `chat.owasp.org`
- `forums.owasp.org`

Malheureusement, cela introduit des vulnérabilités. Un attaquant peut facilement contourner le filtre car un domaine tel que `www.owasp.org.attacker.com` correspondra.

Voici un exemple de code qui n'a pas de vérification d'origine. Ceci est très peu sûr, car il acceptera les entrées de n'importe quel domaine :

```js
window.addEventListener("message", callback, true);

function callback(e) {
        /* process message (e.data) */
}
```

Voici un exemple avec des vulnérabilités de validation d'entrée pouvant conduire à une attaque XSS :

```js
window.addEventListener("message", callback, true);

function callback(e) {
        if(e.origin === "trusted.domain.com") {
            element.innerHTML= e.data;
        }
}
```

Une approche plus sûre consisterait à utiliser la propriété `innerText` au lieu de `innerHTML`.

Pour plus de ressources OWASP concernant la messagerie Web, voir [OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
