# Validation du pipeline CI documentaire

Ce fichier atteste du fonctionnement du workflow [`.github/workflows/docs-quality.yml`](../.github/workflows/docs-quality.yml) introduit en TP 1.7.

## Procédure de validation à exécuter (après push initial)

### Étape 1 — PR de test « cassée intentionnellement »

Créer une branche `test/ci-validation`, ajouter un fichier `docs/scratch.md` contenant délibérément :

1. Un lien externe mort : `[lien mort](https://this-domain-definitely-does-not-exist-12345.test)`.
2. Un bloc Mermaid invalide :
   ```mermaid
   broken syntax that does not parse
   ```
3. Une violation markdownlint (ex : titre `H1` dupliqué).

Ouvrir une pull request et vérifier que les **trois jobs** (`markdown-lint`, `link-check`, `mermaid-validate`) **rejettent la PR** avec messages d'erreur détaillés.

### Étape 2 — Correction et passage au vert

Pousser un commit qui corrige les trois défauts :

1. Corriger ou retirer le lien externe.
2. Remplacer le bloc Mermaid par une syntaxe valide :
   ```mermaid
   flowchart LR
       A --> B
   ```
3. Corriger la duplication de titre.

Vérifier que les trois jobs passent au vert.

### Étape 3 — CODEOWNERS

Vérifier que GitHub a automatiquement ajouté comme reviewers les owners désignés dans [`.github/CODEOWNERS`](../.github/CODEOWNERS) en fonction des fichiers modifiés (par exemple, une PR qui touche `06.4-vue-donnees.md` doit déclencher la revue de `@Titi7834`).

## Captures attendues à archiver dans cette section

Après exécution de la procédure ci-dessus, archiver dans `docs/screenshots/` :

- `pr-rouge-3-jobs.png` — capture de la PR avec les 3 jobs en échec et messages d'erreur.
- `pr-vert-3-jobs.png` — capture de la PR après correction, 3 jobs au vert.
- `codeowners-auto-reviewer.png` — capture des reviewers automatiquement assignés par GitHub.

## Statut au 2026-04-30

- ✅ Workflow `docs-quality.yml` produit avec 3 jobs : `markdown-lint`, `link-check`, `mermaid-validate`.
- ✅ Configuration `.markdownlint.json` produite (MD013/MD033/MD041 désactivés ; MD024 limité aux titres frères ; fenced code blocks).
- ✅ Configuration `link-check-config.json` produite (timeout 5 s, retry sur 429, ignore localhost).
- ✅ `CODEOWNERS` produit avec règle de fallback + 1 owner par section principale + ADR par auteur.
- 🚧 **Procédure de validation en attente d'exécution** : nécessite un push initial sur le remote GitHub avant de pouvoir ouvrir une PR de test. À exécuter en début de TP 1.8.

---

*Dernière mise à jour : 2026-04-30*
