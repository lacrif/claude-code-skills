---
name: plan
description: Créer un plan d'exécution fiable et actionnable pour une fonctionnalité, un bug, une migration, une refactorisation ou un projet. Utiliser quand l'utilisateur demande un plan, une feuille de route, des étapes, des phases, une stratégie d'implémentation ou souhaite enregistrer un plan dans un fichier Markdown.
---

# Plan d'exécution

Produis un plan concret, fondé sur les éléments réellement disponibles, puis enregistre-le dans un fichier Markdown. Ne livre jamais uniquement le plan dans la conversation.

## 1. Cadrer avant de planifier

1. Lis la demande et identifie l'objectif, le périmètre, les contraintes, les livrables et les critères de réussite.
2. Explore les fichiers, la documentation et la configuration pertinents avant d'affirmer qu'une modification est nécessaire. Cite les chemins précis dans le plan.
3. Distingue explicitement les faits vérifiés, les hypothèses et les questions bloquantes.
4. Si une information manquante change sensiblement le périmètre, pose une question courte avant d'écrire le plan. Sinon, avance avec une hypothèse clairement signalée.

## 2. Définir le fichier de plan

1. Utilise le chemin fourni par l'utilisateur s'il existe.
2. Sinon, crée `docs/plans/YYYY-MM-DD-<sujet>-plan.md`, avec un sujet court en kebab-case. Crée le dossier si nécessaire.
3. N'écrase jamais un plan existant sans demande explicite. En cas de collision, choisis un suffixe descriptif ou demande confirmation.
4. Écris le contenu complet dans ce fichier avant de répondre. Indique ensuite son chemin à l'utilisateur.

## 3. Écrire le plan par phases

Structure le document ainsi :

```markdown
# Plan — <objectif>

## Objectif
<résultat attendu et valeur livrée>

## Périmètre
- Inclus : ...
- Exclu : ...

## Contexte vérifié
- `<chemin>` — <constat utile>

## Hypothèses et décisions ouvertes
- <hypothèse ou décision, impact et responsable si connu>

## Phase 1 — <nom orienté résultat>
1. <action atomique, avec fichier/composant concerné et résultat attendu>
2. <action suivante>

**Validation :** <commande, test, contrôle ou critère observable>

## Phase 2 — <nom orienté résultat>
1. ...

**Validation :** ...

## Risques et atténuations
- <risque> — <prévention, plan de retour arrière ou décision>

## Critères d'acceptation
- [ ] <critère testable>

## Ordre d'exécution et dépendances
1. <dépendance ou jalon>
```

Applique ces règles :

- Numérote toutes les actions exécutables ; commence une nouvelle phase lorsqu'un jalon, une responsabilité ou un type de travail change.
- Fais en sorte que chaque étape soit atomique, ordonnée, vérifiable et liée à un fichier, un composant ou un comportement précis lorsque cela est connu.
- Décris le quoi, le où et le pourquoi ; évite les formulations vagues comme « mettre à jour le backend ».
- Place d'abord les décisions, contrats, migrations et fondations ; puis l'implémentation ; puis les tests, la documentation et le déploiement.
- Prévois une validation à chaque phase, avec des commandes uniquement si elles ont été vérifiées dans le dépôt. Sinon, décris le contrôle attendu sans inventer de commande.
- Identifie les impacts sur la compatibilité, les données, la sécurité, les performances, l'observabilité, la documentation et le rollback lorsqu'ils sont pertinents.
- Garde le plan proportionné : fusionne les micro-actions, mais ne masque pas les décisions risquées ou irréversibles.
- Ne réalise pas l'implémentation pendant cette tâche, sauf demande explicite de l'utilisateur.

## 4. Contrôler la qualité avant livraison

Avant de terminer, vérifie que le fichier contient :

1. un objectif et un périmètre ;
2. au moins une phase avec des étapes numérotées ;
3. des validations et critères d'acceptation testables ;
4. les hypothèses, risques et dépendances pertinents ;
5. des chemins et commandes vérifiés, sans détail inventé ;
6. une séquence qui permet à un autre agent ou développeur d'exécuter le plan sans reconstituer l'intention.

Réponds de façon concise avec le chemin du fichier créé, l'objectif couvert et les éventuelles décisions à confirmer.
