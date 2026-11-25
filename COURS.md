# Cours 5 — Jeu de figurines : des agents qui jouent, et un banc d'essai

> **Module de cours produit par Claude Code (Anthropic), modèle Claude Fable 5.2. Créé le 25/11/2026**, à partir des sources officielles listées en fin de document. Le cours donne le chemin ; le code du projet est écrit par Charles Burgy.

| | |
|---|---|
| **Durée estimée** | 4 à 5 semaines, soit 18 à 24 séances de 1 à 2 h |
| **Niveau** | Avancé. C'est le projet de synthèse |
| **Coût** | 0 €. Modèle local ; clé gratuite en option pour aller plus vite |
| **Avant de commencer** | Les projets 1 et 3 sont faits |

## Le projet en une phrase

Un petit jeu coopératif sur grille, dont tu écris les règles, sert de terrain d'essai : des joueurs automatiques y jouent (au hasard, par règles simples, par modèle de langage, avec ou sans outils, seul ou à plusieurs), et un banc d'essai dit lequel gagne, à quelle vitesse.

## Pourquoi ce projet

« Multi-agents » est sur beaucoup de CV. Très peu répondent à la question : est-ce mieux qu'un seul agent, et de combien ? Ici la réponse est un tableau, obtenu sur des parties rejouables à l'identique. Un jeu est un bon terrain : les règles sont nettes, la victoire ne se discute pas, le hasard se fixe par une graine.

## Une règle avant tout : ton jeu à toi

Le dépôt est public. Les règles, les noms et les visuels sont les tiens. On s'inspire d'un genre (escarmouche coopérative sur plateau), pas d'un jeu : aucun texte de règles, aucun nom de personnage ou de faction, aucune image repris d'un jeu du commerce.

## Ce que tu sauras faire à la fin

- Écrire un moteur de jeu testé et rejouable.
- Faire jouer un modèle de langage sous le contrôle du moteur.
- Lui donner des outils et mesurer ce qu'ils apportent.
- Comparer un agent seul à une équipe d'agents.
- Construire un banc d'essai et lire ses résultats sans se raconter d'histoire.

## L'arborescence visée

```
05-jeu-figurines/
├── REGLES.md            tes règles, 2 pages
├── escarmouche/
│   ├── moteur/
│   │   ├── etat.py      plateau, figurines, tour
│   │   ├── actions.py   actions permises, application d'une action
│   │   ├── des.py       hasard à graine
│   │   └── partie.py    déroulement complet
│   ├── joueurs/
│   │   ├── hasard.py
│   │   ├── regles.py
│   │   ├── llm_seul.py
│   │   ├── llm_outils.py
│   │   └── equipe.py
│   ├── outils.py
│   ├── llm.py
│   ├── traces.py
│   └── cli.py
├── tests/
├── banc/
│   ├── banc.py
│   ├── scenarios/
│   └── resultats/
└── app.py               relecture d'une partie (Streamlit)
```

---

## Séance 0 — Mise en place

1. `uv sync`, `.env`, dépôt git local.
2. Reprendre `llm.py`, le journal, le cache et le faux client du projet 3.
3. Reprendre le workflow de tests et `ruff` du projet 4.

---

## Module 1 — Écrire tes règles

**Objectif** : deux pages de règles, assez simples pour être codées en une semaine, assez riches pour qu'il y ait de vrais choix.

**Une base possible, à faire tienne**

- Une grille de 8 × 8 cases, avec des murs et une sortie.
- 3 figurines alliées. Chacune a des points de vie et 3 actions par tour : se déplacer, attaquer, fouiller, ouvrir une porte.
- Des adversaires qui apparaissent à des points fixes et avancent vers la figurine la plus proche, selon une règle automatique.
- Les attaques se jouent aux dés.
- Le bruit attire les adversaires.
- Victoire : les 3 figurines atteignent la sortie avec l'objet à trouver. Défaite : une figurine tombe, ou le nombre de tours est dépassé.

**À faire**

1. Écrire `REGLES.md` avec tes noms et ton univers.
2. Jouer 3 parties sur papier. Le jeu est-il trop facile, trop dur ?
3. Lister les décisions intéressantes : celles où le bon coup n'est pas évident. C'est elles qui départageront les joueurs automatiques.

**Réussi quand** : quelqu'un peut jouer une partie en lisant seulement `REGLES.md`.

**Piège** : trop de règles. Chaque règle en plus est du code, des tests et des coups interdits possibles. Commencer petit, enrichir au module 9.

---

## Module 2 — Le moteur de jeu

**Objectif** : un moteur en Python pur, sans modèle, qui applique les règles et ne laisse rien passer.

**Notions**

- L'état du jeu : tout ce qu'il faut pour reprendre la partie. Le décrire avec Pydantic permet de l'enregistrer en JSON et de le relire.
- Séparer trois fonctions : quelles actions sont permises dans cet état, cette action est-elle permise, quel est l'état après cette action.
- Le hasard à graine : `random.Random(graine)` donne la même suite de dés à chaque fois. Une partie se rejoue à l'identique.
- Le moteur est l'arbitre. Aucun joueur, humain ou modèle, ne modifie l'état lui-même.

**À faire**

1. Écrire `etat.py`, `des.py`, `actions.py`, `partie.py`.
2. Tester avec pytest : déplacement dans un mur refusé, attaque hors de portée refusée, victoire et défaite détectées, même graine = même partie.
3. Écrire un affichage du plateau dans le terminal.
4. Jouer une partie toi-même au clavier.

**Réussi quand** : deux parties lancées avec la même graine et les mêmes coups donnent exactement le même résultat, et les tests passent.

---

## Module 3 — Les joueurs de référence

**Objectif** : savoir ce que valent le hasard et quelques règles simples, avant tout modèle.

**Notions**

- Une interface commune : tout joueur est une fonction qui reçoit l'état et la liste des actions permises, et rend une action. Le moteur ne sait pas qui joue.
- La référence (« baseline ») : le score à battre. Sans elle, un taux de victoire ne veut rien dire.

**À faire**

1. Écrire `joueurs/hasard.py` : une action permise au hasard.
2. Écrire `joueurs/regles.py` : quelques règles de bon sens (attaquer si un adversaire est à portée, sinon avancer vers l'objectif).
3. Lancer 200 parties par joueur, graines 1 à 200. C'est instantané : aucun modèle n'est appelé.
4. Régler la difficulté du jeu pour que le joueur à règles gagne entre 30 % et 60 % des parties. Trop facile ou trop dur, on ne verra aucune différence entre les joueurs.

**Réussi quand** : tu as deux taux de victoire sur 200 parties, et un jeu ni trop facile ni trop dur.

---

## Module 4 — Le joueur par modèle de langage

**Objectif** : un modèle joue, sous le contrôle du moteur.

**Notions**

- Le modèle reçoit l'état sous forme de texte ou de JSON, les règles résumées et la liste des actions permises. Il rend une action en sortie structurée.
- Proposer n'est pas faire : le modèle propose, le moteur valide. Un coup interdit est refusé, expliqué au modèle, et redemandé une fois. Ensuite un coup par défaut est joué.
- La façon de décrire l'état compte autant que le modèle. Un plateau dessiné en caractères, une liste de coordonnées, des phrases : à comparer.

**À faire**

1. Écrire `joueurs/llm_seul.py`.
2. Chronométrer une partie complète. Combien d'appels, combien de minutes ? Ce chiffre fixe la taille de ton banc d'essai.
3. Compter les coups interdits proposés.
4. Essayer deux descriptions de l'état et garder la meilleure, mesure à l'appui.
5. Tester le joueur sans modèle, avec le faux client.

**Réussi quand** : une partie se termine sans intervention, et tu connais le taux de coups interdits.

**Piège** : sans carte graphique dédiée, une partie peut durer plusieurs minutes. Réduire la grille, limiter le nombre de tours, et lancer les séries longues en fin de séance. Le cache ne sert pas ici comme ailleurs : deux états différents donnent deux demandes différentes.

---

## Module 5 — Des outils pour le joueur

**Objectif** : mesurer ce que des outils exacts apportent à la décision.

**À faire**

1. Écrire `outils.py`, des fonctions pures et testées :
   - probabilité de réussite d'une attaque, selon les dés lancés ;
   - plus court chemin entre deux cases, murs compris (parcours en largeur) ;
   - cases que les adversaires peuvent atteindre au prochain tour.
2. Écrire `joueurs/llm_outils.py` avec la boucle d'agent du projet 3 : le modèle peut appeler des outils avant de rendre son action, dans une limite de tours.
3. Comparer au joueur sans outils, sur les mêmes graines.

**Réussi quand** : tu sais dire, chiffres à l'appui, si les outils font gagner plus souvent et ce qu'ils coûtent en durée.

---

## Module 6 — Une équipe d'agents

**Objectif** : répondre honnêtement à « plusieurs agents, est-ce mieux ? ».

**Notions**

- Un agent seul voit tout et décide pour les 3 figurines.
- Une équipe : un agent par figurine, qui ne décide que pour elle, et un coordinateur qui fixe le plan du tour. Ils se parlent par messages courts et structurés.
- Plus d'agents, c'est plus d'appels, plus de durée et plus d'occasions de se contredire. Le gain n'est pas garanti.

**À faire**

1. Écrire `joueurs/equipe.py` : le coordinateur donne une intention à chaque figurine, chaque agent choisit ses actions.
2. Décider ce que chaque agent voit : tout le plateau, ou seulement ses alentours ?
3. Comparer à l'agent seul, sur les mêmes graines.
4. Lire trois parties perdues par l'équipe. L'erreur vient-elle du plan, de l'exécution, ou d'un malentendu entre agents ?

**Réussi quand** : ta conclusion tient en deux phrases et s'appuie sur le tableau du module 7, quelle qu'elle soit.

---

## Module 7 — Le banc d'essai

**Objectif** : une commande, un tableau.

**À faire**

1. Écrire `banc/banc.py` : il joue N parties par joueur, sur les mêmes graines, et écrit un fichier de résultats daté.
2. Choisir N selon la durée mesurée au module 4. Ordre de grandeur : 200 parties pour les joueurs sans modèle ; pour les joueurs à modèle, ce que ta machine joue en une à deux heures (20 à 30 parties sans carte graphique dédiée, davantage avec).
3. Pouvoir reprendre un banc interrompu sans rejouer les parties déjà faites.
4. Sortir le tableau :

| Joueur | Parties | Victoires | Tours (moyenne) | Coups interdits | Appels par partie | Durée par partie |
|---|---|---|---|---|---|---|
| Hasard | 200 | | | | 0 | |
| Règles simples | 200 | | | | 0 | |
| Modèle seul | | | | | | |
| Modèle + outils | | | | | | |
| Équipe | | | | | | |

5. Écrire sous le tableau ce que l'on peut conclure et ce que l'on ne peut pas conclure avec 20 parties. Entre 11 et 13 victoires sur 20, l'écart peut venir du hasard.

**Avec une clé gratuite (option)** : les mêmes parties vont plus vite sur un modèle en ligne, dans la limite du quota. Ajouter une ligne au tableau, avec le nom du fournisseur et du modèle.

**Réussi quand** : `uv run python -m banc.banc` refait le tableau sans autre manipulation.

---

## Module 8 — Traces et relecture

**Objectif** : comprendre pourquoi une partie a été perdue.

**À faire**

1. Écrire `traces.py` : une ligne JSON par décision (tour, figurine, état résumé, action proposée, acceptée ou refusée, outils appelés, durée, tokens).
2. Écrire `app.py` avec Streamlit : choisir une partie, avancer tour par tour, voir le plateau et la décision du joueur.
3. Faire une capture animée d'une partie pour le README.

**Réussi quand** : tu peux montrer en 30 secondes une partie rejouée avec, à chaque tour, ce que l'agent a décidé.

---

## Module 9 — Finition

1. Les tests automatiques sur GitHub et `ruff`, repris du projet 4.
2. Le schéma en Mermaid : moteur, joueurs, banc, traces.
3. Le README complet, avec la conclusion sur l'équipe d'agents.
4. Si le temps le permet : un deuxième scénario plus difficile, et le tableau refait.

## Le livrable

- [ ] `REGLES.md`, écrit par toi
- [ ] Le moteur, testé et rejouable à la graine près
- [ ] Les cinq joueurs derrière la même interface
- [ ] Le banc d'essai et son tableau
- [ ] La relecture de partie et une capture animée
- [ ] Une conclusion chiffrée et prudente sur les outils et sur l'équipe

## Sources officielles

- Appel d'outils avec Ollama : https://docs.ollama.com/capabilities/tool-calling
- Sorties structurées avec Ollama : https://docs.ollama.com/capabilities/structured-outputs
- Module `random` de Python (hasard à graine) : https://docs.python.org/fr/3/library/random.html
- Pydantic : https://pydantic.dev/docs/validation/latest/get-started/
- pytest : https://docs.pytest.org/en/stable/
- Streamlit : https://docs.streamlit.io/
- GitHub Actions : https://docs.github.com/fr/actions
- Ruff : https://docs.astral.sh/ruff/
