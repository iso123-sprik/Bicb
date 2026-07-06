# Bicb - Global Competition Platform

## 🌍 Vision

Bicb est une plateforme mondiale de compétitions permettant à des personnes du monde entier de participer à des compétitions dans différents domaines (sport virtuel, chant, danse, photographie, jeux vidéo, cuisine, art, etc.).

## 📋 Table des matières

- [Architecture](#architecture)
- [Stack Technologique](#stack-technologique)
- [Installation](#installation)
- [Documentation](#documentation)
- [Développement](#développement)
- [Déploiement](#déploiement)

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  CLIENT (Web/Mobile)                │
├─────────────────────────────────────────────────────┤
│  React/Vue Frontend | Mobile App (React Native)     │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ HTTP/REST
                  │
┌─────────────────▼───────────────────────────────────┐
│              API GATEWAY                            │
├─────────────────────────────────────────────────────┤
│  Authentication | Rate Limiting | Routing           │
└─────────────────┬───────────────────────────────────┘
                  │
      ┌───────────┼───────────┬──────────────┐
      │           │           │              │
┌─────▼──┐ ┌─────▼──┐ ┌─────▼──┐ ┌──────▼──┐
│ Users  │ │Compet. │ │ Votes  │ │Payments │
│Service │ │Service │ │Service │ │Service  │
└────────┘ └────────┘ └────────┘ └─────────┘
      │           │           │              │
      └───────────┼───────────┴──────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│           DATABASE (PostgreSQL)                     │
├─────────────────────────────────────────────────────┤
│ Users | Competitions | Participants | Votes | etc.  │
└─────────────────────────────────────────────────────┘
```

## 🛠️ Stack Technologique

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Language**: TypeScript
- **Database**: PostgreSQL 16
- **Cache**: Redis 7
- **Authentication**: JWT + OAuth2
- **Payments**: Stripe API
- **Storage**: AWS S3

### Frontend
- **Framework**: React 18 + TypeScript
- **State Management**: Redux ou Context API
- **Styling**: Tailwind CSS
- **HTTP Client**: Axios

### DevOps
- **Containerization**: Docker
- **Orchestration**: Kubernetes (optionnel)
- **CI/CD**: GitHub Actions

## 📦 Installation

### Prérequis
- Node.js 18+
- Docker et Docker Compose
- PostgreSQL 16 (ou via Docker)
- Redis 7 (ou via Docker)

### Setup Rapide avec Docker

```bash
# 1. Clone le repository
git clone https://github.com/iso123-sprik/Bicb.git
cd Bicb

# 2. Copie le fichier .env
cp backend/.env.example backend/.env

# 3. Lance les services (PostgreSQL, Redis, etc.)
docker-compose up -d

# 4. Install les dépendances backend
cd backend
npm install

# 5. Lance les migrations
npm run migrate:run

# 6. Lance le serveur de développement
npm run dev
```

Le serveur sera disponible sur `http://localhost:3000`

## 📚 Documentation

Consultez les fichiers de documentation pour plus de détails :

- **[DATABASE.md](docs/DATABASE.md)** - Schéma de base de données complet
- **[API.md](docs/API.md)** - Spécification complète des endpoints REST
- **[AUTHENTICATION.md](docs/AUTHENTICATION.md)** - Système d'authentification JWT
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Architecture détaillée du projet

## 💻 Développement

### Structure du Projet

```
backend/
├── src/
│   ├── api/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── competitions/
│   │   ├── votes/
│   │   ├── rewards/
│   │   ├── payments/
│   │   └── challenges/
│   ├── models/
│   ├── middleware/
│   ├── services/
│   ├── utils/
│   ├── config/
│   ├── migrations/
│   └── index.ts
├── tests/
├── package.json
└── tsconfig.json
```

### Commandes Utiles

```bash
# Développement
npm run dev           # Lancer le serveur avec hot-reload

# Tests
npm test             # Exécuter les tests
npm run test:watch   # Tests en mode watch

# Linting
npm run lint         # Vérifier le style du code
npm run lint:fix     # Corriger les erreurs de style

# Database
npm run migrate:run     # Exécuter les migrations
npm run migrate:revert  # Annuler la dernière migration
npm run seed            # Peupler la base de données

# Build
npm run build        # Compiler TypeScript
npm start            # Lancer le serveur compilé
```

## 🚀 Déploiement

### Production

```bash
# 1. Build l'image Docker
docker build -t bicb-api:latest -f backend/Dockerfile backend/

# 2. Push vers le registre
docker push bicb-api:latest

# 3. Déploie sur Kubernetes (ou ton serveur)
kubectl apply -f k8s/
```

## 📊 Phases de Développement

### Phase 1 : MVP (Semaines 1-4)
- ✅ Setup infrastructure
- ✅ Authentification utilisateur
- ✅ Gestion des compétitions
- ✅ Système de votes
- ✅ Dashboard basique

### Phase 2 : Monétisation (Semaines 5-8)
- Intégration Stripe
- Système de paiement
- Wallet utilisateur
- Distribution des récompenses

### Phase 3 : Engagement (Semaines 9-12)
- Défis quotidiens
- Badges et achievements
- Leaderboards
- Notifications

### Phase 4 : Scalabilité (Semaines 13+)
- Optimisation performance
- CDN pour les assets
- Caching avancé
- Mobile app

## 🔐 Sécurité

- JWT tokens avec expiration
- Password hashing avec bcrypt
- Rate limiting
- CORS configuration
- HTTPS only
- Data encryption
- SQL injection prevention
- XSS prevention

## 📝 Contribution

1. Fork le repository
2. Crée une branche (`git checkout -b feature/amazing-feature`)
3. Commit tes changements (`git commit -m 'Add amazing feature'`)
4. Push vers la branche (`git push origin feature/amazing-feature`)
5. Ouvre une Pull Request

## 📄 Licence

Ce projet est sous licence MIT.

## 📧 Contact

Pour plus d'informations, contacte [iso123-sprik](https://github.com/iso123-sprik)

---

**Status**: 🚧 En développement
**Version**: 1.0.0-alpha
**Dernière mise à jour**: 2024
