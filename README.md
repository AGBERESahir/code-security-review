# code-security-review

Fait appliquer à Claude la **même checklist** à chaque relecture de code, orientée **qualité et sécurité**, au lieu d'une relecture « au jugé ». Une relecture au jugé rate toujours les mêmes catégories de défauts, celles qui ne sautent pas aux yeux.

Skill Claude : un dossier d'instructions que Claude charge automatiquement quand ta demande correspond. Tu n'as rien à invoquer : décris ton problème normalement.

## Quand il se déclenche

- tu colles du code et demandes une relecture, un « review » ou un audit ;
- tu demandes « est-ce sûr ? » ou « check my code » ;
- tu veux un avis avant une mise en production ou une revue de pull request, même sans parler de sécurité.

Exemples de messages :

- « Peux-tu relire cette fonction de connexion avant que je la déploie ? »
- « Review de cette pull request, surtout la partie API. »
- « Check my code : est-ce que ce formulaire d'upload est sûr ? »

## Ce que le skill fait faire à Claude

1. **Comprendre avant de juger** : rôle du code, entrées (utilisateur, réseau, fichiers, variables d'environnement), sorties, niveau d'exposition. Un même défaut n'a pas la même gravité en interne ou sur une API publique.
2. **Passer la checklist**, catégorie par catégorie, avec l'emplacement exact de chaque constat :
   - injections (SQL, commandes, XSS, chemins de fichiers, désérialisation, redirections ouvertes, SSRF) ;
   - authentification et autorisation (contrôle côté serveur, IDOR, hachage des mots de passe, sessions et JWT, CSRF) ;
   - secrets et configuration (clés en dur, `.env` versionné, mode debug, CORS trop permissif) ;
   - données et vie privée (logs sensibles, chiffrement, réponses d'API trop bavardes) ;
   - robustesse (erreurs avalées, concurrence, absence de limites, ressources non libérées) ;
   - dépendances (versions anciennes, `npm audit`, `pip-audit`, `cargo audit`) ;
   - qualité (duplication, fonctions trop longues, tests manquants sur le chemin critique).
3. **Classer par gravité** : Critique, Élevé, Moyen, Faible/Style.
4. **Proposer le code corrigé** pour chaque constat Critique et Élevé, pas seulement une description.
5. **Dire ce qui n'a pas pu être vérifié.** Claude ne déclare jamais « sûr » : il dit « aucun problème trouvé dans ce qui a été fourni ».

## Format de la réponse

```
## Résumé
## Constats
### [Critique] Titre court : fichier:ligne
- Problème / Risque concret / Correctif (code)
## Points positifs
## Non vérifié
```

Le **risque concret** (ce qu'un attaquant ou un utilisateur malchanceux peut faire) est toujours expliqué, pour que le correctif se comprenne et ne reste pas une règle à suivre aveuglément.

## Installation

**Claude Code**

```bash
git clone https://github.com/AGBERESahir/code-security-review.git ~/.claude/skills/code-security-review
```

Le dossier doit s'appeler `code-security-review` et contenir `SKILL.md` à sa racine. Relance Claude Code si une session était ouverte.

**Claude.ai**

Zippe le dossier `code-security-review` (avec `SKILL.md` dedans), puis ajoute l'archive dans **Paramètres > Skills**.

## Vérifier qu'il fonctionne

Envoie l'un des messages d'exemple ci-dessus. La réponse doit suivre la structure décrite plus haut. Si Claude répond de façon libre, vérifie que le skill est activé dans tes paramètres et que `SKILL.md` n'est pas dans un sous-dossier.

## Limites

- Claude ne voit que ce que tu lui montres : configuration serveur, dépendances non listées et code absent restent **non vérifiés**, et la réponse le précise.
- Ce n'est pas un scanner de sécurité automatique ni un test d'intrusion : il ne remplace ni `npm audit`, ni un audit professionnel pour un système critique.
- Une relecture peut manquer des défauts ou signaler des faux positifs : garde un œil critique sur chaque constat.

## Voir aussi

- [secret-guard](https://github.com/AGBERESahir/secret-guard) : empêche de committer clés et tokens.
- [systematic-debugging](https://github.com/AGBERESahir/systematic-debugging) : quand le code fait autre chose que prévu.
- [rls-audit](https://github.com/AGBERESahir/rls-audit) : pour la sécurité des tables Supabase.

## Fichiers

| Fichier | Rôle |
|---|---|
| `SKILL.md` | Les instructions : nom, description de déclenchement, démarche et format de sortie. |
| `README.md` | Cette page, pour les humains (Claude ne la lit pas). |

## Licence

Aucune licence n'est encore indiquée dans ce dépôt ; ajoute un fichier `LICENSE` avant de le réutiliser ou de le partager.
