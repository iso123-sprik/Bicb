# ARCHITECTURE - GameWorld Platform

## 📋 Vue d'ensemble du Projet

GameWorld est une **plateforme mondiale de compétitions multidisciplinaires** où les utilisateurs s'affrontent en solo ou en équipe (3-5 joueurs) dans des domaines variés : talents créatifs (chant, danse, art, photographie, cuisine) et jeux/esport (jeux vidéo, quiz, défis).

### Structure Globale

```
┌─────────────────────────────────────────────────────────────┐
│                       GAMEWORLD                             │
├─────────────────────────────────────────────────────────────┤
│  Plateforme mondiale de compétitions multidisciplinaires    │
│  - Solo ou Équipe (3-5 joueurs)                            │
│  - Talents créatifs + Jeux/Esport                          │
│  - Système de votes communauté + Pulse ⚡                  │
└─────────────────────────────────────────────────────────────┘
```

## 🎭 Rôles & Acteurs

### 1. **Compétiteurs (Solo)**
- Créent un profil personnel
- Paient les frais d'inscription
- Soumettent leurs performances (vidéo live, upload vidéo/photo/message)
- Progressent dans des tournois structurés
- Reçoivent des récompenses à chaque étape

### 2. **Compétiteurs (Équipe)**
- Forment des équipes de 3 à 5 joueurs
- Un capitaine crée l'équipe et invite les autres
- Chaque membre paie ses frais d'inscription
- L'équipe est confirmée avec minimum 3 membres
- Soumettent collectivement leurs performances
- Partagent les récompenses équitablement

### 3. **Supporters / Votants**
- Suivent les compétitions
- 1 vote gratuit par jour par compétition
- Dépensent des Pulse ⚡ pour votes/dons supplémentaires
- Écrivent des messages d'encouragement
- Rejoignent des "fan teams"
- Gagnent des badges + XP quand leur favori progresse

### 4. **Administrateurs (Équipe GameWorld)**
- Créent et gèrent les compétitions officielles
- Définissent les règles (solo, équipe, ou les deux)
- Vérifient les participants
- Valident les résultats
- Modèrent les contenus
- Contrôlent les phases du tournoi

## 🏆 Structure des Compétitions

### Phases du Tournoi (Style Ligue des Champions)
```
Qualifications
    ↓
Phase de groupes
    ↓
Huitièmes de finale
    ↓
Quarts de finale ⭐ (Retrait d'argent possible)
    ↓
Demi-finales ⭐ (Retrait d'argent possible)
    ↓
Finale ⭐ (Retrait d'argent possible)
```

### Calcul du Score
```
Score Final = 60% Performance (Jury) + 40% Votes Communauté

À chaque élimination/qualification:
- Badge reçu
- Pulse ⚡ distribué
- Argent (seulement Q4, SF, Final)
```

### 🔒 Restriction de Retrait d'Argent
**IMPORTANT**: Seuls les utilisateurs qui atteignent les **Quarts de Finale** jusqu'à la **Finale** peuvent retirer des fonds en argent réel. Les récompenses des phases antérieures sont en Pulse ⚡ uniquement.

## 💰 Système Monétaire - Pulse ⚡

### Wallet Utilisateur
- Solde en Pulse ⚡ (monnaie virtuelle)
- Solde en Argent réel (EUR, USD, etc.)
- Historique des transactions

### Utilisation de Pulse ⚡
- Votes supplémentaires (au-delà du 1 gratuit)
- Dons directs aux compétiteurs/équipes
- Achat d'avatars, badges, thèmes
- Support live

### Achat de Pulse ⚡
```
Utilisateur → Boutique → Sélectionne package
                   ↓
           Paiement réel (Stripe)
                   ↓
           Pulse crédité dans wallet
```

### Gains en Pulse & Argent
```
Compétiteur/Équipe:
- Participation: 0 EUR, Badges
- Phase de groupes: Pulse ⚡ seulement
- Huitièmes: Pulse ⚡ seulement
- Quarts: Pulse ⚡ + EUR (retrait possible)
- Demi-finales: Pulse ⚡ + EUR (retrait possible)
- Finale: Pulse ⚡ + EUR (retrait possible)

Supporter:
- Badges & XP quand favori progresse
- Pulse ⚡ via défis quotidiens
```

## 🔄 Flux Principaux

### 1️⃣ Inscription Solo
```
Création profil → Parcours compétitions → Sélection → Paiement → Confirmation → Accès manche active
```

### 2️⃣ Inscription Équipe
```
Création équipe ← (3-5 joueurs) → Invitation/Rejoindre
         ↓
    Chaque membre paie
         ↓
    Équipe confirmée (min 3 membres)
         ↓
    Accès manche active
```

### 3️⃣ Soumission de Performance
```
Compétiteur/Équipe → Soumet selon format
                    (live, vidéo, photo, texte)
                          ↓
                    Visible par communauté
                          ↓
                  Votes + Dons Pulse en temps réel
```

### 4️⃣ Progression dans Tournoi
```
Performance soumise
         ↓
Calcul score (60% jury + 40% votes)
         ↓
Classement établi
         ↓
Qualifiés → Prochaine phase
Éliminés → Récompense + Fin
```

### 5️⃣ Vote Supporter
```
Regarde performance
         ↓
    Vote gratuit (1/jour)
         ↓
Dépense Pulse pour votes supplémentaires/dons
         ↓
    Message d'encouragement
         ↓
    Fan team (optionnel)
         ↓
    Gagne badges & XP si favori progresse
```

### 6️⃣ Retrait d'Argent (Quarts, Demi, Final seulement)
```
Compétiteur récompensé à Q4+
         ↓
    Argent crédité dans wallet
         ↓
Demande de retrait (EUR, USD, etc.)
         ↓
Vérification anti-fraude
         ↓
    Paiement par Stripe/virement
```

## 📊 Classements

### Leaderboard Global des Compétiteurs
- Solo et équipes mélangées
- Score cumulé + récompenses
- Par saison / All-time
- Par domaine (gaming, chant, etc.)

### Leaderboard des Supporters
- Fidélité + engagement
- Nombre de votes
- Pulse dépensés
- Badges collectés

### Leaderboard des Équipes
- Performance collective
- Nombre de membres
- Récompenses cumulées

## 🎯 Engagement Quotidien

```
Connexion → Défis quotidiens
              ↓
         Missions spéciales
              ↓
         Récompense Pulse
              ↓
         Vérifier classement
              ↓
         Participer/voter
              ↓
         Notifications de manches
```

## 🛠️ Stack Technologique

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js / NestJS
- **Language**: TypeScript
- **Database**: PostgreSQL 16
- **Cache**: Redis 7
- **WebSocket**: Socket.io (temps réel)
- **Authentication**: JWT + OAuth2
- **Payments**: Stripe API
- **Storage**: AWS S3 (vidéos, photos)
- **Live Streaming**: HLS/RTMP (Twitch/YouTube)

### Frontend
- **Framework**: React 18 + TypeScript
- **Mobile**: React Native (optionnel)
- **State**: Redux / Context API
- **Styling**: Tailwind CSS
- **Real-time**: Socket.io client

### DevOps
- **Docker**: Containerization
- **Kubernetes**: Orchestration (optionnel)
- **CI/CD**: GitHub Actions

## 📦 Modèle de Données - Entités Principales

### Users
- id, email, password_hash, first_name, last_name
- avatar_url, bio, country, date_of_birth
- role (competitor, supporter, admin)
- pulse_balance, money_balance, total_earnings
- experience_points, level, badges
- created_at, updated_at

### Teams
- id, name, avatar_url, captain_id, status
- members (3-5), creation_date, competition_id
- is_open (ouvert aux rejoints), max_members
- total_earnings, is_active

### Competitions
- id, title, description, category
- status (draft, open, ongoing, completed)
- registration_fee, prize_pool
- format (solo, team, both)
- has_voting, start_date, end_date
- organizer_id (admin)

### CompetitionPhases
- id, competition_id, phase_number
- name (Qualifications, Phase de groupes, etc.)
- status, start_date, end_date
- scoring_formula (jury%, votes%)

### Participants (Solo ou Team)
- id, competition_id, user_id (NULL si équipe)
- team_id (NULL si solo)
- status (registered, qualified, eliminated)
- current_phase, total_score, ranking
- submission_url, submission_data

### Votes
- id, competition_id, participant_id, voter_id
- vote_type (free, pulse), pulse_spent
- created_at

### Rewards
- id, competition_id, participant_id
- ranking, phase
- pulse_amount, money_amount
- can_withdraw (TRUE seulement si phase >= Quarts)
- claimed, claimed_at

### Transactions
- id, user_id, type (pulse_buy, vote, donation, prize_claim, withdrawal)
- amount, currency, status
- payment_method, payment_id, metadata

### DailyChallenges
- id, title, description
- difficulty, reward_pulse, reward_xp
- active_date

### FanTeams
- id, participant_id, name
- members, created_at
- collective_points

## 🔒 Sécurité & Conformité

### Authentication
- JWT (1h expiration)
- Refresh tokens (90j)
- OAuth2 (Google, Apple optionnel)
- bcrypt password hashing

### Data Protection
- HTTPS only
- CORS restrictions
- RGPD compliance (politique de confidentialité)
- Vérification d'âge (18+ pour transactions réelles)
- Modération des contenus

### Rate Limiting
- Global: 100 req/min (users), 10 req/min (guests)
- Auth: 5 tentatives/15min
- Votes: 50/heure
- Donations: 100/jour
- Registrations: 1/compétition

### Fraud Prevention
- Anti-vote manipulation (1 vote/user/participant)
- Verification de paiements Stripe
- IP blocking after X failed attempts
- Transaction monitoring

## 🎨 Design System

### Couleurs
- **Fond**: #0A0A0F (noir profond)
- **Accent Principal**: #FFD700 (Pulse ⚡ jaune électrique)
- **Accent Secondaire**: #00E5FF (cyan)
- **Dégradé**: #6C2BD9 → #1A8FFF (violet → bleu)

### Typographie
- **Titres**: Sans-serif condensé bold (impact, prestige)
- **Corps**: Sans-serif léger et lisible

### Composants
- Angles arrondis: 8px
- Cartes: bordures lumineuses (glow 1px)
- Boutons: style pill
- Micro-animations: confetti au vote, compteurs temps réel

### Layout
- **Mobile**: Navigation par onglets en bas
- **Desktop**: Sidebar latéral
- Sections: Accueil, Compétitions, Live, Classement, Profil

## 📈 Phases de Développement

### Phase 1: MVP (Semaines 1-4)
- ✅ Setup infrastructure
- ✅ Authentification
- ✅ Gestion compétitions solo
- ✅ Système votes
- ✅ Dashboard basique

### Phase 2: Équipes & Monétisation (Semaines 5-8)
- Système d'équipes
- Intégration Stripe
- Wallet Pulse ⚡
- Distribution récompenses

### Phase 3: Engagement (Semaines 9-12)
- Défis quotidiens
- Badges & achievements
- Leaderboards
- Notifications temps réel

### Phase 4: Scalabilité & Polish (Semaines 13+)
- Optimisation performance
- Mobile app
- Live streaming
- Analytics

## 📞 Points d'Contact

**Repository**: https://github.com/iso123-sprik/Bicb
**Branch de développement**: `dev/technical-setup`
**Documentation**: `/docs` directory

### Documentations Détaillées
- **[DATABASE.md](docs/DATABASE.md)** - Schéma complet
- **[API.md](docs/API.md)** - Endpoints REST
- **[AUTHENTICATION.md](docs/AUTHENTICATION.md)** - Système auth JWT
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Architecture technique complète

---

**Status**: 🚧 En développement
**Version**: 1.0.0-alpha
**Dernière mise à jour**: 2026-07-06
