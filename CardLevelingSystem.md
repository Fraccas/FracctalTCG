# Card Leveling & XP System

Cards in the game earn experience points (XP) through gameplay and can level up over time. This document explains how XP is earned and how the leveling system works.

---

## How Cards Earn XP

XP is tracked per card type (not per individual card instance). All copies of a card share the same XP pool and level.

### 1. Playing Cards

All card types earn XP when played from hand.

| Mana Cost | XP Earned |
|-----------|-----------|
| 0 | 2 |
| 1 | 3 |
| 2 | 3 |
| 3 | 4 |
| 4 | 4 |
| 5 | 5 |
| 6 | 5 |
| 7 | 6 |
| 8+ | 6+ |

**Formula:** `2 + (mana_cost / 2)` rounded

**Note:** Summoned cards (created by effects) do not earn play XP.

---

### 2. Dealing Damage (Spells Only)

When a spell deals damage to a card, it earns XP equal to the damage dealt.

| Damage Dealt | XP Earned |
|--------------|-----------|
| 3 damage | 3 XP |
| 5 damage | 5 XP |
| 10 damage | 10 XP |

Additionally, all allied cards on board receive **50% of the damage XP** as shared XP.

---

### 3. Killing Enemy Cards

When a card kills an enemy, it earns XP based on the killed card's mana cost.

| Enemy Mana Cost | XP Earned |
|-----------------|-----------|
| 1 | 3 |
| 2 | 6 |
| 3 | 9 |
| 4 | 12 |
| 5 | 15 |
| 6 | 18 |

**Formula:** `enemy_mana_cost × 3`

All allied cards on board receive **50% of the kill XP** as shared XP.

---

### 4. End-of-Game Bonuses

#### Participation Bonus
Cards that were played during the match (now in discard pile) earn bonus XP:

| Result | XP per Card |
|--------|-------------|
| Winner | +5 XP |
| Loser | +2 XP |

#### Survival Bonus
Cards still on the board at game end earn survival XP:

| Card Type | XP Earned |
|-----------|-----------|
| Character | +5 XP |
| Artifact | +3 XP |

#### Winner's Board Bonus
If you win, all cards remaining on your board get an additional **+3 XP**.

---

## XP Summary by Card Type

### Characters
| Source | XP Range |
|--------|----------|
| Played | 2-6 |
| Kills | 3-18+ per kill |
| Ally Assists | 50% of ally kills |
| Survival | 5 |
| Win Bonus | 3 (if winner) |
| Participation | 5 (win) / 2 (loss) |

### Spells
| Source | XP Range |
|--------|----------|
| Played | 2-6 |
| Damage Dealt | = damage amount |
| Kills | 3-18+ per kill |
| Participation | 5 (win) / 2 (loss) |

### Artifacts
| Source | XP Range |
|--------|----------|
| Played | 2-6 |
| Ally Assists | 50% of ally actions |
| Survival | 3 |
| Win Bonus | 3 (if winner) |
| Participation | 5 (win) / 2 (loss) |

---

## Example Scenarios

### Aggressive Character (Wins Game)
A 4-mana character that kills 2 enemies and survives:
- Play XP: 4
- Kill XP: 12 + 15 = 27 (killed 4-mana and 5-mana enemies)
- Ally Assist: ~5 (from ally's kill)
- Survival: 5
- Win Bonus: 3
- **Total: ~44 XP**

### Utility Spell (No Damage)
A 3-mana buff spell in a winning game:
- Play XP: 4
- Participation: 5
- **Total: 9 XP**

### Damage Spell
A 2-mana spell that deals 4 damage and kills a 3-mana enemy:
- Play XP: 3
- Damage XP: 4
- Kill XP: 9
- Participation: 5
- **Total: 21 XP**

### Artifact (Survives to Victory)
A 3-mana artifact that stays on board:
- Play XP: 4
- Ally Assists: ~10 (from 2 ally kills)
- Survival: 3
- Win Bonus: 3
- Participation: 5
- **Total: ~25 XP**

---

## Level Thresholds

Card levels are determined by total accumulated XP. The XP thresholds are configured in `GameplayData` or the backend API.

Typical progression:
| Level | Total XP Required |
|-------|-------------------|
| 1 | 0 |
| 2 | 50 |
| 3 | 150 |
| 4 | 350 |
| 5 | 700 |
| ... | Exponential growth |

*Actual values may vary based on game configuration.*

---

## Technical Details

### Server-Side Tracking
XP is accumulated during gameplay and submitted to the server at match end via `ServerXPUploader`. This ensures XP is only granted for completed matches.

### Key Files
- `ServerXPUploader.cs` - Accumulates and submits XP to API
- `GameLogic.cs` - Contains XP granting logic
- `UserData.cs` - Stores card XP/level data per user

### API Endpoint
XP is submitted to: `POST /users/cards/submitmatchcardxp`

Request format:
```json
{
  "user_id": "player_user_id",
  "xp": [
    { "tid": "card_id_1", "amount": 25 },
    { "tid": "card_id_2", "amount": 12 }
  ]
}
```

---

## Design Philosophy

1. **All cards can level** - Not just damage-dealers
2. **Playing matters** - Base XP for using a card
3. **Winning matters more** - Higher bonuses for winners
4. **Survivors rewarded** - Incentive to protect your cards
5. **Team effort** - Ally assists share XP across your board
6. **Higher cost = more reward** - Expensive cards earn more per play
