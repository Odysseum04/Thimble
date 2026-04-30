# Thimble Event — Dé à Coudre

> **Auteur :** Clément Raes — [Odysseum04](https://github.com/odysseum04)

> **Assistance IA :** Script généré avec l'assistance de [Claude AI](https://claude.ai) (Anthropic)

> **Compatibilité :** Skript 2.9+ | Minecraft 1.21.x | Aucun addon requis

---

## Vue d'ensemble

Le **Dé à Coudre** (*Thimble*) est un mini-jeu multijoueur dans lequel les participants s'élancent à tour de rôle depuis une plateforme surélevée vers un bassin d'eau situé en contrebas. Le but est d'atterrir précisément sur une éponge mouillée (case valide). Chaque atterrissage réussi comble la case touchée — le bassin rétrécit progressivement. Un joueur qui rate le bassin et subit des dégâts de chute est définitivement éliminé. Le dernier survivant remporte la partie.

---

## Prérequis

| Élément | Valeur requise |
|---|---|
| Version Minecraft | 1.21.x |
| Plugin Skript | 2.9 ou supérieur |
| Addons requis | Aucun |
| Monde par défaut | `world` (configurable) |
| Permission admin | `events.admin` |

---

## Installation

1. Placer `Thimble-Event.sk` dans `plugins/Skript/scripts/`.
2. Recharger avec `/skript reload Thimble-Event` ou `/skript reload all`.
3. Se rendre dans le monde configuré et exécuter `/te setspawn` depuis la plateforme de départ.
4. Construire l'arène (voir section **Construction de l'arène**).

---

## Options configurables

Les options se trouvent en haut du fichier `.sk`.

| Option | Valeur par défaut | Description |
|---|---|---|
| `prefix` | `[SERVER-NAME Events]` | Préfixe des messages en jeu |
| `perm-admin` | `events.admin` | Permission requise pour `/te` |
| `world-name` | `world` | Nom du monde dédié au mini-jeu |

---

## Commandes

Toutes les commandes requièrent la permission `events.admin` et doivent être exécutées depuis le monde configuré.

### `/te setspawn`
Définit la position actuelle comme **plateforme de départ**. À exécuter avant tout lancement.

### `/te start`
Lance une nouvelle partie :
1. Vérifie qu'aucune partie n'est déjà en cours.
2. Vérifie qu'une plateforme a été définie.
3. Vérifie la présence d'au moins **2 joueurs** dans le monde configuré.
4. Remplace tous les blocs de **diorite** dans un rayon de 100 blocs par de l'**eau**.
5. Lance un **compte à rebours de 5 secondes**.
6. Enregistre les joueurs et démarre le premier tour.

> ⚠️ Le remplacement de blocs en rayon 100 peut générer une charge serveur temporaire.

### `/te stop`
Arrête immédiatement la partie. Tous les joueurs du monde configuré passent en mode **Aventure**.

### `/te info`
Affiche le statut, nombre de joueurs, file d'attente et état de la plateforme.

---

## Mécaniques de jeu

### Tour par tour
Un seul joueur est **actif** (mode Survie) à la fois. Les autres attendent en mode Spectateur.

### La vague
Les joueurs jouent par cycles. Quand tous ont sauté une fois, une nouvelle vague recommence avec les joueurs restants.

### Atterrissage réussi — Éponge mouillée
- Un bloc de **diorite** est placé au-dessus → la case est comblée, le bassin rétrécit.
- Le joueur est renvoyé en **attente** (non éliminé).
- Le joueur suivant est activé.

### Raté — Dégâts de chute
- Les dégâts sont annulés (pas de mort réelle).
- Le joueur est **éliminé définitivement**.
- Broadcast avec le nombre de joueurs restants.

### Faim
Gelée pour le joueur actif — tout changement est annulé.

### Déconnexion

| Situation | Comportement |
|---|---|
| Joueur **actif** se déconnecte | Éliminé, broadcast, tour suivant |
| Joueur en **attente** se déconnecte | Retiré silencieusement |

### Victoire
Quand il ne reste qu'un seul joueur — titre, broadcast et son de victoire.

---

## Construction de l'arène

```
        [Plateforme de départ]   ← /te setspawn ici
               │
               │  (10–20 blocs de hauteur recommandés)
               │
    ┌──────────▼──────────┐
    │  ~ ~ ~ ~ ~ ~ ~ ~ ~  │  ← Eau (remplace la diorite au /te start)
    │  ~ [S] ~ [S] ~ [S]  │  ← [S] = Éponges mouillées (zones valides)
    │  ~ ~ ~ ~ ~ ~ ~ ~ ~  │
    └─────────────────────┘
```

| Bloc | Rôle |
|---|---|
| **Diorite** | Remplisseur initial du bassin → remplacé par de l'eau au lancement |
| **Eau** | Bassin de réception |
| **Éponge mouillée** | Zone d'atterrissage valide → comblée après chaque saut réussi |

> 💡 Poser les éponges au **fond du bassin** sur un bloc solide. L'eau ne doit pas être trop profonde pour que `on walk on wet sponge` se déclenche.

---
## Variables internes

| Variable | Type | Description |
|---|---|---|
| `{te.spawn}` | Location | Position de la plateforme de départ |
| `{te.running}` | Boolean | `true` si une partie est en cours |
| `{te.joueurs::*}` | Liste | Joueurs encore en jeu |
| `{te.vague::*}` | Liste | File d'attente du cycle courant |
| `{te.%uuid%.playing}` | Boolean | `true` si ce joueur est le joueur actif |

> Indexées par **UUID** — pas de conflit en cas de changement de pseudo.

---

## Fonctions internes

| Fonction | Description |
|---|---|
| `te_playerWorldName(p)` | Retourne le nom du monde du joueur |
| `te_isInConfiguredWorld(p)` | Vérifie si le joueur est dans le monde configuré |
| `te_setWaiting(p)` | Place le joueur en attente (spectateur) |
| `te_setSpectating(p)` | Élimine le joueur (spectateur permanent) |
| `te_setPlaying(p)` | Active le joueur (survie), le téléporte, le retire de la vague |
| `te_cleanLists()` | Nettoie les entrées invalides (joueurs déconnectés) |
| `te_nextPlayer()` | Sélectionne le prochain sauteur ou termine la partie |
| `te_endGame()` | Annonce le vainqueur et nettoie les variables |
| `te_forceStop(stopper)` | Arrêt forcé avec broadcast |
| `te_countPlayersInWorld()` | Compte les joueurs dans le monde configuré |
| `te_registerPlayers()` | Enregistre les joueurs du monde dans les listes de jeu |
| `te_countdown()` | Compte à rebours 5s avec titres et sons |

---

## Événements écoutés

| Événement | Condition | Action |
|---|---|---|
| `on damage` | Joueur actif | Annulation dégâts + élimination |
| `on walk on wet sponge` | Joueur actif | Comblage case + fin de tour |
| `on food level change` | Joueur actif | Annulation du changement |
| `on quit` | Partie en cours | Élimination ou retrait silencieux |

---

## Flux d'une partie

```
/te start
    │
    ├─ Vérifications (en cours ? spawn ? ≥ 2 joueurs ?)
    ├─ Diorite → Eau (rayon 100 blocs)
    ├─ te_countdown() — compte à rebours 5s
    ├─ te_registerPlayers() — enregistrement + te_setWaiting(tous)
    ├─ broadcast démarrage
    └─ te_nextPlayer()
            │
            ├─ 1 joueur restant ? → te_endGame() ── FIN
            ├─ Vague vide ? → rechargement
            └─ te_setPlaying(dernier de la file)
                    │
                    ├─ Marche éponge mouillée
                    │       └─ Comblage + te_setWaiting() + te_nextPlayer()
                    └─ Reçoit dégâts
                            └─ te_setSpectating() + te_nextPlayer()
```

---

## Changelog

| Version | Description |
|---|---|
| 2.3 | Ajout de `te_playerWorldName()` et `te_isInConfiguredWorld()` — correction finale de `There's no location in a command event` |
| 2.2 | Correction `%{@prefix}%` → `{@prefix}` (parse-time vs runtime) |
| 2.2 | Extraction des boucles par monde dans des fonctions dédiées |
| 2.1 | Correction `<SERVER-NAME>` → `SERVER-NAME` (caractères `<>` réservés) |
| 2.1 | Simplification de `play sound` (suppression volume/pitch inline) |
| 2.0 | Réécriture complète pour Skript 2.9+ / Minecraft 1.21.x |
| 1.0 | Version initiale — Skript 1.12.2 + SkQuery / TuSKe / skRayFall / Skellett |

---

## Crédits

| Rôle | Personne |
|---|---|
| Auteur & conception | Clément Raes — [Odysseum04](https://github.com/odysseum04) |
| Assistance à la génération | [Claude AI](https://claude.ai) (Anthropic) |
