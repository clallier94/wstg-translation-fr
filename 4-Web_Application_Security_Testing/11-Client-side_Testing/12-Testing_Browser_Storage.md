# Test du stockage du navigateur

|ID |
|------------|
|WSTG-CLNT-12|

## Sommaire

Les navigateurs fournissent les mécanismes de stockage côté client suivants permettant aux développeurs de stocker et de récupérer des données :

- Stockage local
- Stockage des sessions
- IndexedDB
- Web SQL (obsolète)
- Biscuits

Ces mécanismes de stockage peuvent être visualisés et modifiés à l'aide des outils de développement du navigateur, tels que [Google Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/storage/localstorage) ou [Firefox's Storage Inspector](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector).

Remarque : bien que le cache soit également une forme de stockage, il est couvert dans une [section séparée](../04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md) couvrant ses propres particularités et préoccupations.

## Objectifs des tests

- Déterminez si le site Web stocke des données sensibles dans le stockage côté client.
- La gestion du code des objets de stockage doit être examinée pour les possibilités d'attaques par injection, telles que l'utilisation d'entrées non validées ou de bibliothèques vulnérables.

## Comment tester

### Stockage local

`window.localStorage` est une propriété globale qui implémente [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) et fournit une clé-valeur **persistante** stockage dans le navigateur.

Les clés et les valeurs ne peuvent être que des chaînes, donc toutes les valeurs non-chaînes doivent d'abord être converties en chaînes avant de les stocker, généralement via [JSON.stringify](https://developer.mozilla.org/en-US/docs /Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Les entrées dans `localStorage` persistent même lorsque la fenêtre du navigateur se ferme, à l'exception des fenêtres en mode privé/incognito.

La capacité de stockage maximale de `localStorage` varie selon les navigateurs.

#### Lister toutes les entrées de valeur-clé

```javascript
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### Stockage des sessions

`window.sessionStorage` est une propriété globale qui implémente [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) et fournit une clé-valeur **éphémère** stockage dans le navigateur.

Les clés et les valeurs ne peuvent être que des chaînes, donc toutes les valeurs non-chaînes doivent d'abord être converties en chaînes avant de les stocker, généralement via [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Les entrées de `sessionStorage` sont éphémères car elles sont effacées lorsque l'onglet/la fenêtre du navigateur est fermé.

La capacité de stockage maximale de `sessionStorage` varie selon les navigateurs.

#### Lister toutes les entrées de valeur-clé

```javascript
for (let i = 0; i < sessionStorage.length; i++) {
  const key = sessionStorage.key(i);
  const value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### IndexedDB

IndexedDB est une base de données transactionnelle orientée objet destinée aux données structurées. Une base de données IndexedDB peut avoir plusieurs magasins d'objets et chaque magasin d'objets peut avoir plusieurs objets.

Contrairement au stockage local et au stockage de session, IndexedDB peut stocker plus que de simples chaînes. Tous les objets pris en charge par [l'algorithme de clonage structuré](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) peuvent être stockés dans IndexedDB.

[CryptoKeys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey) est un exemple d'objet JavaScript complexe pouvant être stocké dans IndexedDB, mais pas dans Local/Session Storage.

Recommandation du W3C sur [Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/) [recommande](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage) qui Clés de chiffrement qui doivent être conservées dans le navigateur, pour être stockées dans IndexedDB. Lors du test d'une page Web, recherchez toutes les clés de chiffrement dans IndexedDB et vérifiez si elles sont définies sur "extractable : true" alors qu'elles auraient dû être définies sur "extractable : false" (c'est-à-dire que la clé privée sous-jacente n'est jamais exposée pendant les opérations cryptographiques .)

#### Imprimer tout le contenu de IndexedDB

```javascript
const dumpIndexedDB = dbName => {
  const DB_VERSION = 1;
  const req = indexedDB.open(dbName, DB_VERSION);
  req.onsuccess = function() {
    const db = req.result;
    const objectStoreNames = db.objectStoreNames || [];

    console.log(`[*] Database: ${dbName}`);

    Array.from(objectStoreNames).forEach(storeName => {
      const txn = db.transaction(storeName, 'readonly');
      const objectStore = txn.objectStore(storeName);

      console.log(`\t[+] ObjectStore: ${storeName}`);

      // Print all entries in objectStore with name `storeName`
      objectStore.getAll().onsuccess = event => {
        const items = event.target.result || [];
        items.forEach(item => console.log(`\t\t[-] `, item));
      };
    });
  };
};

indexedDB.databases().then(dbs => dbs.forEach(db => dumpIndexedDB(db.name)));
```

### SQL Web

Web SQL est obsolète depuis le 18 novembre 2010 et il est recommandé aux développeurs Web de ne pas l'utiliser.

### Cookies

Les cookies sont un mécanisme de stockage clé-valeur qui est principalement utilisé pour la gestion de session, mais les développeurs Web peuvent toujours l'utiliser pour stocker des données de chaîne arbitraires.

Les cookies sont traités en détail dans le scénario [test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

#### Lister tous les cookies

```javascript
console.log(window.document.cookie);
```

### Objet de fenêtre globale

Parfois, les développeurs Web initialisent et maintiennent un état global qui n'est disponible que pendant la durée d'exécution de la page en attribuant des attributs personnalisés à l'objet global "window". Par exemple:

```javascript
window.MY_STATE = {
  counter: 0,
  flag: false,
};
```

Toutes les données attachées à l'objet `window` seront perdues lorsque la page est actualisée ou fermée.

#### Lister toutes les entrées de l'objet fenêtre

```javascript
(() => {
  // create an iframe and append to body to load a clean window object
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  // get the current list of properties on window
  const currentWindow = Object.getOwnPropertyNames(window);

  // filter the list against the properties that exist in the clean window
  const results = currentWindow.filter(
    prop => !iframe.contentWindow.hasOwnProperty(prop)
  );

  // remove iframe
  document.body.removeChild(iframe);

  // log key-value entries that are different
  results.forEach(key => console.log(`${key}: ${window[key]}`));
})();
```

_(Version modifiée de cet [extrait](https://stackoverflow.com/a/17246535/3099132))_

### Chaîne d'attaque

Suite à l'identification de l'un des vecteurs d'attaque ci-dessus, une chaîne d'attaque peut être formée avec différents types d'attaques côté client, telles que les attaques [XSS basées sur DOM](01-Testing_for_DOM-based_Cross_Site_Scripting.md).

## Correction

Les applications doivent stocker les données sensibles côté serveur, et non côté client, de manière sécurisée, conformément aux meilleures pratiques.

## Références

- [Stockage local](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Stockage de session](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [API Web Crypto : Stockage de clés](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)
- [WebSQL](https://www.w3.org/TR/webdatabase/)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

Pour plus de ressources OWASP sur l'API de stockage Web HTML5, consultez la [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#html5-web-storage-api).
