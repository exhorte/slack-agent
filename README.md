<div align="center">

<img src="docs/assets/banner.svg" alt="Slack AI Agent — l'agent IA qui qualifie chaque nouveau membre de votre communauté Slack" width="100%"/>

<br/>

[![Node.js](https://img.shields.io/badge/Node.js-20%2B-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![Slack Bolt](https://img.shields.io/badge/Slack-Bolt%20v5-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://slack.dev/bolt-js)
[![LangChain](https://img.shields.io/badge/LangChain-1.x-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)](https://js.langchain.com)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4-412991?style=for-the-badge&logo=openai&logoColor=white)](https://platform.openai.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-JSONB-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![License](https://img.shields.io/badge/License-ISC-blue?style=for-the-badge)](#-licence)

**Chaque personne qui rejoint votre Slack est un lead potentiel.**<br/>
**Slack AI Agent la détecte, l'enrichit, la score avec GPT-4 et prévient votre équipe — automatiquement.**

[Démarrage rapide](#-démarrage-rapide) •
[Fonctionnement](#-comment-ça-marche) •
[Configuration](#%EF%B8%8F-configuration) •
[Déploiement](#%EF%B8%8F-déploiement-sur-render) •
[Roadmap](#%EF%B8%8F-roadmap)

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/exhorte/slack-agent)

</div>

---

## 💡 Pourquoi ce projet ?

Les communautés Slack attirent des centaines de nouveaux membres : développeurs, décideurs, curieux, futurs clients… Mais **qui mérite vraiment l'attention de votre équipe commerciale ?**

Qualifier chaque membre à la main prend du temps, et les meilleurs prospects passent souvent inaperçus.

**Slack AI Agent** règle le problème. Dès qu'un membre arrive, l'agent :

1. 🕵️ **récupère** son profil Slack (nom, poste, email, fuseau horaire),
2. 🌐 **enrichit** ces données avec des recherches publiques (site de l'entreprise, profil GitHub),
3. 🧠 **analyse** le tout avec GPT-4 selon *votre* produit et *votre* entreprise,
4. 🎯 **attribue** un **Fit Score de 0 à 100**, accompagné d'observations et de recommandations,
5. 📣 **publie** une fiche claire et colorée dans un canal privé de votre équipe,
6. 🗄️ **historise** chaque analyse dans PostgreSQL pour le suivi et le reporting.

> **Résultat :** votre équipe sait en quelques secondes *qui* contacter, *pourquoi*, et *comment*.

---

## ✨ Fonctionnalités

<table>
<tr>
<td width="50%" valign="top">

### ⚡ Temps réel
Écoute les événements `team_join` et `member_joined_channel` via **Socket Mode** : aucune URL publique ni webhook à exposer.

### 🌐 Enrichissement automatique
Récupère le titre du site web de l'entreprise à partir du domaine email et recherche un profil GitHub correspondant. Les emails personnels (Gmail, Outlook…) sont ignorés.

### 🧠 Analyse IA contextualisée
Prompt LangChain configuré avec le nom de **votre entreprise** et de **votre produit**. Il évalue le poste, la taille de l'entreprise, le profil technique et le pouvoir de décision budgétaire.

</td>
<td width="50%" valign="top">

### 🎯 Fit Score visuel
Score de 0 à 100 avec un code couleur immédiat (🟢 🟡 🟠 🔴), 3 à 5 observations et 2 à 4 recommandations d'approche.

### 🗄️ Historique persistant
Chaque analyse est enregistrée dans PostgreSQL (JSONB) avec son statut d'envoi Slack. Aucune donnée n'est perdue, même si la publication échoue.

### 🛡️ Robuste
Valeurs par défaut en cas d'échec de l'IA, timeouts réseau, endpoint `/health`, arrêt propre (`SIGINT`/`SIGTERM`) et blueprint Render prêt à l'emploi.

</td>
</tr>
</table>

---

## 👀 Aperçu

Voici ce que votre équipe reçoit dans Slack pour chaque nouveau membre :

<div align="center">
<img src="docs/assets/slack-preview.svg" alt="Exemple de fiche d'analyse publiée dans Slack" width="720"/>
</div>

### 🎨 Lecture du Fit Score

| Score | Niveau | Couleur Slack | Action suggérée |
|:-----:|:-------|:-------------:|:----------------|
| ![80-100](https://img.shields.io/badge/80--100-Hot-36a64f?style=flat-square) | 🔥 Prospect chaud | `#36a64f` | Contact personnalisé sous 24 h |
| ![60-79](https://img.shields.io/badge/60--79-Warm-ffb84d?style=flat-square) | ✨ Prospect tiède | `#ffb84d` | Nurturing ciblé, invitation à une démo |
| ![40-59](https://img.shields.io/badge/40--59-Cool-ff9500?style=flat-square) | 🌱 À surveiller | `#ff9500` | Suivi de l'engagement dans la communauté |
| ![0-39](https://img.shields.io/badge/0--39-Cold-ff4444?style=flat-square) | ❄️ Faible adéquation | `#ff4444` | Aucune action commerciale |

---

## 🔄 Comment ça marche

```mermaid
sequenceDiagram
    autonumber
    participant S as 💬 Slack
    participant A as 🤖 Slack AI Agent
    participant W as 🌐 Web public
    participant L as 🧠 OpenAI GPT-4
    participant D as 🗄️ PostgreSQL

    S->>A: Événement team_join / member_joined_channel
    A->>S: users.info (profil complet)
    alt Email professionnel
        A->>W: Titre du site de l'entreprise
        A->>W: Recherche du profil GitHub
    end
    A->>L: Prompt (profil + recherches + contexte produit)
    L-->>A: JSON { fitScore, insights, recommendations }
    A->>D: INSERT member_analyses
    A->>S: chat.postMessage (canal privé)
    A->>D: UPDATE sent_to_slack = TRUE
```

### 🏗️ Architecture

```mermaid
flowchart LR
    subgraph SLACK["💬 Workspace Slack"]
        EV["Nouveaux membres"]
        CH["🔒 Canal privé équipe"]
    end

    subgraph AGENT["🤖 Slack AI Agent · Node.js"]
        BOLT["Bolt App · Socket Mode"]
        RES["Module de recherche · axios"]
        AI["Chaîne LangChain"]
        EXP["Express · /health"]
    end

    subgraph EXT["🌐 Services externes"]
        WEB["Site de l'entreprise"]
        GH["API GitHub"]
        OAI["OpenAI GPT-4"]
    end

    DB[("🗄️ PostgreSQL")]

    EV -- WebSocket --> BOLT
    BOLT --> RES
    RES --> WEB
    RES --> GH
    RES --> AI
    AI <--> OAI
    AI --> DB
    AI -- Block Kit --> CH
    EXP -. health check .- RENDER["☁️ Render"]
```

---

## 🧰 Stack technique

| Couche | Technologie | Rôle |
|:-------|:------------|:-----|
| **Runtime** | Node.js 20+ (ES Modules) | Exécution de l'agent |
| **Slack** | [`@slack/bolt`](https://slack.dev/bolt-js) · [`@slack/web-api`](https://slack.dev/node-slack-sdk/web-api) | Événements en temps réel, profils, publication Block Kit |
| **IA** | [`@langchain/openai`](https://js.langchain.com) · [`@langchain/core`](https://js.langchain.com) | Prompt templating et appel à GPT-4 (`temperature: 0.3`) |
| **HTTP** | [`express`](https://expressjs.com) v5 · [`axios`](https://axios-http.com) | Health check, endpoint de test, enrichissement web |
| **Données** | [`pg`](https://node-postgres.com) · PostgreSQL | Pool de connexions, stockage JSONB |
| **Infra** | [Render](https://render.com) (`render.yaml`) | Web service et base managée, déployés depuis un blueprint |

---

## 🚀 Démarrage rapide

### 1. Prérequis

- **Node.js** ≥ 20
- Une base **PostgreSQL** (locale, Docker ou Render)
- Un workspace **Slack** où vous avez le droit d'installer des apps
- Une **clé API OpenAI** avec accès à GPT-4

### 2. Créer l'application Slack

Rendez-vous sur [api.slack.com/apps](https://api.slack.com/apps), puis **Create New App → From an app manifest**, et collez ce manifeste :

<details>
<summary><b>📋 Manifeste Slack (cliquer pour déplier)</b></summary>

```yaml
display_information:
  name: Slack AI Agent
  description: Qualifie automatiquement les nouveaux membres grâce à l'IA
  background_color: "#4A154B"
features:
  bot_user:
    display_name: AI Agent
    always_online: true
oauth_config:
  scopes:
    bot:
      - users:read          # Lire les profils membres
      - users:read.email    # Lire les emails (indispensable pour l'enrichissement)
      - channels:read       # Recevoir member_joined_channel (canaux publics)
      - chat:write          # Publier les analyses
settings:
  event_subscriptions:
    bot_events:
      - team_join
      - member_joined_channel
  interactivity:
    is_enabled: false
  org_deploy_enabled: false
  socket_mode_enabled: true
  token_rotation_enabled: false
```

</details>

Ensuite :

1. **Basic Information → App-Level Tokens** : générez un token avec le scope `connections:write` → `SLACK_APP_TOKEN` (`xapp-…`)
2. **Basic Information → Signing Secret** → `SLACK_SIGNING_SECRET`
3. **Install App** : installez l'app dans le workspace → `SLACK_BOT_TOKEN` (`xoxb-…`)
4. Créez un **canal privé** (ex. `#lead-intelligence`), invitez-y le bot avec `/invite @AI Agent` et copiez son ID → `SLACK_PRIVATE_CHANNEL_ID`

> [!TIP]
> Pour trouver l'ID d'un canal, faites un clic droit sur le canal → **Afficher les détails** : l'ID (`C0…`) se trouve en bas de la fenêtre.

### 3. Installer et lancer

```bash
# Cloner le dépôt
git clone https://github.com/exhorte/slack-agent.git
cd slack-agent

# Installer les dépendances
npm install

# Configurer l'environnement
cp .env.sample .env   # puis complétez les valeurs

# Lancer en mode développement (rechargement automatique)
npm run dev
```

Si tout est correctement configuré, vous verrez :

```text
[INFO] 🗄️ Initilazing database...
[INFO] Database connected
[INFO] Database schema initialized
[INFO] 🚀 Express server running on port 3000
[INFO] ⚡️ Slack bot connected
[INFO] 🎉 Slack AI Agent is running!
[INFO] Test endpoint: POST http://localhost:3000/test/analyze-member
```

> [!NOTE]
> La table `member_analyses` et ses index sont créés automatiquement au premier démarrage : aucune migration n'est nécessaire.

---

## ⚙️ Configuration

Toutes les options passent par des variables d'environnement (voir [`.env.sample`](.env.sample)).

| Variable | Requis | Description | Exemple |
|:---------|:------:|:------------|:--------|
| `SLACK_BOT_TOKEN` | ✅ | Token OAuth du bot | `xoxb-…` |
| `SLACK_APP_TOKEN` | ✅ | Token applicatif pour le Socket Mode | `xapp-…` |
| `SLACK_SIGNING_SECRET` | ✅ | Secret de signature de l'app | `a1b2c3…` |
| `SLACK_PRIVATE_CHANNEL_ID` | ✅ | Canal où sont publiées les analyses | `C1234567890` |
| `OPENAI_API_KEY` | ✅ | Clé API OpenAI | `sk-…` |
| `DATABASE_URL` | ✅ | Chaîne de connexion PostgreSQL (SSL activé) | `postgres://user:pass@host:5432/db` |
| `COMPANY_NAME` | ➖ | Nom de votre entreprise, injecté dans le prompt | `Acme Inc.` |
| `COMPANY_PRODUCT` | ➖ | Produit à qualifier, injecté dans le prompt | `Acme Analytics Cloud` |
| `NODE_ENV` | ➖ | `development` active les logs debug et l'endpoint de test | `development` |
| `PORT` | ➖ | Port du serveur Express (défaut : `3000`) | `3000` |

> [!IMPORTANT]
> Plus `COMPANY_NAME` et `COMPANY_PRODUCT` sont précis, plus le Fit Score est pertinent : ce sont eux qui donnent à GPT-4 le contexte de qualification.

---

## 🔌 API HTTP

| Méthode | Route | Disponibilité | Description |
|:-------:|:------|:--------------|:------------|
| `GET` | `/health` | Toujours | Sonde de santé (utilisée par Render) |
| `POST` | `/test/analyze-member` | `NODE_ENV=development` | Lance une analyse complète sur un profil fictif |

### Tester une analyse sans attendre un vrai membre

```bash
curl -X POST http://localhost:3000/test/analyze-member \
  -H "Content-Type: application/json" \
  -d '{
    "memberInfo": {
      "name": "Camille Laurent",
      "email": "camille@acme.io",
      "title": "VP Engineering"
    }
  }'
```

> [!WARNING]
> L'endpoint de test exécute **tout le pipeline** : il appelle OpenAI (et consomme donc des crédits), écrit en base et **publie réellement** dans le canal Slack configuré.

---

## 🗄️ Modèle de données

```mermaid
erDiagram
    MEMBER_ANALYSES {
        serial id PK
        varchar member_id "ID Slack (indexé)"
        varchar member_name "NOT NULL"
        varchar member_email
        varchar member_title
        varchar member_timezone
        integer fit_score "0 à 100"
        jsonb insights
        jsonb recommendations
        jsonb research_data
        timestamp analyzed_at "indexé"
        boolean sent_to_slack
        timestamp sent_to_slack_at
        timestamp created_at
        timestamp updated_at
    }
```

### 📊 Requêtes utiles

<details>
<summary><b>Top 10 des prospects des 7 derniers jours</b></summary>

```sql
SELECT member_name, member_title, member_email, fit_score, analyzed_at
FROM member_analyses
WHERE analyzed_at > NOW() - INTERVAL '7 days'
ORDER BY fit_score DESC
LIMIT 10;
```
</details>

<details>
<summary><b>Analyses non publiées dans Slack (à rejouer)</b></summary>

```sql
SELECT id, member_name, fit_score, analyzed_at
FROM member_analyses
WHERE sent_to_slack = FALSE
ORDER BY analyzed_at DESC;
```
</details>

<details>
<summary><b>Répartition des membres par niveau de Fit Score</b></summary>

```sql
SELECT
  CASE
    WHEN fit_score >= 80 THEN '🔥 Hot'
    WHEN fit_score >= 60 THEN '✨ Warm'
    WHEN fit_score >= 40 THEN '🌱 Cool'
    ELSE '❄️ Cold'
  END AS niveau,
  COUNT(*) AS membres,
  ROUND(AVG(fit_score)) AS score_moyen
FROM member_analyses
GROUP BY niveau
ORDER BY score_moyen DESC;
```
</details>

---

## ☁️ Déploiement sur Render

Le projet inclut un blueprint [`render.yaml`](render.yaml) qui provisionne **en un clic** :

- 🗄️ une base **PostgreSQL** managée (`slack-agent-db`),
- 🌐 un **web service** Node.js (`slack-ai-agent`) avec health check sur `/health`,
- 🔗 l'injection automatique de `DATABASE_URL`.

<div align="center">

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/exhorte/slack-agent)

</div>

Il ne reste ensuite qu'à renseigner les secrets (`SLACK_*`, `OPENAI_API_KEY`) dans le dashboard Render, ainsi que `COMPANY_NAME` et `COMPANY_PRODUCT`.

> [!NOTE]
> Grâce au Socket Mode, l'agent ouvre lui-même une connexion WebSocket vers Slack : **aucune Request URL publique n'est à configurer** côté Slack.

---

## 📁 Structure du projet

```text
slack-agent/
├── 📄 index.js          # Classe SlackAIAgent : événements Slack, recherche, IA, publication
├── 🗄️ db.js             # Pool PostgreSQL, schéma, persistance des analyses
├── ☁️ render.yaml       # Blueprint Render (web service + base de données)
├── 🔐 .env.sample       # Modèle des variables d'environnement
├── 📦 package.json      # Dépendances et scripts npm
└── 🖼️ docs/assets/      # Illustrations du README
```

### 🎛️ Personnaliser l'agent

| Vous voulez… | Où intervenir |
|:-------------|:--------------|
| Changer de modèle ou de température | Constructeur `ChatOpenAI` dans [`index.js`](index.js#L30-L34) |
| Adapter les critères de qualification | Prompt dans [`analyzeWithAI()`](index.js#L198-L221) |
| Ajouter des domaines d'email personnels | [`isPersonalEmail()`](index.js#L326-L330) |
| Modifier les seuils de couleur | [`postAnalysisToChannel()`](index.js#L260-L263) |
| Ajouter une source d'enrichissement | [`doBasicResearch()`](index.js#L134-L151) |

---

## 🔐 Confidentialité et conformité

Cet agent traite des **données personnelles** (nom, email, poste) et les transmet à des services tiers (OpenAI, GitHub). Avant de le déployer :

- 📢 **Informez** les membres de votre communauté (charte, message d'accueil, politique de confidentialité).
- ⚖️ **Vérifiez la base légale** de ce traitement au regard du **RGPD** (intérêt légitime, consentement…).
- 🗓️ **Définissez une durée de conservation** et purgez régulièrement la table `member_analyses`.
- 🔑 **Ne commitez jamais** votre fichier `.env` (il est déjà exclu par `.gitignore`).
- 🤝 Utilisez les analyses comme une **aide à la décision**, pas comme un jugement automatisé sur les personnes.

---

## 🗺️ Roadmap

- [ ] 🔁 Dédupliquer les analyses quand `team_join` et `member_joined_channel` se déclenchent pour le même membre
- [ ] 🧩 Sorties structurées OpenAI (`withStructuredOutput`) au lieu du parsing JSON manuel
- [ ] 🎚️ Modèle OpenAI configurable par variable d'environnement (`OPENAI_MODEL`)
- [ ] 🔑 Authentification à l'API GitHub pour dépasser la limite de requêtes anonymes
- [ ] 🔘 Boutons interactifs Slack (« Assigner », « Envoyer au CRM », « Ignorer »)
- [ ] 🔗 Intégrations CRM : HubSpot, Salesforce, Pipedrive
- [ ] 📈 Tableau de bord analytics des leads
- [ ] 🧪 Suite de tests automatisés et CI GitHub Actions

---

## 🤝 Contribuer

Les contributions sont les bienvenues !

1. Forkez le projet
2. Créez une branche (`git checkout -b feat/ma-fonctionnalite`)
3. Committez vos changements (`git commit -m "feat: ajoute ma fonctionnalité"`)
4. Poussez la branche (`git push origin feat/ma-fonctionnalite`)
5. Ouvrez une **Pull Request** 🎉

Un bug ou une idée ? [Ouvrez une issue](https://github.com/exhorte/slack-agent/issues).

---

## 📄 Licence

Distribué sous licence **ISC**. Voir [`package.json`](package.json).

<div align="center">

<br/>

**Construit avec ❤️, ☕ et beaucoup de tokens.**

Si ce projet vous est utile, laissez-lui une ⭐ sur GitHub !

</div>

