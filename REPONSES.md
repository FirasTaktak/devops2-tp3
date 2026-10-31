# TP3 — Réponses

**Binôme :** TAKTAK Firas — REGUIGUI Mohamed

> Répondez sous chaque question. Les mesures (durées, pourcentages) sont celles de **votre** machine.

## Partie 1 — Pyramide des tests

**Q1.** Pour chacun des dossiers `tests/unit`, `tests/component`, `tests/integration` et `tests/e2e`, indiquez : le niveau de la pyramide, l'outil utilisé, et si la stack Docker doit tourner pour l'exécuter.

| Dossier | Niveau | Outil | Stack ? |
|---|---|---|---|
| `tests/unit` | Base | Vitest | Non |
| `tests/component` | Base, juste au-dessus de l'unitaire | Vitest, Testing Library, jsdom | Non |
| `tests/integration` | Milieu | Vitest (environnement node) | Oui, Strapi et PostgreSQL |
| `tests/e2e` | Sommet | Playwright avec Chromium | Oui, frontend, Strapi et PostgreSQL |

**Q2.** Pourquoi la pyramide des tests est-elle large à la base et étroite au sommet ? Donnez au moins deux critères.

Le coût et la vitesse : un test unitaire ne demande rien et tourne en quelques millisecondes, un E2E demande toute la stack.
La précision : un test unitaire pointe la fonction en cause, un E2E dit juste que le parcours est cassé.
La fiabilité aussi : plus il y a de services, plus il y a de raisons d'échouer sans vrai bug.

## Partie 2 — Tests unitaires

**Q3.** Combien de tests sont exécutés et combien échouent ? Recopiez le message d'échec et expliquez ce que signifient `Expected` et `Received`.

17 tests exécutés, 1 échoue, 16 passent.

```
AssertionError: expected '1 h 0' to be '1 h'
Expected: "1 h"
Received: "1 h 0"
```

`Expected` est la valeur voulue par le test. `Received` est ce que la fonction a vraiment renvoyé.

**Q4.** Quelle était la cause du bug dans `formatDuration` ? Expliquez pourquoi la correction fournie le résout.

La fonction affichait toujours les minutes restantes, même quand il n'y en avait pas. Pour 60 min, `rest` vaut 0 et on obtenait `1 h 0`.
La correction teste `rest === 0` : dans ce cas on n'affiche que les heures.

**Q5.** Avant d'ajouter des tests : relevez la couverture (lignes, branches, fonctions) de `lib/cours.ts`. D'après le rapport HTML, quelles lignes ou branches ne sont pas couvertes ?

Lignes 93,33 % (14/15), branches 78,57 % (11/14), fonctions 85,71 % (6/7).

Aucun test ne passe de `query`, donc :
- la ligne 48 n'est jamais exécutée, avec ses deux côtés du `||`
- le cas `needle` non vide du `if` ligne 45 n'est jamais pris
- le callback `(topic) => ...` n'est jamais appelé

**Q6.** Après l'ajout des deux tests fournis : quelle est la nouvelle couverture de `lib/cours.ts` ? Quelles lignes ou branches ces tests exécutent-ils qui ne l'étaient pas ? Reste-t-il quelque chose de non couvert, et pourquoi ?

Lignes 100 %, branches 100 %, fonctions 85,71 %.

Les tests exécutent la ligne 48, les deux côtés du `||` et le cas `needle` non vide.

Il reste le callback `(topic) => normalize(topic).includes(needle)`. Avec « git », le cours d a un titre qui correspond, donc le `||` s'arrête là. Le cours b n'a aucun mot-clé, donc `some` n'appelle rien.

**Q7.** Quelle différence entre couverture de lignes et couverture de branches ? Un code couvert à 100 % est-il forcément sans bug ? Justifiez avec un exemple tiré de ce TP.

La couverture de lignes dit si une ligne a été exécutée. La couverture de branches dit si chaque chemin (`if`, ternaire, `||`) a été pris.

Non. L'ancienne ligne de `formatDuration` était exécutée par le test « 1 h 30 », et le bug sur 60 min était pourtant là. Autre exemple : `cours.ts` est à 100 % de lignes et de branches alors que la recherche par mot-clé n'a jamais tourné.

## Partie 3 — Tests de composant

**Q8.** Que vérifie `CoursCard.test.tsx` qu'un test unitaire de `formatDuration` ne vérifie pas ? Quel outil simule le navigateur, et dans quel fichier est-il configuré ?

Il vérifie ce qui est vraiment affiché : le lien vers `/cours/abc123`, la durée et le niveau dans la page, les mots-clés, et leur absence quand la liste est vide.

Le navigateur est simulé par jsdom, configuré dans `frontend/vitest.config.mts` (`environment: "jsdom"`).

**Q9.** Pourquoi les tests recherchent-ils les éléments par rôle ou par texte (`getByRole`, `getByText`) plutôt que par classe CSS ?

Parce que c'est ce que l'utilisateur voit. Une classe CSS peut changer sans rien casser pour lui, et le test ne doit pas tomber pour ça. Le rôle vérifie aussi un peu l'accessibilité au passage.

**Q10.** Pourquoi n'y a-t-il pas de test de composant pour `app/page.tsx` ? Quel niveau de test le couvre ?

La page rend un composant serveur async qui appelle `connection()` et `getCours()`. Il lui faut Next.js et Strapi, ce que jsdom ne fournit pas. Elle est couverte par les tests E2E (`parcours.spec.ts`).

## Partie 4 — Tests d'intégration

**Q11.** Que vérifient les tests d'intégration qui n'est vérifié par aucun test unitaire ou de composant ?

Que `lib/api.ts` parle vraiment à Strapi : la bonne URL, le format de la réponse, les 6 cours du seed, le tri par séance, et `null` sur un identifiant inconnu. Les autres tests utilisent des données écrites à la main.

**Q12.** Strapi arrêté : quel résultat et quel message obtenez-vous ? Pourquoi les tests unitaires passent-ils toujours ? À quoi sert `--no-deps` ?

4 tests sur 4 échouent en 16,62 s :

```
TypeError: fetch failed
Caused by: Error: getaddrinfo ENOTFOUND strapi
```

Les tests unitaires passent toujours (19 sur 19) car ils n'appellent aucun service : tout est en mémoire.

`--no-deps` empêche Compose de lancer les services de `depends_on`. Sans lui, Strapi aurait redémarré avant les tests.

**Q13.** Un test d'intégration en échec signale-t-il forcément un bug dans le code ? Citez deux autres causes possibles.

Non. Strapi peut être arrêté ou pas encore prêt, et les données du seed peuvent avoir changé. Une mauvaise configuration comme `STRAPI_URL` donne le même résultat.

## Partie 5 — Tests de bout en bout

**Q14.** Quel scénario échoue ? Que montrent la capture d'écran et la trace ? Quel lien avec la question Q3 ? Pourquoi échoue-t-il alors que vous avez déjà corrigé le bug en partie 2 ?

« la fiche du cours d'introduction affiche sa durée » (`parcours.spec.ts:27`). `getByText('1 h', { exact: true })` n'est pas trouvé au bout de 5000 ms.

La capture montre « Durée 1 h 0 ». Dans la trace, la navigation, le clic, l'URL et le titre passent, seule la dernière assertion échoue.

C'est le même bug qu'en Q3. Il revient parce que le conteneur `frontend` tourne sur l'image construite en partie 0, avant la correction. Il fallait `docker compose up -d --build frontend`.

**Q15.** Le même bug est détecté par un test unitaire et par un test E2E. Lequel permet de trouver la cause le plus vite ? Lequel prouve que l'utilisateur est touché ?

Le test unitaire trouve la cause le plus vite : il donne la fonction, la valeur reçue et la ligne (`cours.test.ts:39`). Le test E2E prouve que l'utilisateur est touché : « 1 h 0 » est bien affiché sur la page.

**Q16.** Complétez le tableau avec vos mesures :

| Niveau | Nombre de tests | Durée d'exécution | Besoin de la stack ? |
|---|---|---|---|
| Unitaire + composant | 19 | 7,25 s | Non |
| Intégration | 4 | 839 ms | Oui, Strapi et PostgreSQL |
| E2E | 4 | 4,8 s | Oui, frontend, Strapi et PostgreSQL |

**Q17.** Qu'est-ce qu'un test « flaky » (instable) ? Quels mécanismes de Playwright utilisés ici (attente automatique, assertions `expect(...).toHaveCount`, etc.) réduisent ce risque ?

Un test flaky passe ou échoue d'une fois à l'autre sans que le code change, souvent à cause du timing.

Ici, `click` et `fill` attendent que l'élément soit prêt. `toHaveCount`, `toHaveText`, `toHaveURL` et `toBeVisible` réessaient jusqu'au timeout au lieu de vérifier une seule fois. Il n'y a aucune pause fixe dans les tests, et `retries: 0` ne cache aucune instabilité.

## Partie 6 — SonarQube

**Q18.** Collez votre `sonar-project.properties` complété et expliquez chaque valeur ajoutée. Pourquoi exclut-on `app/**` et `lib/api.ts` du calcul de couverture ?

```properties
sonar.projectKey=tp3-cours
sonar.projectName=TP3 - Cours DevOps
sonar.sources=app,components,lib
sonar.tests=tests
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.exclusions=node_modules/**,.next/**,coverage/**,playwright-report/**,test-results/**
sonar.coverage.exclusions=app/**,lib/api.ts,components/CoursList.tsx,components/DevOpsLoop.tsx,*.config.*
sonar.sourceEncoding=UTF-8
sonar.qualitygate.wait=true
```

- `tp3-cours` : l'identifiant du projet dans SonarQube.
- `app,components,lib` : le code applicatif à analyser.
- `tests` : le code de test, pour qu'il ne compte pas comme du code applicatif.
- `coverage/lcov.info` : le rapport écrit par Vitest. Sonar ne lance pas les tests, il lit ce fichier.

On les exclut parce qu'ils sont testés en intégration et en E2E, et ces tests n'alimentent pas `lcov.info`. Sans l'exclusion, ils compteraient à 0 % alors qu'ils sont testés.

**Q19.** Analyse avec le Quality Gate TP3 : statut du gate et conditions en échec ; nombre de bugs, vulnérabilités, security hotspots et code smells ; couverture ; taux de duplication.

Gate : Failed. Une seule condition en échec : Vulnerabilities vaut 1 alors qu'il faut 0.

| Bugs | Vulnérabilités | Hotspots | Code smells | Couverture | Duplication |
|---|---|---|---|---|---|
| 0 | 1 | 0 | 14 | 97,8 % | 5,6 % |

**Q20.** Expliquez la différence entre bug, vulnérabilité, security hotspot et code smell, avec un exemple tiré de votre rapport quand il en existe.

- Bug : du code qui va mal se comporter à l'exécution. Aucun dans notre rapport.
- Vulnérabilité : une faille à corriger. Exemple : le mot de passe en dur de `lib/api.ts:4`.
- Security hotspot : un code sensible qu'un humain doit relire, pas forcément une faille. Aucun dans notre rapport.
- Code smell : du code qui marche mais qui est pénible à maintenir. Exemple : l'import inutile de `useEffect` dans `CoursList.tsx:3`.

**Q21.** Ouvrez la vulnérabilité : quelle règle est en cause et quel est le risque ? Expliquez la correction fournie : d'où vient maintenant le mot de passe, et par quel chemin arrive-t-il jusqu'au code ? Pourquoi un secret dans le code est-il dangereux, même dans un dépôt privé ?

Règle `typescript:S2068`, « Credentials should not be hard-coded », sévérité Major. Toute personne qui lit le code a le mot de passe.

Il vient maintenant du fichier `.env`, qui n'est pas commité. Le chemin : `.env`, puis `${STRAPI_CLIENT_PASSWORD}` dans `docker-compose.yml`, puis l'environnement du conteneur `frontend`, puis `process.env.STRAPI_CLIENT_PASSWORD` dans `lib/api.ts`.

Même privé, un dépôt est cloné, partagé et lu par les outils de CI. Le secret reste aussi dans l'historique Git après la correction, donc il faut changer l'ancien mot de passe.

**Q22.** Quels code smells la correction fournie de `CoursList.tsx` fait-elle disparaître ? Pour chacun : règle, sévérité, explication, et ce qui, dans le nouveau code, règle le problème.

| Règle | Sévérité | Explication | Ce qui le règle |
|---|---|---|---|
| S1128 (2 fois) | Minor | Imports inutiles de `useEffect` et `sortBySession` | Ils sont retirés |
| S3776 | Critical | Complexité cognitive de 16 pour 15 autorisés dans `describeResults` | La logique est coupée en `describeEmpty` et `describeResults` |
| S6660 (2 fois) | Major | Un `if` seul dans un `else`, lignes 17 et 24 | Plus de `else` imbriqué, des `return` directs et des ternaires |

**Q23.** Où se trouve le code dupliqué ? Comment le supprimer ? (Ne le faites pas, décrivez seulement.)

24 lignes sont dupliquées entre `components/CoursCard.tsx` (lignes 12 à 35) et `app/cours/[documentId]/page.tsx` (lignes 21 à 44) : le bloc `cours-meta` et la liste des mots-clés.

Il faut extraire ce bloc dans un composant, par exemple `CoursMeta`, qui reçoit `cours`, et l'utiliser aux deux endroits.

**Q24.** Qu'est-ce que la dette technique dans SonarQube et comment est-elle exprimée ? Valeur avant et après vos corrections.

C'est le temps estimé pour corriger les code smells. Elle est exprimée en temps, et aussi en ratio par rapport au coût d'écriture du code.

Avant : 63 min (ratio 0,3 %). Après : 45 min (ratio 0,2 %).

**Q25.** Que signifie « New Code » dans SonarQube ? Quelles conditions du gate TP3 portent sur le nouveau code ? Pourquoi SonarQube recommande-t-il de juger le nouveau code plutôt que le code global ?

C'est le code ajouté ou modifié depuis une référence, ici la version précédente (`PREVIOUS_VERSION`).

Quatre conditions : aucune nouvelle issue, couverture du nouveau code ≥ 80 %, duplication ≤ 3 %, 100 % des hotspots relus.

On ne peut pas tout reprendre d'un coup sur un vieux projet. En gardant le nouveau code propre, la dette arrête de grossir. Chez nous, le mot de passe existait dès la première analyse : c'est la condition sur le code global qui l'a attrapé, pas celles du New Code.

**Q26.** Avec `sonar.qualitygate.wait=true`, que fait le scanner quand le Quality Gate est rouge ? Pourquoi est-ce utile dans un pipeline CI ?

Il attend le verdict puis se termine en erreur (code de sortie 3 chez nous, avec `QUALITY GATE STATUS: FAILED`). En CI, le job échoue tout seul et bloque la suite, sans que personne ait à ouvrir le tableau de bord.

## Partie 7 — Synthèse

**Q27.** Proposez l'ordre des étapes d'un pipeline CI pour ce projet (build, tests unitaires, intégration, E2E, analyse Sonar…) et justifiez-le. Quelles étapes doivent bloquer le merge d'une pull request ?

1. Build des images : si ça ne compile pas, inutile d'aller plus loin.
2. Tests unitaires et de composant : les plus rapides, sans stack.
3. Analyse Sonar : juste après, car elle a besoin de `coverage/lcov.info`.
4. Tests d'intégration : on démarre Strapi et PostgreSQL.
5. Tests E2E : les plus lents, ils ont besoin de toute la stack.

On va du moins cher au plus cher pour échouer le plus tôt possible. Toutes ces étapes doivent bloquer le merge : un seul test rouge ou un gate rouge veut dire que le code n'est pas livrable.
