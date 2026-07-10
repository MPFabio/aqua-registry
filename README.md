# aqua-registry — catalogue interne d'outils (gouvernance SLSA)

Source de vérité **centrale** des outils CLI approuvés pour toute l'organisation.
Consommé par les repos projet (ex. `devops-base`) via `github_content` + **tag**.
Aucun binaire ne transite ici : le catalogue pointe vers la **source officielle
upstream**, ce qui préserve la vérification **cosign / attestations / checksum** d'aqua.

## Contenu

| Fichier | Rôle |
|---------|------|
| `registry.yaml` | **Le catalogue** : outils + URLs upstream + métadonnées SLSA (recopié du registre standard aqua `v4.534.0`). |
| `aqua.yaml` | Harness : versions de référence pour prouver le catalogue (non consommé par les projets). |
| `aqua-policy.yaml` | N'autorise que le registre interne (pour le gate). |
| `aqua-checksums.json` | Lock des versions de référence. |
| `.github/workflows/aqua-verify.yml` | **Gate SLSA** : bloque le merge si une vérif casse. |

## Consommer ce catalogue (repo projet)

```yaml
# aqua.yaml du projet
registries:
  - name: internal
    type: github_content
    repo_owner: MPFabio
    repo_name: aqua-registry
    ref: v1.0.0          # tag immuable = reproductibilité
    path: registry.yaml
packages:
  - name: kubernetes/kubectl@v1.30.14
    registry: internal
```

+ une `aqua-policy.yaml` côté projet qui autorise ce registre `github_content`.
Le repo étant **privé**, le build (et la CI du projet) doit fournir un token GitHub
en lecture sur ce repo (`AQUA_GITHUB_TOKEN` / `GITHUB_TOKEN`).

## Cycle de gouvernance

- **Ajouter / modifier un outil du catalogue** → PR ici → `aqua-verify` (SLSA) → merge → **nouveau tag** (ex. `v1.1.0`).
- **Consommer une nouvelle version d'outil** → PR dans le repo projet (bump `@version` + éventuel bump du `ref`), piloté par Renovate.
