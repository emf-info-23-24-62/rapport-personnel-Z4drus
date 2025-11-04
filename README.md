<h1>Module 323 - Programmer de manière fonctionnelle</h1>

# 👍 RP / `Cheat-sheet` perso 🍒

>[!TIP]
>Pour avancer de manière adéquate dans ces exercices et ces apprentissages avec la programmation fonctionnelle, un bon résumé personnel vous sera non seulement très utile mais indispensable (réaliser les exercices prévus) !!
>
>Ce résumé vous facilitera la réalisation des exercices et vous permettra de progresser plus facilement avec l'utilisation de ces outils javascript de programmation fonctionnelle. Et aussi de détecter lorsque vous n'avez pas compris une méthode (car incapable de facilement l'expliquer).
>
>Et dans le doute 🤔, en farfouillant dans les possibilités offertes, vous y trouverez peut-être un indice sur le comment résoudre tel ou tel besoin.

---

<img src="res/EMF_logo_RVB_Info_long.png" width="25%" style="margin-left:-20px;">

## Table des matières

- **Introduction rapide à la PF**
- **Fonctions fléchées (arrow functions)**
- **map**
- **filter**
- **reduce**
- **Immuabilité**
- **Fonctions pures**
- **Composition** (currying, closure, pipe, compose)
- **Récursion**
- **Annexes** (mémo + liens + export PDF)

---

## Introduction rapide à la PF

La PF (programmation fonctionnelle) en deux mots:

- **Déclaratif**: on dit «quoi faire», pas «comment faire pas à pas».
- **Données immuables**: on ne modifie pas, on crée de nouvelles valeurs.
- **Fonctions pures**: même entrée → même sortie, **sans effets de bord**.
- **Pas de `for`/`while` classiques**: on préfère `map`/`filter`/`reduce`.
- **Composition**: on enchaîne des petites fonctions simples.

Mini-exemples express:

```js
const users = [
  { id: 1, name: 'Noé', age: 19 },
  { id: 2, name: 'Ana', age: 22 },
  { id: 3, name: 'Léo', age: 17 },
];

// map: transformer
const namesUpper = users.map(u => u.name.toUpperCase());
// ['NOÉ', 'ANA', 'LÉO']

// filter: garder une partie
const adults = users.filter(u => u.age >= 18);
// [{ id:1, ... }, { id:2, ... }]

// reduce: «plier» en une valeur
const totalAge = users.reduce((acc, u) => acc + u.age, 0);
// 58
```

Astuce export PDF: il est possible de styler l’impression via `res/pdf-style.css` (Ctrl/Cmd+P → «Enregistrer au format PDF»).

---

## Fonctions fléchées (arrow functions)

Pourquoi les utiliser ?

- **Courtes et lisibles** pour des callbacks (`map`, `filter`, etc.).
- **Pas de nouveau `this`** créé (utile avec des méthodes/timeout).
- Parfaites pour des **fonctions simples** et anonymes.

Exemples concrets:

```js
// 1) De fonction classique → fonction fléchée
function add(a, b) { return a + b; }
const addArrow = (a, b) => { return a + b; };

// 2) Retour implicite (quand c'est une seule expression)
const square = n => n * n;

// 3) Retourner un objet littéral → parenthèses obligatoires
const userPreview = user => ({ id: user.id, name: user.name });

// 4) En callbacks avec map/filter/reduce
const people = [
  { name: 'Mina', score: 10 },
  { name: 'Alex', score: 18 },
  { name: 'Zoé',  score: 15 },
];

const names = people.map(p => p.name);                // ['Mina','Alex','Zoé']
const passed = people.filter(p => p.score >= 15);     // Alex, Zoé
const avg    = people.reduce((a, p, _, arr) => a + p.score / arr.length, 0);
// 14.333...

// 5) this lexical (rapide): pas de nouveau this
const timer = {
  count: 0,
  start() {
    setTimeout(() => {
      this.count += 1; // ici this === timer
    }, 10);
  }
};
```

À retenir:

- Retour implicite pratique, mais cela ne convient pas en cas de plusieurs instructions.
- Pour retourner un **objet**, il faut utiliser des **parenthèses**: `() => ({ ... })`.
- Idéal en **petites briques** à composer.

---

## map

Rappel rapide: `array.map(fn)` applique `fn(element)` à chaque élément et retourne un **nouveau** tableau (même longueur).

Exemples utiles:

```js
// 1) Transformation simple
const nums = [1, 2, 3];
const doubled = nums.map(n => n * 2); // [2, 4, 6]

// 2) Extraction d’un champ
const products = [
  { id: 1, name: 'Stylo', price: 2.5 },
  { id: 2, name: 'Cahier', price: 4.2 },
];
const names = products.map(p => p.name); // ['Stylo', 'Cahier']

// 3) Formatage
const labels = products.map(p => `${p.name} – CHF ${p.price.toFixed(2)}`);
// ['Stylo – CHF 2.50', 'Cahier – CHF 4.20']

// 4) Ajouter un champ dérivé (sans muter l’original)
const withVat = products.map(p => ({
  ...p,
  priceVat: +(p.price * 1.077).toFixed(2)
}));

// 5) Normaliser des textes
const raw = ['  Hello ', 'World  ', '  JS  '];
const clean = raw.map(s => s.trim().toLowerCase()); // ['hello','world','js']

// 6) Mapper vers un autre type (ex: longueurs)
const lengths = clean.map(s => s.length); // [5, 5, 2]

// 7) Mapper avec index (deuxième arg de la callback)
const numbered = clean.map((s, i) => `${i + 1}. ${s}`);

// 8) map + fonctions réutilisables (composition friendly)
const toUpper = s => s.toUpperCase();
const exclaim = s => `${s}!`;
const shout = raw.map(s => exclaim(toUpper(s.trim())));

// 9) map async (attention: cela retourne un tableau de Promises)
const ids = [1, 2, 3];
const fetchUser = async id => ({ id, name: `User ${id}` });
const userPromises = ids.map(fetchUser);
const users = await Promise.all(userPromises); // [{ id:1,... }, ...]
```

Anti-pattern fréquent: utiliser `map` pour ses effets de bord (ex: push, mutation). Si aucune valeur utile n’est retournée, il vaut mieux utiliser `forEach`.

## filter

Rappel rapide: `array.filter(fn)` garde seulement les éléments où `fn(element)` retourne `true`.

Exemples utiles:

```js
// 1) Filtrage numérique simple
const nums = [1, 2, 3, 4, 5, 6];
const even = nums.filter(n => n % 2 === 0); // [2,4,6]

// 2) Filtrer des objets par condition
const users = [
  { id: 1, name: 'Ana', age: 22 },
  { id: 2, name: 'Noé', age: 17 },
  { id: 3, name: 'Lea', age: 19 },
];
const adults = users.filter(u => u.age >= 18); // Ana, Lea

// 3) Prédicats réutilisables
const byMinAge = min => u => u.age >= min;
const canDrink = users.filter(byMinAge(18));

// 4) Recherche texte insensible à la casse
const query = 'an';
const matches = users.filter(u => u.name.toLowerCase().includes(query.toLowerCase()));
// Ana

// 5) Nettoyer une liste (falsy → retirés)
const messy = ['A', '', null, 'B', undefined, 'C'];
const cleaned = messy.filter(Boolean); // ['A','B','C']

// 6) Chaînage filter → map
const productList = [
  { name: 'Pomme', inStock: true, price: 1.2 },
  { name: 'Poire', inStock: false, price: 2.1 },
  { name: 'Kiwi',  inStock: true, price: 1.8 },
];
const labelsInStock = productList
  .filter(p => p.inStock)
  .map(p => `${p.name} – CHF ${p.price}`);
```

Astuce: extraire les prédicats (fonctions qui retournent `true/false`) les rend testables et réutilisables.

## reduce

Rappel rapide: `array.reduce(fn, init)` accumule en une seule valeur. `fn(acc, cur)` doit retourner le nouvel accumulateur.

Exemples utiles:

```js
// 1) Somme
const nums = [2, 5, 7];
const sum = nums.reduce((acc, n) => acc + n, 0); // 14

// 2) Moyenne
const avg = nums.reduce((acc, n, _, arr) => acc + n / arr.length, 0); // 4.666...

// 3) Min / Max
const max = nums.reduce((m, n) => (n > m ? n : m), -Infinity); // 7
const min = nums.reduce((m, n) => (n < m ? n : m), +Infinity); // 2

// 4) Aplatir un niveau
const nested = [[1,2], [3], [4,5]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []); // [1,2,3,4,5]

// 5) Indexer par clé (id → objet)
const users = [
  { id: 'u1', name: 'Ana' },
  { id: 'u2', name: 'Noé' },
];
const byId = users.reduce((acc, u) => { acc[u.id] = u; return acc; }, {});
// { u1: {…}, u2: {…} }

// 6) Compter occurrences
const fruits = ['pomme', 'banane', 'pomme', 'kiwi', 'banane'];
const counts = fruits.reduce((acc, f) => {
  acc[f] = (acc[f] || 0) + 1;
  return acc;
}, {});
// { pomme: 2, banane: 2, kiwi: 1 }

// 7) GroupBy lisible (clé → tableau)
const animals = [
  { name: 'Tigrou', type: 'chat' },
  { name: 'Rex',    type: 'chien' },
  { name: 'Maya',   type: 'chat' },
];
const groupBy = (arr, keyFn) => arr.reduce((acc, item) => {
  const key = keyFn(item);
  (acc[key] ||= []).push(item);
  return acc;
}, {});
const byType = groupBy(animals, a => a.type);
// { chat: [Tigrou,Maya], chien: [Rex] }

// 8) Pipeline chiffrage (TVA + total en une passe)
const cart = [
  { name: 'Stylo',  price: 2.5, qty: 2 },
  { name: 'Cahier', price: 4.2, qty: 1 },
];
const totals = cart.reduce((acc, item) => {
  const line = item.price * item.qty;
  acc.subtotal += line;
  acc.vat = +(acc.subtotal * 0.077).toFixed(2);
  acc.total = +(acc.subtotal + acc.vat).toFixed(2);
  return acc;
}, { subtotal: 0, vat: 0, total: 0 });
```

Conseil: si l’accumulateur devient trop gros/illisible, il est conseillé de découper en petites fonctions ou de composer avec `map`/`filter` avant le `reduce`.

## Immuabilité

Idée simple: on ne **mute** pas une valeur existante; on **crée** une nouvelle valeur.

Exemples rapides:

```js
// Objets
const user = { id: 1, name: 'Ana' };
// ❌ mutation
user.name = 'Anaïs';
// ✅ immuable (nouvel objet)
const user2 = { ...user, name: 'Anaïs' };

// Tableaux
const a = [1, 2];
// ❌ mutation
a.push(3);
// ✅ immuable (nouveau tableau)
const b = [...a, 3];

// Retirer un élément: filter
const without2 = b.filter(n => n !== 2);

// Remplacer un élément: map
const replaced = b.map(n => (n === 3 ? 30 : n));
```

Anti-pattern classique avec `map`:

```js
// ❌ Muter les objets internes
const eleves = [
  { nom: 'Mina', notes: [5, 4] },
  { nom: 'Alex', notes: [3.5, 4.5] },
];
const r1 = eleves.map(e => {
  const avg = e.notes.reduce((a, n, _, arr) => a + n / arr.length, 0);
  e.moyenne = +avg.toFixed(2); // mutation
  return e;
});

// ✅ Retourner de nouveaux objets
const r2 = eleves.map(e => {
  const avg = e.notes.reduce((a, n, _, arr) => a + n / arr.length, 0);
  return { ...e, moyenne: +avg.toFixed(2) };
});
```

Cas un peu plus profond (mise à jour d’un item dans une liste imbriquée):

```js
const state = {
  cart: {
    items: [
      { id: 'p1', qty: 1, price: 2.5 },
      { id: 'p2', qty: 2, price: 4.2 },
    ]
  }
};

// ✅ immuable: recréer chaque niveau
const inc = id => ({ cart: { items } }) => ({
  cart: {
    items: items.map(it => it.id === id ? { ...it, qty: it.qty + 1 } : it)
  }
});

const newState = inc('p1')(state);
```

Note: pour des structures très profondes, `structuredClone(obj)` (si dispo) ou des utilitaires dédiés peuvent aider, mais garder le **principe**: ne pas muter, recréer.

## Fonctions pures

Définition courte: même entrée → même sortie, **pas d’effet de bord**.

Impure → Pure:

```js
// ❌ Impure: lit une variable globale et la modifie
let rate = 0.077;
const addVatImpure = price => price * (1 + rate);
rate = 0.08; // change le résultat sans changer l’input

// ✅ Pure: injecter les dépendances en paramètres (currying possible)
const addVat = vat => price => price * (1 + vat);
const addVat077 = addVat(0.077);

// ❌ Impure: side effect (modifie le tableau reçu)
const addItemImpure = (arr, x) => { arr.push(x); return arr; };

// ✅ Pure: retourner un nouveau tableau
const addItem = (arr, x) => [...arr, x];

// ❌ Impure: dépend de l’heure courante
const greetingImpure = name => `Hello ${name} @ ${new Date().toISOString()}`;

// ✅ Pure: l’horloge est passée en argument (testable)
const greeting = (clock, name) => `Hello ${name} @ ${clock.nowISO()}`;
const fixedClock = { nowISO: () => '2025-11-03T10:00:00.000Z' };
```

Règle pratique: placer les effets (log, réseau, DOM, Date, Math.random) **aux bords** de l’application. Le cœur (les fonctions de transformation) reste pur, donc facile à tester.

## Composition (currying, closure, pipe, compose)

### Currying

Transformer une fonction multi-arguments en **chaîne de fonctions unaires** pour mieux composer.

```js
// add(a, b) → curry → add(a)(b)
const add = a => b => a + b;
const inc = add(1);
inc(41); // 42

// Utilité: passer une fonction unaire à map/filter
const plus10 = add(10);
[1,2,3].map(plus10); // [11,12,13]

// Petit utilitaire de curry (démonstration simple)
const curry2 = fn => a => b => fn(a, b);
const multiply = (a, b) => a * b;
const double = curry2(multiply)(2);
double(6); // 12
```

### Closure

Une fonction «se souvient» des variables de son contexte de création.

```js
// 1) Compteur avec état privé
const makeCounter = () => {
  let count = 0;
  return () => ++count;
};
const c1 = makeCounter();
c1(); // 1
c1(); // 2

// 2) Fabrique de prédicats (réutilisable avec filter)
const minAge = min => user => user.age >= min; // min capturé
const canDrink = minAge(18);
[{age:16},{age:20}].filter(canDrink); // [{age:20}]
```

### pipe / compose

Composer des fonctions pures en chaîne lisible.

```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

// Fonctions simples
const trim = s => s.trim();
const lower = s => s.toLowerCase();
const slugify = s => s.replace(/\s+/g, '-');

// Pipeline lisible (gauche → droite)
const toSlug = pipe(trim, lower, slugify);
toSlug('  Hello World  '); // 'hello-world'
```

Cas pratique: panier → filtrer en stock → prix TTC → total.

```js
const inStock = p => p.inStock;
const map = fn => arr => arr.map(fn);
const filter = fn => arr => arr.filter(fn);
const reduce = (fn, init) => arr => arr.reduce(fn, init);

const addVat = vat => price => +(price * (1 + vat)).toFixed(2);
const priceTtc = vat => p => ({ ...p, price: addVat(vat)(p.price) });
const sum = reduce((a, n) => a + n, 0);

const totalTtc = vat => pipe(
  filter(inStock),
  map(priceTtc(vat)),
  map(p => p.price * p.qty),
  sum,
);

const products = [
  { name: 'Stylo',  price: 2.5, qty: 2, inStock: true },
  { name: 'Cahier', price: 4.2, qty: 1, inStock: false },
  { name: 'Gomme',  price: 1.1, qty: 3, inStock: true },
];

totalTtc(0.077)(products); // ex: 7.82
```

Astuce: `compose` fait la même chose que `pipe` mais de **droite → gauche**. Il est possible d’utiliser celui dont la lecture semble la plus naturelle.

## Récursion

Idée: une fonction s’appelle elle-même jusqu’à une **condition d’arrêt**.

Exemple simple (compte à rebours):

```js
// Itératif
const countdownIter = n => {
  const out = [];
  for (let i = n; i >= 0; i--) out.push(i);
  return out;
};

// Récursif
const countdownRec = n => (n < 0 ? [] : [n, ...countdownRec(n - 1)]);
```

Parcours d’un arbre (somme des tailles):

```js
const tree = {
  name: 'root', size: 2,
  children: [
    { name: 'A', size: 3, children: [] },
    { name: 'B', size: 1, children: [ { name: 'B1', size: 4, children: [] } ] },
  ]
};

const sumSizes = node => node.size + node.children.reduce((a, c) => a + sumSizes(c), 0);
sumSizes(tree); // 10
```

Alternative itérative (éviter les très grandes profondeurs → risque de stack overflow en JS):

```js
const sumSizesIter = root => {
  let total = 0;
  const stack = [root];
  while (stack.length) {
    const node = stack.pop();
    total += node.size;
    stack.push(...node.children);
  }
  return total;
};
```

À retenir:

- Il faut toujours définir une **condition d’arrêt** claire.
- En JS, la **TCO** (optimisation de récursion terminale) n’est pas fiable → pour de grandes profondeurs, il est préférable d’utiliser une version itérative (pile manuelle).

## Annexes (mémo + liens + export PDF)

Mémo express:

- **map**: `(fn) => Array` → transforme chaque élément, garde la longueur.
- **filter**: `(pred) => Array` → garde les éléments où `pred(x)` est true.
- **reduce**: `(reducer, init) => any` → plie en une valeur.
- **pure**: aucune dépendance cachée, pas d’effet de bord.
- **immuable**: créer de nouvelles valeurs (`{...obj}`, `[...arr]`).
- **pipe/compose**: composer des fonctions unaires.

Liens utiles:

- MDN Array.map: [developer.mozilla.org → Array/map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- MDN Array.filter: [developer.mozilla.org → Array/filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- MDN Array.reduce: [developer.mozilla.org → Array/reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

Astuces PDF:

- Placer les styles d’impression dans `res/pdf-style.css` et les activer avant l’export.
- Impression: Cmd/Ctrl+P → «Enregistrer au format PDF».
- Conseils:
  - Police lisible (ex: Inter, 12–13px impression), marges régulières.
  - Utiliser des titres courts, du code bien indenté, et éviter les lignes trop longues.
