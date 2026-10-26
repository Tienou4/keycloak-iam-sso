# Système IAM avec SSO — Keycloak

Projet structurant de 2ᵉ année de prépa intégrée (IUC Douala, 25 avril – 17 mai 2026) : **conception et déploiement d'un système de gestion des identités et des accès (IAM)** avec **authentification unique (SSO)**, sur **AWS EC2**.

![Keycloak](https://img.shields.io/badge/Keycloak-24.0-4D4D4D?logo=keycloak) ![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white) ![ELK](https://img.shields.io/badge/ELK-8.12-005571?logo=elasticsearch)

## Fonctionnalités

- **IdP central Keycloak 24** + PostgreSQL 16, conteneurisés avec Docker Compose
- **SSO OpenID Connect** (Authorization Code Flow, JWT signés) sur **n8n** et **Portainer** : un seul login, deux applications
- **RBAC** : realm `entreprise-iam` avec 4 rôles, 4 groupes et 5 utilisateurs de test
- **MFA TOTP** obligatoire pour les administrateurs
- **Anti brute-force** : verrouillage après 5 échecs (jusqu'à 15 min) + politique de mots de passe forte
- **HTTPS partout** : Nginx en reverse proxy + certificats Let's Encrypt (Certbot)
- **Audit de sécurité** : logs Keycloak collectés par Filebeat → Elasticsearch → tableaux de bord Kibana
- **Sauvegarde** quotidienne de PostgreSQL (`pg_dump`, script Bash)

## Architecture

```
Utilisateur ──HTTPS──> Nginx (TLS Let's Encrypt)
                         ├── Keycloak 24 (IdP, realm entreprise-iam) ── PostgreSQL 16
                         ├── n8n        (client OIDC)
                         └── Portainer  (client OIDC)
Logs Docker ── Filebeat ── Elasticsearch ── Kibana (audit)
Hébergement : AWS EC2 (Security Groups restreints)
```

## Modèle RBAC

| Rôle | Groupe | Accès |
|---|---|---|
| `admin-iam` | GRP-Admins | Toutes les applications + administration Keycloak (MFA obligatoire) |
| `responsable-it` | GRP-IT | Gestion des utilisateurs |
| `employe` | GRP-Employes | Accès standard limité |
| `stagiaire` | GRP-Stagiaires | Lecture seule |

## Tests de validation

| Test | Résultat |
|---|---|
| SSO admin → n8n (rôle présent dans le JWT) | ✅ |
| Connexion admin refusée sans code TOTP | ✅ |
| Verrouillage du compte après 5 échecs + trace dans Kibana | ✅ |
| SSO cross-app n8n + Portainer | ✅ |
| Révocation de rôle effective immédiatement | ✅ |
| Sauvegarde PostgreSQL | ✅ |
| Certificats HTTPS valides | ✅ |

## Perspectives

Synchronisation LDAP/Active Directory · haute disponibilité (cluster Keycloak, réplication PostgreSQL)

## Contenu du dépôt

```
docs/      rapport et schémas UML / architecture
docker/    docker-compose.yml (IAM) et docker-compose-elk.yml
nginx/     configuration du reverse proxy
scripts/   sauvegarde PostgreSQL
```

> 🚧 Fichiers de configuration en cours de mise en ligne (secrets retirés).
