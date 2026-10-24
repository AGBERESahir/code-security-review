---
name: code-security-review
description: Revue de code orientée qualité et sécurité avec une checklist systématique (injection SQL/XSS/commandes, secrets en clair, validation des entrées, authentification et autorisation, dépendances vulnérables, gestion des erreurs, fuites de données dans les logs). À utiliser dès que l'utilisateur colle du code et demande une relecture, un "review", un audit, "est-ce sûr ?", "check my code", un avis avant mise en production ou une revue de pull request, même s'il ne mentionne pas explicitement la sécurité. Use for any code review, security audit or PR review request.
---

# Revue de code et sécurité

Objectif : appliquer le même passage rigoureux à chaque revue, au lieu de relire "au jugé". Une revue au jugé rate systématiquement les mêmes catégories de défauts (celles qui ne sautent pas aux yeux), d'où la checklist.

## Démarche

1. **Comprendre avant de juger.** Identifier le rôle du code, ses entrées (utilisateur, réseau, fichiers, variables d'environnement), ses sorties et son niveau d'exposition (interne, API publique, formulaire web). Un défaut n'a pas la même gravité selon l'exposition.
2. **Passer la checklist** ci-dessous, catégorie par catégorie, en notant les constats avec leur emplacement exact (fichier:ligne ou extrait).
3. **Classer par gravité** : Critique (exploitable à distance, fuite de données, exécution de code), Élevé, Moyen, Faible/Style.
4. **Proposer un correctif concret** pour chaque constat Critique et Élevé, avec le code corrigé, pas seulement une description.
5. **Dire ce qui n'a pas pu être vérifié** (code non fourni, configuration serveur, dépendances non listées). Ne jamais déclarer "sûr" : dire "aucun problème trouvé dans ce qui a été fourni".

## Checklist

**Entrées et injections**
- Requêtes SQL construites par concaténation ou f-string au lieu de requêtes paramétrées
- Commandes shell construites avec des entrées utilisateur (`os.system`, `exec`, backticks)
- XSS : données non échappées injectées dans le HTML (`innerHTML`, `dangerouslySetInnerHTML`)
- Chemins de fichiers issus de l'utilisateur (path traversal `../`), téléversements sans vérification de type/taille
- Désérialisation de données non fiables (`pickle`, `eval`, YAML non sécurisé)
- Redirections ouvertes, SSRF (URL fournie par l'utilisateur appelée côté serveur)

**Authentification et autorisation**
- Contrôle d'accès vérifié côté serveur et non seulement côté client
- Vérification de propriété de la ressource (IDOR : l'utilisateur A peut-il lire l'objet de B en changeant un id ?)
- Mots de passe hachés avec un algorithme adapté (bcrypt, argon2), jamais en clair ni en MD5/SHA1
- Sessions/JWT : expiration, secret robuste, algorithme imposé, cookies `HttpOnly`/`Secure`/`SameSite`
- Protection CSRF sur les actions modifiant l'état

**Secrets et configuration**
- Clés d'API, mots de passe, tokens en dur dans le code ou dans l'historique Git
- Fichiers `.env` versionnés ; secrets envoyés au front-end
- Mode debug activé, CORS trop permissif (`*` avec identifiants), messages d'erreur détaillés exposés

**Données et vie privée**
- Logs contenant mots de passe, tokens, données personnelles, numéros de carte
- Données sensibles non chiffrées en transit (HTTP) ou au repos
- Réponses d'API renvoyant plus de champs que nécessaire

**Robustesse**
- Erreurs avalées silencieusement (`except: pass`), absence de gestion des cas `null`/vide
- Conditions de concurrence (vérifier puis agir sans verrou/transaction)
- Absence de limites : taille des entrées, pagination, débit (rate limiting)
- Ressources non libérées (fichiers, connexions)

**Dépendances**
- Versions anciennes ou abandonnées, paquets inutiles, versions non figées
- Recommander `npm audit`, `pip-audit`, `cargo audit` ou équivalent selon l'écosystème

**Qualité**
- Duplication, fonctions trop longues, noms trompeurs, absence de tests sur le chemin critique

## Format de sortie

```
## Résumé
[2-3 phrases : verdict global, nombre de constats par gravité]

## Constats
### [Critique] Titre court : fichier:ligne
- Problème : ...
- Risque concret : ce qu'un attaquant ou un utilisateur malchanceux peut faire
- Correctif : (code)

## Points positifs
[Ce qui est bien fait : utile et honnête]

## Non vérifié
[Limites de la revue]
```

Rester constructif : expliquer le *risque concret* plutôt qu'énoncer une règle, pour que la personne comprenne pourquoi corriger.
