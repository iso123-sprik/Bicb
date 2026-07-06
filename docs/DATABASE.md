# Schéma de Base de Données - Bicb

## Vue d'ensemble

Schéma PostgreSQL pour la plateforme Bicb.

## Tables Principales

### 1. Users (Utilisateurs)

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
  role VARCHAR(50) DEFAULT 'user', -- 'user', 'organizer', 'admin'
  wallet_balance DECIMAL(15,2) DEFAULT 0.00,
  total_earnings DECIMAL(15,2) DEFAULT 0.00,
  total_spent DECIMAL(15,2) DEFAULT 0.00,
  experience_points INT DEFAULT 0,
  level INT DEFAULT 1,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login_at TIMESTAMP,
  verification_token VARCHAR(255),
  password_reset_token VARCHAR(255),
  password_reset_expires_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_country ON users(country);
```

### 2. Competitions (Compétitions)

```sql
CREATE TABLE competitions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organizer_id UUID NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(100) NOT NULL, -- 'virtual_sport', 'singing', 'dancing', 'photography', 'gaming', 'cooking', 'art', etc.
  image_url TEXT,
  banner_url TEXT,
  status VARCHAR(50) DEFAULT 'draft', -- 'draft', 'open', 'ongoing', 'completed', 'cancelled'
  max_participants INT,
  current_participants INT DEFAULT 0,
  registration_fee DECIMAL(10,2) NOT NULL,
  start_date TIMESTAMP NOT NULL,
  end_date TIMESTAMP NOT NULL,
  prize_pool DECIMAL(15,2) NOT NULL,
  rules TEXT,
  location VARCHAR(255),
  is_global BOOLEAN DEFAULT TRUE,
  has_voting BOOLEAN DEFAULT TRUE,
  voting_enabled_at TIMESTAMP,
  voting_ends_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_competitions_organizer ON competitions(organizer_id);
CREATE INDEX idx_competitions_status ON competitions(status);
CREATE INDEX idx_competitions_category ON competitions(category);
```

### 3. Competition Stages (Étapes des compétitions)

```sql
CREATE TABLE competition_stages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  stage_number INT NOT NULL,
  name VARCHAR(100) NOT NULL, -- 'Qualifications', 'Phase de groupes', 'Huitièmes', etc.
  description TEXT,
  status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'ongoing', 'completed'
  start_date TIMESTAMP NOT NULL,
  end_date TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(competition_id, stage_number)
);

CREATE INDEX idx_competition_stages_competition ON competition_stages(competition_id);
```

### 4. Participants (Participants aux compétitions)

```sql
CREATE TABLE participants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  status VARCHAR(50) DEFAULT 'registered', -- 'registered', 'qualified', 'eliminated', 'withdrawn'
  current_stage INT DEFAULT 1,
  total_votes INT DEFAULT 0,
  total_points INT DEFAULT 0,
  ranking INT,
  submission_url TEXT, -- URL vers la soumission (vidéo, image, etc.)
  submission_data JSONB, -- Données supplémentaires
  payment_status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'completed', 'failed'
  payment_transaction_id VARCHAR(255),
  registered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  eliminated_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(competition_id, user_id)
);

CREATE INDEX idx_participants_competition ON participants(competition_id);
CREATE INDEX idx_participants_user ON participants(user_id);
CREATE INDEX idx_participants_status ON participants(status);
```

### 5. Votes

```sql
CREATE TABLE votes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  voter_id UUID NOT NULL REFERENCES users(id),
  vote_weight INT DEFAULT 1, -- Poids du vote (premium voters pourraient avoir plus)
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(competition_id, participant_id, voter_id) -- Un utilisateur ne peut voter qu'une fois par participant
);

CREATE INDEX idx_votes_competition ON votes(competition_id);
CREATE INDEX idx_votes_participant ON votes(participant_id);
CREATE INDEX idx_votes_voter ON votes(voter_id);
```

### 6. Rewards (Récompenses)

```sql
CREATE TABLE rewards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID NOT NULL REFERENCES competitions(id) ON DELETE CASCADE,
  participant_id UUID NOT NULL REFERENCES participants(id) ON DELETE CASCADE,
  ranking INT NOT NULL, -- Position finale
  prize_amount DECIMAL(15,2) NOT NULL,
  badge_id UUID REFERENCES badges(id),
  experience_points INT DEFAULT 0,
  claimed BOOLEAN DEFAULT FALSE,
  claimed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_rewards_competition ON rewards(competition_id);
CREATE INDEX idx_rewards_participant ON rewards(participant_id);
```

### 7. Badges (Badges)

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

### 8. User Badges

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

### 9. Transactions (Paiements et transferts)

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  type VARCHAR(50) NOT NULL, -- 'registration_fee', 'prize_claim', 'withdrawal', 'refund', 'premium_purchase'
  amount DECIMAL(15,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'EUR',
  status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'completed', 'failed', 'cancelled'
  payment_method VARCHAR(100), -- 'stripe', 'paypal', 'bank_transfer'
  payment_id VARCHAR(255),
  competition_id UUID REFERENCES competitions(id),
  description TEXT,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_transactions_competition ON transactions(competition_id);
```

### 10. Daily Challenges (Défis quotidiens)

```sql
CREATE TABLE daily_challenges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  reward_points INT DEFAULT 10,
  reward_xp INT DEFAULT 50,
  difficulty VARCHAR(50), -- 'easy', 'medium', 'hard'
  active_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_daily_challenges_active_date ON daily_challenges(active_date);
```

### 11. User Challenge Progress

```sql
CREATE TABLE user_challenge_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  challenge_id UUID NOT NULL REFERENCES daily_challenges(id),
  completed BOOLEAN DEFAULT FALSE,
  completed_at TIMESTAMP,
  UNIQUE(user_id, challenge_id)
);

CREATE INDEX idx_user_challenge_progress_user ON user_challenge_progress(user_id);
```

### 12. Leaderboards (Classements)

```sql
CREATE TABLE leaderboards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competition_id UUID REFERENCES competitions(id) ON DELETE CASCADE,
  period VARCHAR(50) NOT NULL, -- 'daily', 'weekly', 'monthly', 'all_time', 'season'
  user_id UUID NOT NULL REFERENCES users(id),
  ranking INT NOT NULL,
  score INT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_leaderboards_competition_period ON leaderboards(competition_id, period);
CREATE INDEX idx_leaderboards_user ON leaderboards(user_id);
```

### 13. Social Impact Fund (Fonds d'impact social)

```sql
CREATE TABLE social_impact_projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(100), -- 'education', 'health', 'sport', 'environment', etc.
  target_amount DECIMAL(15,2) NOT NULL,
  current_amount DECIMAL(15,2) DEFAULT 0.00,
  partner_organization VARCHAR(255),
  status VARCHAR(50) DEFAULT 'active', -- 'active', 'completed', 'paused', 'cancelled'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Migrations

Toutes les tables doivent être créées via des migrations TypeORM ou Sequelize pour le versioning.

## Indexes et Performance

- Index sur les clés étrangères principales
- Index sur les colonnes de filtrage fréquentes (status, category)
- Partitioning pour les tables volumineuses (votes, transactions)
- Archive des anciennes compétitions
