# Jeu de figurines — des agents qui jouent, et un banc d'essai

> **État : projet en cours.** Les résultats sont remplis au fil des mesures. Une case « à mesurer » n'a pas encore de chiffre.

Un petit jeu coopératif sur grille, aux règles originales, sert de terrain d'essai : cinq joueurs automatiques s'y mesurent sur des parties rejouables à l'identique, et un banc d'essai dit lequel gagne et à quelle vitesse.

## Ce que le projet montre

- Moteur de jeu testé, rejouable à la graine près
- Le modèle propose, le moteur valide : aucun coup interdit ne passe
- Apport mesuré des outils (probabilités, chemins, cases menacées)
- Agent seul contre équipe d'agents, sur les mêmes parties
- Banc d'essai en une commande, traces et relecture de partie

## Résultats mesurés

| Joueur | Parties | Victoires | Coups interdits | Appels par partie | Durée par partie |
|---|---|---|---|---|---|
| Hasard | 200 | à mesurer | | 0 | |
| Règles simples | 200 | à mesurer | | 0 | |
| Modèle seul | | à mesurer | | | |
| Modèle + outils | | à mesurer | | | |
| Équipe | | à mesurer | | | |

- Date de la mesure : à compléter
- Modèle : à compléter
- Nombre de cas : à compléter
- Commande pour refaire la mesure : à compléter

## Installation

Il faut Python 3.12, [uv](https://docs.astral.sh/uv/) et [Ollama](https://ollama.com/).

```
uv sync
copy .env.example .env
```

Puis écrire dans `.env` le nom du modèle utilisé, téléchargé avec `ollama pull`.

## Utilisation

À compléter : la commande principale et un exemple d'entrée et de sortie.

## Comment ça marche

À compléter : un schéma (Mermaid) et cinq lignes d'explication.

## Limites connues

À compléter : ce qui échoue encore, avec un ou deux exemples.

## Coût

0 €. Le projet tourne en local avec des modèles ouverts. Aucun service payant n'est nécessaire.

## Crédits

- **Code, données et mesures** : Charles Burgy.
- **Cours suivi ([COURS.md](COURS.md)) et structure de départ** : module produit par Claude Code (Anthropic), modèle Claude Fable 5.2, créé le 25/11/2026.
- **Sources officielles** : listées à la fin de [COURS.md](COURS.md).

## Licence

[MIT](LICENSE). Projet d'apprentissage, sans but commercial.
