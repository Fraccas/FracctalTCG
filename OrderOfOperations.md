# Order of Operations - Game Logic Timing Chart

This document explains the exact order in which game events resolve. Understanding timing is crucial for predicting outcomes in edge cases.

---

## Start of Turn Sequence

When a player's turn begins, the following happens **in this exact order**:

| Step | Event | Description |
|------|-------|-------------|
| 1 | **StartOfTurnPreDraw Abilities** | Abilities that trigger before the draw step |
| 2 | **Draw Card** | Active player draws their card for the turn |
| 3 | **Gain Mana** | Mana maximum increases, mana refills to max |
| 4 | **Player Poison/Burn** | Player takes damage from Poisoned/Burned status |
| 5 | **Hero Refreshes** | Hero power becomes available again |
| 6 | **Sleep Wake Check** | 33% chance for sleeping creatures to wake up |
| 7 | **Creatures Refresh** | Non-sleeping creatures become ready to act (unexhaust) |
| 8 | **Creature Poison/Burn** | Creatures take damage from Poisoned/Burned status |
| 9 | **Curse Effect** | Cursed creatures lose attack |
| 10 | **Mind Control Expiration** | Time-based mind control effects may expire |
| 11 | **Increment Turns on Board** | Each creature's turn counter increases |
| 12 | **Evolution Check** | Creatures that meet evolution requirements evolve |
| 13 | **Update Ongoing Effects** | All auras and ongoing abilities recalculate |
| 14 | **StartOfTurn Abilities** | Abilities that trigger at start of turn |
| 15 | **Main Phase Begins** | Player can now take actions |

### Important: Poison vs Evolution

**Q: If a creature has 1 HP, is poisoned, and is about to evolve - does it evolve or die?**

**A: It dies.** Poison damage (Step 8) happens **before** the evolution check (Step 12). The creature takes poison damage first, and if it dies, it never gets the chance to evolve.

---

## End of Turn Sequence

When the active player ends their turn:

| Step | Event | Description |
|------|-------|-------------|
| 1 | **Reduce Status Durations** | All timed status effects tick down by 1 |
| 2 | **End Turn Bleed Damage** | Creatures with EndTurnBleed take damage |
| 3 | **EndOfTurn Abilities** | Abilities that trigger at end of turn |
| 4 | **Stalemate Check** | Game checks for passive turn stalemate |
| 5 | **Next Turn Starts** | Turn passes to opponent |

---

## Combat Resolution

When a creature attacks another creature:

| Step | Event | Description |
|------|-------|-------------|
| 1 | **OnBeforeAttack** | Attacker's before-attack abilities trigger |
| 2 | **OnBeforeDefend** | Defender's before-defend abilities trigger |
| 3 | **Secret Check** | Trap cards may trigger and prevent the attack |
| 4 | **Attack Damage** | Attacker deals damage to defender |
| 5 | **Counter Damage** | Defender deals damage back (unless attacker has Intimidate) |
| 6 | **Attacker Exhausts** | Attacker becomes exhausted |
| 7 | **OnAfterAttack** | Attacker's after-attack abilities trigger |
| 8 | **OnAfterDefend** | Defender's after-defend abilities trigger |
| 9 | **Death Check** | Any creatures with 0 or less HP are destroyed |

### Combat Notes
- Damage is dealt **simultaneously** in steps 4-5
- A creature can kill its attacker even if it dies in the same combat
- Intimidate prevents counter damage but NOT OnAfterDefend triggers

---

## When Attacking a Player

| Step | Event | Description |
|------|-------|-------------|
| 1 | **Secret Check** | Trap cards may trigger and prevent the attack |
| 2 | **OnBeforeAttack** | Attacker's before-attack abilities trigger |
| 3 | **Deal Damage** | Player takes damage equal to attacker's attack |
| 4 | **Attacker Exhausts** | Attacker becomes exhausted |
| 5 | **OnAfterAttack** | Attacker's after-attack abilities trigger |

---

## Playing a Card

When a card is played from hand:

| Step | Event | Description |
|------|-------|-------------|
| 1 | **Pay Mana Cost** | Mana is spent |
| 2 | **Remove from Hand** | Card leaves the hand |
| 3 | **Enter Play Zone** | Card enters board/secret/discard as appropriate |
| 4 | **Exhaust** | Board cards become exhausted (unless they have Rush) |
| 5 | **Update Ongoing** | All auras recalculate with new card in play |
| 6 | **OnPlayOther Secrets** | Opponent's trap cards may trigger |
| 7 | **OnPlay Abilities** | The played card's OnPlay abilities trigger |
| 8 | **OnPlayOther Abilities** | Other cards react to the new card being played |

### Playing Cards Notes
- **Rush** creatures enter play ready to attack creatures (but not players on first turn)
- **Haste** creatures can attack players immediately
- Trap cards can potentially counter/cancel OnPlay effects

---

## Ability Resolution Order

When multiple abilities trigger at the same time:

1. **Active player's abilities resolve first**
2. **Abilities on the same side resolve in board order** (left to right)
3. **Chain abilities** from a single effect resolve in the order listed
4. **Secrets/Traps** resolve before regular abilities of the same timing

---

## Status Effect Priority

### Negative (Curable) Status Effects
- Silenced - Abilities disabled
- Paralysed - Cannot act
- Poisoned - Takes damage at start of turn
- Sleep - Doesn't refresh, 33% wake chance
- Frozen - Cannot act for 1 turn
- Cursed - Loses attack at start of turn
- Burned - Takes damage at start of turn
- EndTurnBleed - Takes damage at end of turn

### Positive Status Effects
- Stealth - Cannot be targeted until acting
- Invincibility - Cannot be attacked
- Shell - Blocks first damage instance
- Armor - Reduces incoming damage
- Fury - Can attack twice
- Intimidate - Prevents counter-attacks
- Flying - Ignores taunt
- And more...

---

## Common Timing Questions

### Q: If two creatures would kill each other, who wins?
**A:** Neither. Combat damage is simultaneous, so both die.

### Q: Can I respond to my opponent playing a card?
**A:** Not directly, but your Secrets/Traps will automatically trigger if their conditions are met.

### Q: When do ongoing/aura effects update?
**A:** After any game state change - playing cards, combat, abilities resolving, start/end of turn.

### Q: If I silence a creature with an ongoing effect, does it stop immediately?
**A:** Yes. Ongoing effects recalculate immediately after silence is applied.

### Q: Does Rush let me attack face on the first turn?
**A:** No. Rush only allows attacking creatures on the first turn. You need Haste to attack players immediately.

---

## Quick Reference: Ability Triggers

| Trigger | When It Fires |
|---------|---------------|
| OnPlay | When this card is played from hand |
| OnPlayOther | When another card is played |
| OnSummon | When this card is summoned (by an effect) |
| StartOfTurn | At the start of your turn |
| StartOfTurnPreDraw | Before you draw your card |
| EndOfTurn | At the end of your turn |
| OnBeforeAttack | Just before this creature attacks |
| OnAfterAttack | After this creature attacks |
| OnBeforeDefend | Just before this creature is attacked |
| OnAfterDefend | After this creature is attacked |
| OnDeath | When this creature dies |
| OnKill | When this creature kills another |
| Activate | When manually activated (hero powers, etc.) |
| Ongoing | Constantly active while in play |
