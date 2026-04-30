# Thimble Event — Dé à Coudre

> **Auteur :** Clément Raes — [Odysseum04](https://github.com/odysseum04)

> **Assistance IA :** Script généré avec l'assistance de [Claude AI](https://claude.ai) (Anthropic)

> **Compatibilité :** Skript 2.9+ | Minecraft 1.21.x | Aucun addon requis

---

## Vue d'ensemble

Le **Dé à Coudre** (*Thimble*) est un mini-jeu multijoueur dans lequel les participants s'élancent à tour de rôle depuis une plateforme surélevée vers un bassin d'eau situé en contrebas. Le but est d'atterrir précisément sur une éponge mouillée (case valide). Chaque atterrissage réussi comble la case touchée — le bassin rétrécit progressivement au fil des tours. Un joueur qui rate le bassin et subit des dégâts de chute est définitivement éliminé. Le dernier survivant remporte la partie.

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

```
/te setspawn
```

### `/te start`

Lance une nouvelle partie dans l'ordre suivant :

1. Vérifie qu'aucune partie n'est déjà en cours.
2. Vérifie qu'une plateforme a été définie via `/te setspawn`.
3. Vérifie la présence d'au moins **2 joueurs** dans le monde configuré.
4. Remplace tous les blocs de **diorite** dans un rayon de 100 blocs par de l'**eau**.
5. Lance un **compte à rebours de 5 secondes** avec titres et sons.
6. Enregistre les joueurs et démarre le premier tour.

```
/te start
```

> ⚠️ Le remplacement de blocs en rayon 100 peut générer une charge serveur temporaire. Réduire le rayon dans le code si nécessaire.

### `/te stop`

Arrête immédiatement la partie. Tous les joueurs du monde configuré passent en mode **Aventure**.

```
/te stop
```

### `/te info`

Affiche l'état actuel du jeu : statut, nombre de joueurs, file d'attente, plateforme définie ou non.

```
/te info
```

---

## Mécaniques de jeu

### Tour par tour

Un seul joueur est **actif** (mode Survie) à la fois. Les autres spectent depuis la plateforme en mode Spectateur.

### La vague

Les joueurs jouent par **cycles** (vagues). Chaque joueur saute une fois par cycle. Quand tous ont sauté, une nouvelle vague commence avec les joueurs restants.

### Atterrissage réussi — Éponge mouillée

Lorsqu'un joueur actif marche sur une **éponge mouillée** :
- Un bloc de **diorite** est placé dessus → la case est **comblée**, le bassin rétrécit.
- Le joueur est renvoyé en **attente** (non éliminé).
- Le joueur suivant dans la file est activé.

### Raté — Dégâts de chute

Lorsqu'un joueur actif **reçoit des dégâts** :
- Les dégâts sont annulés (pas de mort réelle).
- Le joueur est **éliminé définitivement** → mode Spectateur.
- Broadcast avec le nombre de joueurs restants.
- Le joueur suivant est activé.

### Faim

La barre de faim est **gelée** pour le joueur actif — tout changement est annulé.

### Déconnexion

| Situation | Comportement |
|---|---|
| Joueur **actif** se déconnecte | Éliminé, broadcast, tour suivant |
| Joueur en **attente** se déconnecte | Retiré silencieusement des listes |

### Victoire

Quand il ne reste **qu'un seul joueur**, il est déclaré vainqueur — titre, broadcast et son de victoire pour tous les joueurs du monde.

---

## Construction de l'arène

### Structure recommandée

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

### Blocs utilisés

| Bloc | Rôle |
|---|---|
| **Diorite** | Remplisseur initial du bassin → remplacé par de l'eau au lancement |
| **Eau** | Bassin de réception |
| **Éponge mouillée** | Zone d'atterrissage valide → comblée après chaque saut réussi |

> 💡 Poser les éponges mouillées **au fond du bassin** sur un bloc solide, avec une couche d'eau par-dessus. L'eau ne doit pas être trop profonde pour que `on walk on wet sponge` se déclenche quand le joueur touche le fond.

---

## Bugs corrigés & notes techniques

### Merci de contacter "odysseum04" via discord.
---

## Variables internes

| Variable | Type | Description |
|---|---|---|
| `{te.spawn}` | Location | Position de la plateforme de départ |
| `{te.running}` | Boolean | `true` si une partie est en cours |
| `{te.joueurs::*}` | Liste | Joueurs encore en jeu (non éliminés) |
| `{te.vague::*}` | Liste | File d'attente du cycle courant |
| `{te.%uuid%.playing}` | Boolean | `true` si ce joueur est le joueur actif |

> Les joueurs sont indexés par **UUID** — pas de conflit en cas de changement de pseudo.

---

## Fonctions internes

| Fonction | Description |
|---|---|
| `te_setWaiting(p)` | Spectateur, téléport au spawn, titre d'attente |
| `te_setSpectating(p)` | Élimination définitive, son de mort |
| `te_setPlaying(p)` | Activation du joueur (survie), téléport, son de départ |
| `te_cleanLists()` | Suppression des entrées invalides (déconnectés) |
| `te_nextPlayer()` | Sélection du prochain sauteur ou fin de partie |
| `te_endGame()` | Annonce du vainqueur, nettoyage des variables |
| `te_forceStop(stopper)` | Arrêt admin avec broadcast |

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
    ├─ Compte à rebours 5s
    ├─ Enregistrement → te_setWaiting(tous)
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
| 2.1 | Correction `<SERVER-NAME>` → `SERVER-NAME` (caractères `<>` réservés par Skript) |
| 2.1 | Correction `play sound` : suppression volume/pitch inline (bug de parsing Skript) |
| 2.1 | Correction `world of player` → `player's world` dans les triggers commande |
| 2.0 | Réécriture complète Skript 2.9+ / 1.21.x, suppression addons dépréciés |
| 2.0 | UUID pour les variables joueurs, gestion déconnexion, `/te info` |
| 1.0 | Version initiale — Skript 1.12.2 + SkQuery / TuSKe / skRayFall / Skellett |

---

## Crédits

| Rôle | Personne |
|---|---|
| Auteur & conception | Clément Raes — [Odysseum04](https://github.com/odysseum04) |
| Assistance à la génération | [Claude AI](https://claude.ai) (Anthropic) |
