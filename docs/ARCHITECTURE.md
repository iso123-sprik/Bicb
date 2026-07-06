# ARCHITECTURE TECHNIQUE COMPLÈTE - GameWorld

## 🏗️ Architecture Système

```
┌─────────────────────────────────────────────────────┐
│                  CLIENT LAYER                       │
├─────────────────────────────────────────────────────┤
│  Web (React 18)  │  Mobile (React Native)           │
│  WebSocket       │  Push Notifications              │
└────────────┬──────────────────────────────────────┘
             │
             │ HTTPS/WebSocket
             │
┌────────────▼──────────────────────────────────────┐
│              API GATEWAY LAYER                    │
├──────────────────────────────────────────────────┤
│  Rate Limiting  │  CORS  │  Request Validation   │
│  Load Balancer  │  JWT Verification              │
└────────────┬──────────────────────────────────────┘
             │
    ┌────────┼────────┬─────────────┐
    │        │        │             │
┌───▼──┐ ┌──▼──┐ ┌───▼────┐ ┌────▼────┐
│Auth  │ │User │ │Compet. │ │Payment  │
│Service│ │Svc  │ │Service │ │Service  │
└──────┘ └─────┘ └────────┘ └─────────┘
    │        │        │             │
    └────────┼────────┴─────────────┘
             │
        ┌────▼────────┐
        │ CACHE LAYER │
        │   Redis 7   │
        └────┬────────┘
             │
   ┌─────────┼──────────┐
   │         │          │
┌──▼──────┐ │      ┌───▼────┐
│PostgreSQL  │      │AWS S3  │
│  (Primary) │      │(Files) │
└──────────┘ │      └────────┘
   │         │
┌──▼──────┐  │
│PostgreSQL  │
│(Replicas) │
└──────────┘
```

## 💾 Schéma de Base de Données

### Tables Principales

#### 1. **users**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  avatar_url TEXT,
  bio TEXT,
  country VARCHAR(2),
  date_of_birth DATE NOT NULL,
  email_verified BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  role VARCHAR(50) DEFAULT 'supporter', -- 'competitor', 'supporter', 'admin'
  
  -- Wallet
  pulse_balance DECIMAL(15,2) DEFAULT 0.00,
  money_balance DECIMAL(15,2) DEFAULT 0.00,
  total_earnings_pulse DECIMAL(15,2) DEFAULT 0.00,
  total_earnings_money DECIMAL(15,2) DEFAULT 0.00,
  total_spent DECIMAL(15,2) DEFAULT 0.00,
  
  -- Gamification
  experience_points INT DEFAULT 0,
  level INT DEFAULT 1,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login_at TIMESTAMP,
  verification_token VARCHAR(255),
  password_reset_token VARCHAR(255),
  password_reset_expires_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_country ON users(country);
CREATE INDEX idx_users_role ON users(role);
```

#### 2. **teams**
```sql
CREATE TABLE teams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  avatar_url TEXT,
  description TEXT,
  captain_id UUID NOT NULL REFERENCES users(id),
  
  -- Team info
  member_count INT DEFAULT 1,
  max_members INT DEFAULT 5,
  min_members INT DEFAULT 3,
  is_open BOOLEAN DEFAULT TRUE,
  
  -- Stats
  total_earnings_pulse DECIMAL(15,2) DEFAULT 0.00,
  total_earnings_money DECIMAL(15,2) DEFAULT 0.00,
  total_victories INT DEFAULT 0,
  
  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_teams_captain ON teams(captain_id);
CREATE INDEX idx_teams_is_open ON teams(is_open);
```

#### 3. **team_members**
```sql
CREATE TABLE team_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  join_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  is_captain BOOLEAN DEFAULT FALSE,
  
  UNIQUE(team_id, user_id)
);

CREATE INDEX idx_team_members_team ON team_members(team_id);
CREATE INDEX idx_team_members_user ON team_members(user_id);
```

#### 4. **competitions**
```sql
CREATE TABLE competitions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organizer_id UUID NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(100) NOT NULL,
  image_url TEXT,
  banner_url TEXT,
  
  -- Status & Dates
  status VARCHAR(50) DEFAULT 'draft',
  start_date TIMESTAMP NOT NULL,
  end_date TIMESTAMP NOT NULL,
  current_phase INT DEFAULT 0,
  
  -- Participants & Registration
  registration_fee DECIMAL(10,2) NOT NULL,
  max_participants INT,
  current_participants INT DEFAULT 0,
  format VARCHAR(50) NOT NULL, -- 'solo', 'team', 'both'
  
  -- Prize Pool & Rewards
  prize_pool DECIMAL(15,2) NOT NULL,
  pulse_distribution JSONB, -- {phase: amount}
  money_distribution JSONB, -- {phase: amount}
  
  -- Features
  has_voting BOOLEAN DEFAULT TRUE,
  voting_enabled_at TIMESTAMP,
  voting_ends_at TIMESTAMP,
  
  -- Metadata
  rules TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_competitions_organizer ON competitions(organizer_id);
CREATE INDEX idx_competitions_status ON competitions(status);
CREATE INDEX idx_competitions_category ON competitions(category);
```

#### 5. **competition_phases**
```sql
CREATE TABLE competition_phases (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  phase_number INT NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  status VARCHAR(50) DEFAULT 'pending',
  
  -- Dates
  start_date TIMESTAMP NOT NULL,
  end_date TIMESTAMP NOT NULL,
  
  -- Scoring
  jury_percentage INT DEFAULT 60,
  community_votes_percentage INT DEFAULT 40,
  
  -- Money withdrawal eligibility
  can_withdraw_money BOOLEAN DEFAULT FALSE, -- TRUE seulement à partir Quarts
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(competition_id, phase_number)
);

CREATE INDEX idx_phases_competition ON competition_phases(competition_id);
```

#### 6. **participants**
```sql
CREATE TABLE participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE, -- NULL si équipe
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE, -- NULL si solo
  
  -- Status & Progress
  status VARCHAR(50) DEFAULT 'registered',
  current_phase INT DEFAULT 1,
  total_score DECIMAL(10,2) DEFAULT 0,
  jury_score DECIMAL(10,2) DEFAULT 0,
  community_score DECIMAL(10,2) DEFAULT 0,
  final_ranking INT,
  
  -- Submissions
  submission_url TEXT,
  submission_data JSONB,
  submitted_at TIMESTAMP,
  
  -- Payment
  payment_status VARCHAR(50) DEFAULT 'pending',
  payment_transaction_id VARCHAR(255),
  paid_at TIMESTAMP,
  
  -- Dates
  registered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  eliminated_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(competition_id, user_id) WHERE user_id IS NOT NULL,
  UNIQUE(competition_id, team_id) WHERE team_id IS NOT NULL
);

CREATE INDEX idx_participants_competition ON participants(competition_id);
CREATE INDEX idx_participants_user ON participants(user_id);
CREATE INDEX idx_participants_team ON participants(team_id);
CREATE INDEX idx_participants_status ON participants(status);
```

#### 7. **votes**
```sql
CREATE TABLE votes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  voter_id UUID NOT NULL REFERENCES users(id),
  
  -- Vote type
  vote_type VARCHAR(50) DEFAULT 'free', -- 'free', 'pulse'
  pulse_spent INT DEFAULT 0,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(competition_id, participant_id, voter_id) -- Un vote par participant par utilisateur
);

CREATE INDEX idx_votes_competition ON votes(competition_id);
CREATE INDEX idx_votes_participant ON votes(participant_id);
CREATE INDEX idx_votes_voter ON votes(voter_id);
```

#### 8. **donations**
```sql
CREATE TABLE donations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  donor_id UUID NOT NULL REFERENCES users(id),
  
  -- Donation
  pulse_amount DECIMAL(15,2) NOT NULL,
  message TEXT,
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_donations_competition ON donations(competition_id);
CREATE INDEX idx_donations_participant ON donations(participant_id);
CREATE INDEX idx_donations_donor ON donations(donor_id);
```

#### 9. **rewards**
```sql
CREATE TABLE rewards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  phase INT NOT NULL,
  
  -- Rewards
  ranking INT NOT NULL,
  pulse_amount DECIMAL(15,2) DEFAULT 0.00,
  money_amount DECIMAL(15,2) DEFAULT 0.00,
  badge_id UUID REFERENCES badges(id),
  experience_points INT DEFAULT 0,
  
  -- Withdrawal eligibility
  can_withdraw BOOLEAN DEFAULT FALSE, -- TRUE si phase >= Quarts
  
  -- Status
  claimed BOOLEAN DEFAULT FALSE,
  claimed_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_rewards_competition ON rewards(competition_id);
CREATE INDEX idx_rewards_participant ON rewards(participant_id);
CREATE INDEX idx_rewards_can_withdraw ON rewards(can_withdraw);
```

#### 10. **transactions**
```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  
  -- Transaction type
  type VARCHAR(50) NOT NULL, -- 'pulse_purchase', 'vote', 'donation', 
                             -- 'reward_claim', 'withdrawal', 'conversion'
  amount DECIMAL(15,2) NOT NULL,
  currency VARCHAR(10) NOT NULL, -- 'pulse', 'EUR', 'USD', etc.
  
  -- Conversion
  converted_from VARCHAR(10), -- Pour les conversions pulse → argent
  conversion_rate DECIMAL(10,6),
  
  -- Status
  status VARCHAR(50) DEFAULT 'pending',
  
  -- References
  competition_id UUID REFERENCES competitions(id),
  reward_id UUID REFERENCES rewards(id),
  
  -- Payment details
  payment_method VARCHAR(100), -- 'stripe', 'bank_transfer'
  payment_id VARCHAR(255),
  payment_status VARCHAR(50),
  
  -- Metadata
  description TEXT,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_type ON transactions(type);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_transactions_created ON transactions(created_at);
```

#### 11. **daily_challenges**
```sql
CREATE TABLE daily_challenges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  difficulty VARCHAR(50), -- 'easy', 'medium', 'hard'
  reward_pulse INT DEFAULT 10,
  reward_xp INT DEFAULT 50,
  active_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_daily_challenges_active_date ON daily_challenges(active_date);
```

#### 12. **user_challenge_progress**
```sql
CREATE TABLE user_challenge_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  challenge_id UUID NOT NULL REFERENCES daily_challenges(id),
  completed BOOLEAN DEFAULT FALSE,
  completed_at TIMESTAMP,
  
  UNIQUE(user_id, challenge_id)
);

CREATE INDEX idx_user_challenge_user ON user_challenge_progress(user_id);
```

#### 13. **badges**
```sql
CREATE TABLE badges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  description TEXT,
  icon_url TEXT,
  rarity VARCHAR(50), -- 'common', 'uncommon', 'rare', 'epic', 'legendary'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 14. **user_badges**
```sql
CREATE TABLE user_badges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  badge_id UUID NOT NULL REFERENCES badges(id),
  earned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, badge_id)
);

CREATE INDEX idx_user_badges_user ON user_badges(user_id);
```

#### 15. **fan_teams**
```sql
CREATE TABLE fan_teams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  member_count INT DEFAULT 0,
  collective_points INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_fan_teams_participant ON fan_teams(participant_id);
```

#### 16. **fan_team_members**
```sql
CREATE TABLE fan_team_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  fan_team_id UUID NOT NULL REFERENCES fan_teams(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(fan_team_id, user_id)
);

CREATE INDEX idx_fan_team_members_team ON fan_team_members(fan_team_id);
CREATE INDEX idx_fan_team_members_user ON fan_team_members(user_id);
```

## 💰 Système de Pulse & Conversion

### Règles de Conversion Pulse → Argent

```
1. CONDITION: Utilisateur doit avoir >= 50,000 Pulse
2. PHASE: Doit être en phase Quarts, Demi-finales ou Finale
3. PROCESS:
   - Utilisateur demande conversion
   - Validation du solde
   - Taux de conversion: 1 Pulse = 0.01 EUR (configurable)
   - Exemple: 50,000 Pulse = 500 EUR
   - Retrait par virement bancaire ou Stripe Payout

4. FEES: (optionnel)
   - 2-3% frais de conversion
   - Minimum de conversion: 50,000 Pulse (500 EUR)

5. PROCESS WITHDRAWAL:
   Utilisateur → Demande conversion/retrait
        ↓
   Validation solde (>= 50,000 Pulse)
        ↓
   Validation d'éligibilité (phase Q4+)
        ↓
   Conversion Pulse → EUR/USD
        ↓
   Paiement Stripe/Virement
        ↓
   Confirmation & MAJ wallet
```

### Pricing Pulse ⚡

```
Package 1: 100 Pulse = 1 EUR
Package 2: 550 Pulse = 5 EUR (10% bonus)
Package 3: 1,100 Pulse = 10 EUR (10% bonus)
Package 4: 5,500 Pulse = 50 EUR (10% bonus)
Package 5: 11,000 Pulse = 100 EUR (10% bonus)
Package 6: 55,000 Pulse = 500 EUR (10% bonus)
```

## 🔄 Flux des Transactions

### Achat de Pulse
```
Client clique "Acheter Pulse"
    ↓
Sélectionne package
    ↓
Redirigé vers Stripe Checkout
    ↓
Paiement validé
    ↓
Webhook Stripe → Backend
    ↓
Pulse crédité dans wallet
    ↓
Transaction enregistrée (type: pulse_purchase)
    ↓
Notification utilisateur
```

### Dépense de Pulse (Vote, Donation)
```
Utilisateur vote/donne
    ↓
Validation solde Pulse
    ↓
Débit du wallet
    ↓
Credit du participant
    ↓
Transaction enregistrée
    ↓
Notification temps réel
```

### Conversion Pulse → Argent (>= 50,000 Pulse)
```
Utilisateur accède à "Convertir Pulse"
    ↓
Validation: Solde >= 50,000 Pulse
    ↓
Validation: Phase >= Quarts OU Compétition terminée
    ↓
Calcul: 50,000 Pulse = 500 EUR (moins frais)
    ↓
Demande de retrait
    ↓
Paiement initié (Stripe Payout)
    ↓
Vérification anti-fraude
    ↓
Virement envoyé (1-5 jours ouvrables)
    ↓
Transaction enregistrée (type: conversion)
    ↓
Notification confirmation
```

## 🔐 Sécurité des Transactions

### Validation Retrait
```
✓ Utilisateur authentifié
✓ Email vérifié
✓ Solde Pulse >= 50,000
✓ Soit en phase Q4+, soit compétition terminée avec récompense
✓ Pas de retrait en cours
✓ Anti-fraude Stripe passée
✓ KYC (Know Your Customer) si montant > seuil
```

### Rate Limiting Retraits
```
- Maximum 1 retrait/jour par utilisateur
- Maximum 5 retraits/semaine par utilisateur
- Délai minimum 24h entre retraits
```

## 🎮 Service d'Engagement

### Défis Quotidiens
```
Daily Challenge 1: "Voter pour 3 participants" → 10 Pulse + 50 XP
Daily Challenge 2: "Regarder une performance complète" → 5 Pulse + 25 XP
Daily Challenge 3: "Laisser un message d'encouragement" → 5 Pulse + 25 XP

Bonus: Si 3/3 challenges complétés → +20 Pulse bonus
```

### Système de Niveaux
```
Lvl 1: 0 XP
Lvl 2: 500 XP
Lvl 3: 1,500 XP
Lvl 4: 3,000 XP
Lvl 5: 6,000 XP
...
Lvl 20+: Récompenses spéciales

Récompenses par niveau:
- Badge spécial
- +100 Pulse
- Accès early à certaines compétitions
```

## 📊 Analytics & Monitoring

### Métriques Clés
```
- Nombre de compétitions actives
- Nombre de participants (solo/équipe)
- Votes totaux par compétition
- Pulse économie (achetés, dépensés, convertis)
- Argent retiré (EUR, USD, etc.)
- Engagement rate (votes/jour)
- Retention rate (7j, 30j)
```

### Logs & Audit Trail
```
Toutes les transactions enregistrées:
- Timestamp exact
- User ID
- Action (vote, donation, withdrawal, etc.)
- Montant
- Statut
- IP address
```

## 🚀 Infrastructure DevOps

### Environments
```
Development (localhost:3000)
Staging (staging.gameworld.io)
Production (api.gameworld.io)
```

### CI/CD Pipeline
```
Git Push → GitHub Actions
    ↓
Linting & Tests
    ↓
Build Docker Image
    ↓
Push to Registry
    ↓
Deploy to Staging
    ↓
Smoke Tests
    ↓
Deploy to Production (manual approval)
```

### Monitoring
```
- Uptime monitoring (Pingdom)
- Error tracking (Sentry)
- Performance monitoring (New Relic)
- Logs aggregation (ELK Stack)
- Database performance (pgAdmin)
```

## 📱 WebSocket Events (Temps Réel)

### Events Participants
```
- participant.submitted (nouvelle soumission)
- participant.qualified (qualification)
- participant.eliminated (élimination)
- participant.ranked (changement de ranking)
```

### Events Communauté
```
- vote.received (vote reçu)
- donation.received (don reçu)
- comment.posted (nouveau commentaire)
- phase.complete (phase terminée)
```

### Events Utilisateur
```
- reward.claimed (récompense reçue)
- pulse.received (Pulse reçu)
- challenge.completed (défi complété)
- level.up (montée de niveau)
```

---

**Documentation Complète**: Voir `/docs` directory
**Status**: 🚧 En développement actif
**Dernière mise à jour**: 2026-07-06
