# PRIMAL TROVE - GAME DESIGN DOCUMENT
**Version:** 3.0
**Last Updated:** January 2026
**Platform:** Web (Mobile-First, Portrait Only)
**Genre:** Idle/Passive Income Management

---

## 1. GAME OVERVIEW

**Primal Trove** is a passive gem income management system where players earn gems through automated production (Gem Mine) and time-locked investment deposits (Treasure Trove). Players progress by upgrading their facility level, which increases production rates, storage capacity, investment caps, and deposit interest bonuses.

### Core Pillars
1. **Passive Income**: Gems accumulate automatically over time
2. **Active Collection**: Players must manually collect all rewards
3. **Strategic Investment**: Lock gems for time-based returns
4. **Progression**: 10 levels with escalating benefits

### Future Features
- **Upgrades Tab**: Non-linear time-based progression system (Coming Soon)

---

## 2. CORE LOOP

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  1. GEM MINE PRODUCES GEMS                     │
│     (passive, real-time accumulation)          │
│                 ↓                              │
│  2. PLAYER COLLECTS GEMS                       │
│     (manual tap, adds to gem_balance)          │
│                 ↓                              │
│  3. PLAYER INVESTS GEMS                        │
│     (time-locked deposits)                     │
│                 ↓                              │
│  4. WAIT FOR DEPOSIT TO COMPLETE               │
│     (7/14/30/60 days real-time)               │
│                 ↓                              │
│  5. COLLECT DEPOSIT PAYOUT                     │
│     (principal + interest)                     │
│                 ↓                              │
│  6. UPGRADE LEVEL                              │
│     (spend gems, unlock better rates)          │
│                 ↓                              │
│  LOOP BACK TO STEP 1 (with better rates)       │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Session Flow
```
Player Opens App
    ↓
View Hero Banner ("Primal Trove")
    ↓
Select Tab (Gem Mine | Investments | Upgrades)
    ↓
[GEM MINE TAB]
    Check Storage Status
    ↓
    IF storage >= 1 gem:
        → Collect Button: Pulsing + Sparkle
        → Tab Badge shows when on different tab
    ↓
    Tap Collect → Particles Spawn → Gems Added to Balance
    ↓
    Check if can upgrade level
        IF yes: Tap Upgrade → Animation → Level Up

[INVESTMENTS TAB]
    View 4 Deposit Cards (7/14/30/60-day)
    ↓
    IF locked:
        → Show lock icon + level requirement
        → Tap shows tooltip
    IF available:
        → Tap to open detail view
        → Adjust investment slider
        → Confirm deposit
        → Gems locked for duration
    IF completed:
        → Card pulses + badge on tab
        → Tap to collect payout + particles

Player Leaves → Progress Persists → Return Later
```

---

## 3. FEATURES & SYSTEMS

### 3.1 GEM MINE (Passive Production)

**Description**: Automatically produces gems at a rate determined by player_level. Gems accumulate in storage until manually collected.

**Behavior**:
- Production runs continuously (every 100ms tick)
- Storage fills at rate: `1 gem per gem_production_interval_in_seconds`
- Storage capped at `max_storage_capacity_for_level`
- Production continues even when storage is full (excess is lost)
- Manual collection required to add gems to player_gem_balance

**Visual States**:
- **Empty** (storage < 1): Collect button disabled, gray
- **Has Gems** (storage >= 1): Collect button enabled, pulsing animation, sparkle icon
- **Full** (storage >= max_storage_capacity_for_level): Glowing text effect

**Actions**:
| Action | Trigger | Requirements | Result | Side Effects |
|--------|---------|--------------|--------|--------------|
| collect_mine_gems | Player taps "Collect" button | current_mine_storage >= 1 | `player_gem_balance += floor(current_mine_storage)`<br>`current_mine_storage = 0`<br>`last_collection_timestamp = current_timestamp` | Spawn particle_count_mine particles at button position |

**Formulas**:
```
gems_to_add = floor(current_mine_storage)
time_until_full_in_seconds = (max_storage_capacity_for_level - current_mine_storage) * gem_production_interval_in_seconds
```

**Display**:
- Title: "Gem Production: X/Y" where X = floor(current_mine_storage), Y = max_storage_capacity_for_level
- Rate: "Rate: 1 Gem every [formatted time]"
- Timer: "Time until full: Xd Yh Zm Ws"

---

### 3.2 TREASURE TROVE (Time-Locked Deposits)

**Description**: Players lock gems for fixed durations to earn interest-based returns. Four deposit tiers with increasing durations and returns.

**Deposit Tiers**:
| Tier ID | Display Name | Duration (days) | Base Interest | Unlock Level | Icon |
|---------|--------------|-----------------|---------------|--------------|------|
| 7day | 7-Day Deposit | deposit_duration_7_day | deposit_base_interest_7_day | deposit_unlock_level_7_day | ⛰️ |
| 14day | 14-Day Deposit | deposit_duration_14_day | deposit_base_interest_14_day | deposit_unlock_level_14_day | 🏔️ |
| 30day | 30-Day Deposit | deposit_duration_30_day | deposit_base_interest_30_day | deposit_unlock_level_30_day | 🗻 |
| 60day | 60-Day Deposit | deposit_duration_60_day | deposit_base_interest_60_day | deposit_unlock_level_60_day | ⚡ |

**Card States**:
- **Locked**: player_level < deposit_unlock_level → Gray overlay, lock icon, tooltip on tap
- **Available**: Can invest → White background, "Invest" button
- **Active**: Investment in progress → Purple border, countdown timer, progress bar
- **Completed**: Ready to collect → Green pulsing border, "Collect!" text, tab badge

**Investment Flow**:
```
Player taps available deposit card
    ↓
Detail screen opens
    ↓
Shows:
    - Deposit time (X days)
    - Base interest (Y%)
    - Level bonus (Z%)
    - Total interest ((Y+Z)%)
    - Max investment (max_investment_cap_for_level)
    ↓
Player adjusts slider (10 gem increments)
    ↓
Shows expected payout = floor(investment_amount * (1 + total_interest_rate))
    ↓
Player taps "Deposit"
    ↓
Confirmation modal appears with warning
    ↓
Player taps "Confirm"
    ↓
Gems deducted, deposit starts, timer begins
```

**Actions**:

| Action | Trigger | Requirements | Result | Side Effects |
|--------|---------|--------------|--------|--------------|
| open_deposit_detail | Tap deposit card | player_level >= deposit_unlock_level<br>AND deposit not active | Show deposit_detail_screen | Set selected_deposit_id<br>Set initial_investment_amount = min(100, player_gem_balance, max_investment_cap_for_level) |
| tap_locked_deposit | Tap deposit card | player_level < deposit_unlock_level | Show tooltip | Display "Unlock at Level X" for tooltip_display_duration |
| adjust_investment_slider | Drag slider | In deposit detail screen | Update investment_amount | Recalculate expected_payout |
| confirm_deposit_start | Tap "Deposit" button | investment_amount >= minimum_investment_amount<br>AND investment_amount <= player_gem_balance<br>AND investment_amount <= max_investment_cap_for_level | Show confirmation_modal | None |
| execute_deposit | Tap "Confirm" in modal | Same as above | `player_gem_balance -= investment_amount`<br>Create active_deposit record:<br>`deposit_start_timestamp = current_timestamp`<br>`deposit_end_timestamp = current_timestamp + (deposit_duration_in_days * seconds_per_day)`<br>`deposit_principal = investment_amount`<br>`deposit_is_completed = false` | Close detail screen<br>Close modal |
| collect_completed_deposit | Tap completed deposit card | deposit_is_completed = true | Calculate: `total_interest = deposit_base_interest + level_deposit_bonus_for_level`<br>`payout = floor(deposit_principal * (1 + total_interest))`<br>`player_gem_balance += payout`<br>Delete active_deposit record | Spawn particle_count_deposit particles with deposit icon |

**Formulas**:
```
total_interest_rate = deposit_base_interest_for_tier + level_deposit_bonus_for_level
expected_payout = floor(investment_amount * (1 + total_interest_rate))
time_remaining_in_seconds = max(0, (deposit_end_timestamp - current_timestamp) / 1000)
deposit_progress_percentage = ((total_duration - time_remaining) / total_duration) * 100
```

**Constraints**:
- Players can have ONE active deposit per tier (4 total simultaneous)
- Investment must be: `minimum_investment_amount <= investment_amount <= min(player_gem_balance, max_investment_cap_for_level)`
- Investment increments: `investment_slider_step_size` (10 gems)
- Deposits CANNOT be cancelled or withdrawn early
- Interest is calculated and paid at collection time (not accrued during)

---

### 3.3 LEVEL PROGRESSION

**Description**: 10-level system where each upgrade improves all game systems.

**Level Benefits**:
| Benefit | Description | Effect |
|---------|-------------|--------|
| Storage Capacity | Max gems mine can hold | Increases from level_1_max_storage to level_10_max_storage |
| Production Speed | Time to produce 1 gem | Decreases from level_1_production_time to level_10_production_time |
| Deposit Bonus | Added to all deposit interest rates | Increases from level_1_deposit_bonus to level_10_deposit_bonus |
| Investment Cap | Max gems can invest per deposit | Increases from level_1_investment_cap to level_10_investment_cap |
| Tier Unlocks | New deposit tiers become available | See unlock_level_for_each_tier |

**Upgrade Flow**:
```
Check requirements:
    player_level < max_player_level (10)
    AND
    player_gem_balance >= level_upgrade_cost_for_current_level
        ↓
Player taps "Upgrade" button
        ↓
Deduct: player_gem_balance -= level_upgrade_cost_for_current_level
        ↓
Increment: player_level += 1
        ↓
Play level_up_animation (2 second duration)
        ↓
Spawn particle_count_level_up star particles
        ↓
Update all systems with new level values
```

**Actions**:
| Action | Trigger | Requirements | Result | Side Effects |
|--------|---------|--------------|--------|--------------|
| upgrade_level | Tap "Upgrade" button | player_level < max_player_level<br>AND player_gem_balance >= level_upgrade_cost_for_current_level | `player_gem_balance -= level_upgrade_cost_for_current_level`<br>`player_level += 1` | Show level_up_overlay<br>Spawn star particles<br>Auto-hide overlay after level_up_animation_duration<br>Trigger save_to_server |
| view_level_info | Tap info (i) icon on level badge | None | Open level_info_modal | Scroll to current level<br>Highlight current level row |

**Level Up Animation**:
- Full-screen radial gradient overlay
- "LEVEL X!" text with bounce + rotation
- 20 star (⭐) particles spawn at center
- Duration: level_up_animation_duration
- Z-index: Above all other UI

---

### 3.4 TAB NAVIGATION

**Description**: Three tabs switch content area while maintaining hero banner and bottom bar.

**Tabs**:
| Tab ID | Label | State | Behavior |
|--------|-------|-------|----------|
| mine | "Gem Mine" | Default active | Shows production system |
| investments | "Investments" | Available | Shows 4 deposit cards |
| upgrades | "Upgrades" | Locked | Shows "Coming Soon" message + lock icon |

**Tab Badges**:
Badges appear on inactive tabs to indicate collectible items:
- **Gem Mine Badge**: Shows when `active_tab != 'mine' AND current_mine_storage >= 1`
- **Investments Badge**: Shows when `active_tab != 'investments' AND any_deposit_is_completed = true`
- Badge displays pulsing red circle with "!" icon

**Actions**:
| Action | Trigger | Requirements | Result | Side Effects |
|--------|---------|--------------|--------|--------------|
| switch_to_mine_tab | Tap "Gem Mine" button | None | `active_tab = 'mine'` | Hide other tab content<br>Show mine content<br>Remove badge from mine tab |
| switch_to_investments_tab | Tap "Investments" button | None | `active_tab = 'investments'` | Hide other tab content<br>Show investments content<br>Remove badge from investments tab |
| tap_upgrades_tab | Tap "Upgrades" button | None | `active_tab = 'upgrades'` | Show "Coming Soon" tooltip<br>Display locked message screen |

---

### 3.5 VISUAL FEEDBACK SYSTEM

#### 3.5.1 Collection Ready Indicators

**Pulsing Animation** (CSS: collectPulse):
- Triggers when: collectible item available
- Effect: Scale 1.0 → 1.05, box-shadow expands
- Duration: 1.5 seconds, infinite loop
- Applied to: Collect buttons with gems available

**Sparkle Icon** (✨):
- Appears: Top-right of collect buttons
- Animation: Opacity 0 → 1, rotation 0° → 180°, scale 0.5 → 1
- Duration: 2 seconds, infinite loop

#### 3.5.2 Particle System

**Spawn Conditions**:
| Event | Particle Type | Count | Spawn Origin |
|-------|---------------|-------|--------------|
| collect_mine_gems | 💎 (gem emoji) | particle_count_mine | Center of collect button |
| collect_completed_deposit | Deposit tier icon | particle_count_deposit | Center of deposit card |
| upgrade_level | ⭐ (star emoji) | particle_count_level_up | Center of screen |

**Particle Behavior**:
- Each particle has unique ID (incremented from particle_id_counter)
- Random horizontal offset: ±particle_spread_horizontal
- Random vertical offset: ±particle_spread_vertical
- Animation: Float upward particle_float_distance over particle_animation_duration
- Transforms: Opacity 1 → 0, Scale 1 → 0.3, Rotation 0° → 360°
- Auto-cleanup after particle_animation_duration

**Formulas**:
```
particle_x_position = button_center_x + (random() - 0.5) * particle_spread_horizontal
particle_y_position = button_center_y + (random() - 0.5) * particle_spread_vertical
```

#### 3.5.3 Level Up Animation

**Overlay**:
- Full-screen radial gradient: Purple (center) → Black (edges)
- Fade: 0 → 1 (30% mark) → 1 (70% mark) → 0 (100%)
- Duration: level_up_animation_duration

**Text**:
- Content: "LEVEL {player_level}!"
- Animation: Scale 0 → 1.2 → 0.9 → 1, Rotation -180° → 10° → -5° → 0°
- Text shadow: Purple glow
- Duration: level_up_animation_duration

---

## 4. UI/UX SPECIFICATIONS

### 4.1 Layout Structure

```
┌─────────────────────────────────────────┐
│  TOP BAR (Sticky)                       │
│  [💎 X,XXX gems]        [Level Y (i)]  │
├─────────────────────────────────────────┤
│                                         │
│  HERO BANNER                            │
│  "Primal Trove"                         │
│  [Image Placeholder]                    │
│                                         │
├─────────────────────────────────────────┤
│  TAB NAVIGATION                         │
│  [Gem Mine(!)] [Investments(!)] [🔒]   │
├─────────────────────────────────────────┤
│                                         │
│  SCROLLABLE CONTENT AREA                │
│  (Tab-specific content)                 │
│                                         │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│  BOTTOM BAR (Fixed)                     │
│  [Icon] [Icon] [🏕️Camping] [Icon] [⚙️] │
└─────────────────────────────────────────┘
```

**Container**:
- Max width: viewport_max_width_pixels
- Centered on desktop with black sidebars
- Mobile: Full width, portrait only
- No landscape support

### 4.2 Color Palette (Wireframe)

| Element | Color | Hex | Usage |
|---------|-------|-----|-------|
| Background | Black | #0a0a0a | Main background |
| Cards | Light Gray | #f5f5f5 | Content containers |
| Borders | Gray | #c0c0c0 | Card outlines |
| Text Primary | Black | #000000 | Main text |
| Text Secondary | Gray | #666666 | Descriptive text |
| Accent Active | Purple | #7c3aed | Active tabs, progress |
| Accent Success | Green | #10b981 | Completed deposits, payouts |
| Accent Alert | Red | #ef4444 | Badges, warnings |

### 4.3 Typography

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Hero Title | 32px | Bold | Black |
| Tab Labels | 14px | Semi-bold | Variable (active/inactive) |
| Section Headers | 18px | Bold | Black |
| Body Text | 14px | Regular | Gray |
| Numbers | 16-24px | Bold | Black |
| Button Text | 16px | Semi-bold | Variable |

### 4.4 Interactive Elements

**Buttons**:
- Border radius: button_border_radius
- Padding: 12-16px vertical
- States: Default, Active (scale 0.98), Disabled (opacity 0.5)
- Pill-shaped for primary actions

**Sliders**:
- Track height: slider_track_height
- Thumb size: slider_thumb_size
- Color: Black thumb, gray track
- Step increments: investment_slider_step_size

**Cards**:
- Border: card_border_width solid
- Border radius: card_border_radius
- Shadow: Subtle for depth
- Hover: Border color change (desktop only)

### 4.5 Responsive Behavior

**Portrait Only**:
- Viewport locked: No rotation
- Scale disabled: No pinch-zoom
- Fixed layout: No horizontal scroll

**Breakpoints**: None (single layout for all devices)

---

## 5. PROGRESSION SYSTEM

### 5.1 Player Journey

**Early Game (Levels 1-3)**:
- Players learn basic loop: produce → collect → invest
- Only 7-day deposit available after Level 3
- Focus: Understanding mechanics, seeing first returns
- Bottleneck: Low storage capacity, slow production

**Mid Game (Levels 4-7)**:
- More deposit options unlock (14-day at L5, 30-day at L7)
- Investment caps increase significantly
- Players optimize timing between collection and investment
- Bottleneck: Upgrade costs increase, need strategic saving

**Late Game (Levels 8-10)**:
- All deposits unlocked (60-day at L9)
- High investment caps enable large deposits
- Deposit bonuses make returns very profitable
- Bottleneck: Long deposit durations, opportunity cost

### 5.2 Unlock Sequence

| Level | What Unlocks | Significance |
|-------|--------------|--------------|
| 1 | Starting state | Mine only, learn production |
| 3 | 7-Day Deposit | First investment option |
| 5 | 14-Day Deposit | Mid-range commitment |
| 7 | 30-Day Deposit | Long-term investment |
| 9 | 60-Day Deposit | Maximum duration, highest returns |
| 10 | Max Level | Peak efficiency, no more upgrades |

### 5.3 Time Investment Analysis

**To Max Level (1→10)**:
Total gems needed: SUM(level_upgrade_cost_1 through level_upgrade_cost_9)

**Example Session**:
```
Day 1:
  - Start Level 1
  - Collect mine every 5 hours
  - Save for Level 2 upgrade (500 gems)
  - Time to Level 2: ~25 collections = ~125 hours (5 days)

After Level 3:
  - Unlock 7-day deposits
  - First deposit completes after 7 real days
  - Returns accelerate progression

Long-term:
  - Strategic deposit use can multiply gem income
  - But requires patience (up to 60 days)
```

---

## 6. ECONOMY & BALANCE

### 6.1 Income Sources

| Source | Rate | Notes |
|--------|------|-------|
| Gem Mine | Continuous passive | Capped by storage, requires collection |
| 7-Day Deposits | deposit_base_interest_7_day + level bonus | Lowest risk/reward |
| 14-Day Deposits | deposit_base_interest_14_day + level bonus | Medium risk/reward |
| 30-Day Deposits | deposit_base_interest_30_day + level bonus | High risk/reward |
| 60-Day Deposits | deposit_base_interest_60_day + level bonus | Maximum risk/reward |

### 6.2 Sinks

| Sink | Cost | Frequency |
|------|------|-----------|
| Level Upgrades | level_upgrade_cost_for_level | One-time per level |
| Deposit Investments | Player choice | As often as player chooses |

### 6.3 Economic Formulas

**Mine Lifetime Value** (theoretical max if collected instantly):
```
gems_per_hour_at_level = 3600 / gem_production_interval_in_seconds_for_level
daily_mine_income = gems_per_hour_at_level * 24
```

**Deposit ROI**:
```
deposit_roi = (expected_payout - investment_amount) / investment_amount
deposit_roi_percentage = deposit_roi * 100
deposit_daily_rate = deposit_roi / deposit_duration_in_days
```

**Break-Even Analysis**:
```
time_to_recover_upgrade_cost = level_upgrade_cost / daily_mine_income_before_upgrade
value_if_invested_instead = level_upgrade_cost * (1 + best_available_interest_rate)
```

### 6.4 Balance Levers

To adjust difficulty/pacing:
- **Speed up**: Decrease gem_production_interval_in_seconds
- **Slow down**: Increase level_upgrade_cost
- **Increase rewards**: Increase deposit_base_interest
- **Reduce rewards**: Decrease level_deposit_bonus
- **Cap growth**: Adjust max_investment_cap_for_level

---

## 7. TELEMETRY & KPI

### 7.1 Core Metrics

**Player Engagement**:
| Metric | Definition | Target | Calculation |
|--------|------------|--------|-------------|
| DAU | Daily Active Users | TBD | Unique players per day |
| MAU | Monthly Active Users | TBD | Unique players per 30 days |
| DAU/MAU Ratio | Stickiness | >20% | (DAU / MAU) * 100 |
| Average Session Length | Time per visit | >5 min | SUM(session_duration) / COUNT(sessions) |
| Sessions per DAU | Frequency | >3 | COUNT(sessions) / COUNT(unique_users_today) |

**Retention**:
| Metric | Definition | Target |
|--------|------------|--------|
| D1 Retention | % return next day | >40% |
| D7 Retention | % return after 7 days | >20% |
| D30 Retention | % return after 30 days | >10% |

**Progression**:
| Metric | Definition |
|--------|------------|
| Average Level | MEAN(player_level) across all active players |
| Level Distribution | COUNT(players) per level bucket (1-3, 4-7, 8-10) |
| Time to Level 3 | Median hours from account creation to reaching Level 3 |
| Time to Level 10 | Median days from account creation to max level |
| % Players Reaching Level 10 | Completion rate |

**Economy**:
| Metric | Definition |
|--------|------------|
| Total Gems in Economy | SUM(player_gem_balance) + SUM(active_deposit_principals) |
| Gems Earned (Lifetime) | SUM(all mine_collection events) + SUM(all deposit_collection events) |
| Gems Spent (Lifetime) | SUM(all upgrade_level costs) + SUM(all deposit_investment amounts) |
| Average Gem Balance | MEAN(player_gem_balance) |
| Median Gem Balance | MEDIAN(player_gem_balance) |

### 7.2 Event Tracking

**Required Events** (snake_case):

| Event Name | When Fired | Parameters |
|------------|------------|------------|
| `session_start` | Player opens app | `player_id`, `player_level`, `session_id`, `timestamp` |
| `session_end` | Player closes/idles | `player_id`, `session_id`, `session_duration_seconds`, `timestamp` |
| `mine_gems_collected` | Tap Collect on mine | `player_id`, `player_level`, `gems_collected`, `current_storage_before`, `player_gem_balance_before`, `player_gem_balance_after`, `timestamp` |
| `tab_switched` | Change active tab | `player_id`, `from_tab`, `to_tab`, `timestamp` |
| `level_info_viewed` | Tap info (i) button | `player_id`, `player_level`, `timestamp` |
| `level_upgraded` | Complete upgrade | `player_id`, `from_level`, `to_level`, `gems_spent`, `player_gem_balance_before`, `player_gem_balance_after`, `timestamp` |
| `deposit_detail_viewed` | Tap deposit card | `player_id`, `deposit_tier_id`, `player_level`, `timestamp` |
| `deposit_slider_adjusted` | Move slider | `player_id`, `deposit_tier_id`, `investment_amount`, `timestamp` (sample 10% of events) |
| `deposit_started` | Confirm deposit | `player_id`, `deposit_tier_id`, `investment_amount`, `expected_payout`, `deposit_duration_days`, `player_level`, `total_interest_rate`, `player_gem_balance_before`, `player_gem_balance_after`, `timestamp` |
| `deposit_completed` | Deposit timer ends | `player_id`, `deposit_tier_id`, `investment_amount`, `timestamp_completed` |
| `deposit_collected` | Tap to collect payout | `player_id`, `deposit_tier_id`, `principal`, `payout`, `actual_interest_earned`, `deposit_duration_days`, `time_to_collect_after_completion_seconds`, `player_gem_balance_before`, `player_gem_balance_after`, `timestamp` |
| `locked_feature_tapped` | Tap locked deposit or upgrade tab | `player_id`, `feature_type` (deposit/upgrade), `unlock_requirement`, `timestamp` |
| `tooltip_shown` | Any tooltip displayed | `player_id`, `tooltip_message`, `trigger_element`, `timestamp` |

### 7.3 Funnel Analysis

**Investment Funnel**:
```
1. Players who view Investments tab
   ↓ (conversion_rate_1)
2. Players who tap available deposit card
   ↓ (conversion_rate_2)
3. Players who adjust investment slider
   ↓ (conversion_rate_3)
4. Players who tap "Deposit" button
   ↓ (conversion_rate_4)
5. Players who confirm in modal
   ↓ (conversion_rate_5)
6. Players who collect completed deposit
```

**Progression Funnel**:
```
New Player
   ↓
First Mine Collection (tutorial completion rate)
   ↓
Reach Level 3 (unlock first deposit)
   ↓
First Deposit Investment (engagement rate)
   ↓
First Deposit Collection (completion rate)
   ↓
Reach Level 10 (mastery rate)
```

**Retention Cohort**:
```
Group players by:
  - First action taken (mine vs deposit)
  - Time to first deposit
  - Number of deposits in first 7 days

Track D1/D7/D30 retention per cohort
```

### 7.4 A/B Test Hooks

**Testable Variables**:
- starting_gem_balance (currently: 10,000)
- gem_production_interval_in_seconds (varies by level)
- deposit_base_interest_rates (all tiers)
- level_upgrade_costs (all levels)
- deposit_unlock_levels (when tiers unlock)
- investment_slider_step_size (currently: 10)

**Recommended Tests**:
1. **Faster Onboarding**: Reduce gem_production_interval at Level 1 by 50%
2. **Higher Starting Capital**: Increase starting_gem_balance to 20,000
3. **Earlier Deposits**: Unlock 7-day at Level 2 instead of 3
4. **Better Early Returns**: Increase deposit_base_interest_7_day by 5%

---

## 8. QA & TESTING

### 8.1 Functional Test Cases

#### Mine Collection Tests

| Test ID | Test Case | Steps | Expected Result |
|---------|-----------|-------|-----------------|
| MINE-001 | Collect with gems available | 1. Wait for storage >= 1<br>2. Tap Collect | Gems added to balance<br>Storage resets to 0<br>Particles spawn |
| MINE-002 | Collect button disabled when empty | 1. Ensure storage < 1<br>2. Check button state | Button is disabled and gray |
| MINE-003 | Collect with full storage | 1. Wait for storage = max_storage_capacity<br>2. Tap Collect | All gems collected<br>Glowing text effect present before collection |
| MINE-004 | Production continues after collection | 1. Collect gems<br>2. Wait 10 seconds | Storage increases from 0 |
| MINE-005 | Fractional gems rounded down | 1. Collect when storage = 5.7 | player_gem_balance increases by 5 (floor) |
| MINE-006 | Collection updates "Time until full" | 1. Note time remaining<br>2. Collect<br>3. Check timer | Timer resets to maximum time |
| MINE-007 | Multiple rapid collections | 1. Collect gems<br>2. Wait 0.1s<br>3. Try to collect again | Second collection disabled (storage = 0) |

#### Deposit Tests

| Test ID | Test Case | Steps | Expected Result |
|---------|-----------|-------|-----------------|
| DEP-001 | Open unlocked deposit detail | 1. Reach required level<br>2. Tap deposit card | Detail screen opens<br>Shows stats correctly |
| DEP-002 | Tap locked deposit | 1. Below required level<br>2. Tap locked card | Tooltip shows unlock requirement |
| DEP-003 | Adjust investment slider | 1. Open detail<br>2. Drag slider | Amount updates in real-time<br>Expected payout recalculates |
| DEP-004 | Invest with sufficient gems | 1. Have enough gems<br>2. Set amount<br>3. Confirm | Gems deducted<br>Deposit starts<br>Timer begins |
| DEP-005 | Invest with insufficient gems | 1. Set amount > gem_balance<br>2. Try to confirm | Error or amount capped at balance |
| DEP-006 | Investment below minimum | 1. Try to set amount < minimum_investment<br>2. Confirm | Slider/input prevents invalid amount |
| DEP-007 | Investment above cap | 1. Try to set amount > max_investment_cap<br>2. Confirm | Slider capped at maximum |
| DEP-008 | Deposit timer countdown | 1. Start deposit<br>2. Wait 10s<br>3. Check timer | Time remaining decreases correctly |
| DEP-009 | Deposit completes | 1. Start deposit<br>2. Wait for completion (or debug skip) | Card shows "Collect!"<br>Pulsing animation<br>Badge on tab |
| DEP-010 | Collect completed deposit | 1. Wait for completion<br>2. Tap card | Payout calculated correctly<br>Gems added<br>Particles spawn<br>Deposit removed |
| DEP-011 | Multiple simultaneous deposits | 1. Start 7-day<br>2. Start 14-day<br>3. Start 30-day<br>4. Start 60-day | All 4 run independently<br>Each has own timer |
| DEP-012 | Cannot start second deposit in same tier | 1. Start 7-day deposit<br>2. Try to start another 7-day | First deposit still active<br>Second attempt blocked |
| DEP-013 | Interest calculation with level bonus | 1. Note current level bonus<br>2. Start deposit<br>3. Collect | Payout = floor(principal * (1 + base_interest + level_bonus)) |
| DEP-014 | Upgrade level during active deposit | 1. Start deposit at Level 5<br>2. Upgrade to Level 6<br>3. Collect later | Bonus locked at start time OR recalculated (define in implementation) |

#### Level Progression Tests

| Test ID | Test Case | Steps | Expected Result |
|---------|-----------|-------|-----------------|
| LVL-001 | Upgrade with sufficient gems | 1. Have enough gems<br>2. Tap Upgrade | Level increases<br>Gems deducted<br>Animation plays<br>Stats update |
| LVL-002 | Upgrade button disabled when insufficient | 1. Spend gems below cost<br>2. Check button | Button shows "Need X 💎"<br>Disabled state |
| LVL-003 | Cannot upgrade at max level | 1. Reach Level 10<br>2. Check upgrade UI | Shows "MAX LEVEL"<br>No upgrade button |
| LVL-004 | Level up animation plays | 1. Upgrade level | Overlay appears<br>Text bounces<br>Stars spawn<br>Dismisses after 2s |
| LVL-005 | Storage cap increases | 1. Note old cap<br>2. Upgrade<br>3. Check new cap | max_storage_capacity increased per balance table |
| LVL-006 | Production speed increases | 1. Note old rate<br>2. Upgrade<br>3. Time 1 gem | Production faster per balance table |
| LVL-007 | Deposit unlocks at threshold | 1. Reach Level 3/5/7/9<br>2. Check Investments tab | New tier becomes available |
| LVL-008 | Investment cap increases | 1. Note old cap<br>2. Upgrade<br>3. Open deposit detail | Slider max value increased |
| LVL-009 | View level info modal | 1. Tap (i) button | Modal opens<br>All 10 levels shown<br>Current level highlighted<br>Scrolls to current |
| LVL-010 | Level info shows correct stats | 1. Open modal<br>2. Check each level | Storage, production, bonus, cap, cost all match balance table |

#### UI/Navigation Tests

| Test ID | Test Case | Steps | Expected Result |
|---------|-----------|-------|-----------------|
| UI-001 | Switch to Gem Mine tab | 1. On different tab<br>2. Tap Gem Mine | Mine content visible<br>Other tabs hidden<br>Tab highlighted |
| UI-002 | Switch to Investments tab | 1. On different tab<br>2. Tap Investments | Investment cards visible<br>Other tabs hidden<br>Tab highlighted |
| UI-003 | Tap Upgrades tab | 1. Tap Upgrades | Shows "Coming Soon"<br>Tooltip appears<br>Lock icon visible |
| UI-004 | Badge appears on Gem Mine tab | 1. On Investments tab<br>2. Mine storage >= 1 | Red "!" badge visible on Gem Mine tab |
| UI-005 | Badge appears on Investments tab | 1. On Gem Mine tab<br>2. Complete a deposit | Red "!" badge visible on Investments tab |
| UI-006 | Badge disappears when switching | 1. Badge on tab<br>2. Switch to that tab | Badge removed |
| UI-007 | Deposit detail back button | 1. Open deposit detail<br>2. Tap back arrow | Returns to investment grid |
| UI-008 | Modal overlay dismisses on click | 1. Open any modal<br>2. Tap outside modal | Modal closes |
| UI-009 | Confirmation modal cancel | 1. Start deposit flow<br>2. Open confirmation<br>3. Tap Cancel | Modal closes<br>No gems deducted<br>No deposit started |
| UI-010 | Tooltip auto-dismisses | 1. Trigger tooltip<br>2. Wait 2s | Tooltip fades away |

### 8.2 Edge Cases

| Edge Case ID | Scenario | Expected Behavior |
|--------------|----------|-------------------|
| EDGE-001 | Player has exactly upgrade cost but 10 gems in storage | Allow collection first, then upgrade |
| EDGE-002 | Storage fills to max while player AFK | Storage stops at max_storage_capacity<br>Excess production lost |
| EDGE-003 | Multiple deposits complete simultaneously | All show completed state<br>Can collect in any order<br>Each spawns particles |
| EDGE-004 | Player upgrades level right before deposit completes | Deposit uses interest rate at start time (if locked) OR completion time (define in implementation) |
| EDGE-005 | Player closes app mid-animation | Animation stops<br>State changes persist<br>No gems lost |
| EDGE-006 | Player tries to invest 0 gems | Slider enforces minimum_investment_amount |
| EDGE-007 | Player has 995 gems, tries to invest 1000 | Slider capped at 990 (995 rounded down to nearest 10) |
| EDGE-008 | Player levels up from 2 to 3 while on Investments tab | 7-day deposit immediately becomes unlocked<br>Lock icon removed |
| EDGE-009 | Deposit completes while player offline | On return, deposit is completed<br>Timer shows "COMPLETED!" |
| EDGE-010 | Player spams Collect button rapidly | Only first click processes<br>Subsequent clicks do nothing (button disabled) |
| EDGE-011 | Player refreshes page during level up animation | Animation may not play<br>Level increase persists (if saved) |
| EDGE-012 | Storage at 9.99 gems | Collect button disabled (floor < 1) |
| EDGE-013 | Player invests last gem into deposit | gem_balance = 0<br>Mine can still produce<br>Cannot upgrade until collecting mine or deposit |
| EDGE-014 | Level 10 player with max storage full | Shows "MAX LEVEL" on upgrade<br>Can still collect and invest normally |
| EDGE-015 | Two deposits complete 1 second apart | Both show badges<br>Tab badge shows "!"<br>Both have pulsing animation |
| EDGE-016 | Player leaves tab open for 7+ days | Deposit completes on schedule<br>Mine storage capped at max<br>Timers stay accurate |
| EDGE-017 | Clock/timezone changes during deposit | Use server timestamp for deposit completion (if server-authoritative) OR local timestamp (if client-side) |
| EDGE-018 | Investment exactly equals max_investment_cap | Allowed (edge is inclusive: amount <= cap) |
| EDGE-019 | Player has active deposit, tier unlocks for same duration | Cannot start second deposit in same tier<br>Must wait for first to complete or collect |
| EDGE-020 | Payout calculation results in X.99 gems | Payout floored: floor(195.99) = 195 |

### 8.3 Performance Tests

| Test ID | Metric | Target | Test Method |
|---------|--------|--------|-------------|
| PERF-001 | Initial load time | <2s | Measure time to first render |
| PERF-002 | Tab switch latency | <100ms | Measure time from click to content visible |
| PERF-003 | Particle spawn lag | <16ms | Measure frame time during particle spawn |
| PERF-004 | Level up animation FPS | 60 FPS | Monitor frame rate during animation |
| PERF-005 | Memory usage (1 hour idle) | <50MB increase | Monitor heap size over time |
| PERF-006 | Timer accuracy (1 hour) | ±1 second | Compare displayed time to actual elapsed |
| PERF-007 | Multiple deposits timer update | 60 FPS | All 4 deposits updating simultaneously |

### 8.4 Compatibility Tests

| Platform | Browsers | Minimum Version | Test Coverage |
|----------|----------|-----------------|---------------|
| iOS | Safari | iOS 14+ | Primary |
| Android | Chrome | Android 9+ | Primary |
| Desktop | Chrome, Firefox, Safari, Edge | Latest 2 versions | Secondary |

**Specific Tests**:
- Portrait orientation lock (mobile only)
- Touch interactions (tap, slide)
- Emoji rendering (💎⛰️🏔️🗻⚡✨⭐)
- CSS animations (no jank)
- LocalStorage persistence (if implemented)

### 8.5 Regression Test Checklist

Run after any code change:
- [ ] Mine production ticks correctly
- [ ] Collect button adds correct gem amount
- [ ] All 4 deposit tiers can be started
- [ ] Deposit timers count down accurately
- [ ] Completed deposits can be collected
- [ ] Level upgrades work
- [ ] Level up animation plays
- [ ] Tab badges appear/disappear correctly
- [ ] All particles spawn correctly
- [ ] Investment slider respects caps
- [ ] Confirmation modal works
- [ ] Tooltips display and auto-hide
- [ ] Level info modal scrolls to current level
- [ ] Locked deposits show tooltips
- [ ] No console errors
- [ ] Save/load works (if implemented)

---

## 9. PERSISTENCE & SERVER COMMUNICATION

### 9.1 Save Data Structure

**Player State** (saved to server and/or localStorage):
```json
{
  "player_id": "uuid",
  "player_level": 1-10,
  "player_gem_balance": integer,
  "current_mine_storage": float,
  "last_save_timestamp": unix_timestamp,
  "active_deposits": [
    {
      "deposit_tier_id": "7day|14day|30day|60day",
      "deposit_principal": integer,
      "deposit_start_timestamp": unix_timestamp,
      "deposit_end_timestamp": unix_timestamp,
      "deposit_is_completed": boolean
    }
  ],
  "total_gems_earned_lifetime": integer,
  "total_gems_spent_lifetime": integer,
  "total_mine_collections": integer,
  "total_deposits_started": integer,
  "total_deposits_collected": integer,
  "account_creation_timestamp": unix_timestamp
}
```

### 9.2 When to Save

**Server Save Triggers** (Critical - must sync):
| Event | What to Save | Why |
|-------|--------------|-----|
| `mine_gems_collected` | player_gem_balance, current_mine_storage, last_save_timestamp | Prevent gem duplication |
| `level_upgraded` | player_level, player_gem_balance, last_save_timestamp | Prevent level/gem rollback |
| `deposit_started` | player_gem_balance, active_deposits array, last_save_timestamp | Lock gems, prevent double-spend |
| `deposit_collected` | player_gem_balance, active_deposits array, last_save_timestamp | Credit payout, prevent double-collect |

**Local Save Triggers** (Convenience - can rely on server):
| Event | What to Save | Why |
|-------|--------------|-----|
| `session_end` | Full player state | Quick resume without server fetch |
| Every 60 seconds | current_mine_storage, last_save_timestamp | Recover partial progress if crash |

### 9.3 Load Sequence

**On App Open**:
```
1. Show loading screen
2. Fetch player_state from server
3. IF server unavailable:
     → Load from localStorage (if exists)
     → Show "Offline Mode" indicator
4. Calculate catch-up production:
     time_offline = current_timestamp - last_save_timestamp
     gems_produced_while_offline = time_offline / gem_production_interval_in_seconds
     current_mine_storage = min(previous_storage + gems_produced_while_offline, max_storage_capacity)
5. Check active deposits:
     FOR EACH deposit in active_deposits:
         IF current_timestamp >= deposit_end_timestamp:
             SET deposit_is_completed = true
6. Render UI with updated state
7. Save corrected state to server
```

### 9.4 Conflict Resolution

**Scenario**: Player opens app on two devices simultaneously

**Strategy**: Server-authoritative (last write wins)
```
Device A: Collects 50 gems at 12:00:00
Device B: Collects 30 gems at 12:00:05
Server: Accepts Device B's state (more recent timestamp)
Result: Device A's collection overwritten (edge case, warn user)
```

**Alternative**: Queue-based (append-only event log)
```
Server maintains event log:
  12:00:00 - Device A - mine_gems_collected: +50
  12:00:05 - Device B - mine_gems_collected: +30
Server replays events in order to compute true state
Result: Both collections count (more complex, more accurate)
```

**Recommendation**: Use last-write-wins for prototype, implement event log if needed for production.

### 9.5 Network Error Handling

| Error | User Experience | Recovery |
|-------|-----------------|----------|
| Save failed | Show "Connection lost" banner<br>Allow continued play<br>Queue save for retry | Retry every 10s<br>Save to localStorage<br>Sync when reconnected |
| Load failed | Show "Cannot load progress"<br>Offer: Retry or Play Offline | Use localStorage if available<br>Warn unsaved progress at risk |
| Timeout (>5s) | Show loading spinner<br>Then error after timeout | Cancel request<br>Fall back to local |

### 9.6 Anti-Cheat Measures

**Client-Side Time Manipulation**:
- Problem: Player changes device clock to skip deposit timers
- Solution: Server validates timestamps
  - deposit_end_timestamp must be >= server_timestamp at start
  - Server refuses deposits that "complete" before scheduled

**Gem Balance Tampering**:
- Problem: Player edits localStorage to add gems
- Solution: Server stores authoritative balance
  - Client sends events, not absolute values
  - Server recalculates: `new_balance = old_balance + gems_collected - gems_spent`
  - If client balance doesn't match, reject and force sync

**Deposit Duplication**:
- Problem: Player starts deposit, disconnects, starts again on reconnect
- Solution: Idempotent deposit creation
  - Assign unique ID to each deposit at start
  - Server rejects duplicate deposit_id

**Debug Button Exploit**:
- Problem: Debug "SKIP TIME" button in production
- Solution: Remove or gate behind developer flag in production build

---

## 10. TECHNICAL REQUIREMENTS

### 10.1 Platform

- **Type**: Web application (Single-page app)
- **Framework**: React 18+ (via CDN)
- **Transpiler**: Babel (browser-based)
- **Rendering**: Client-side only
- **Hosting**: GitHub Pages (static hosting)

### 10.2 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| First Contentful Paint | <1.5s | Lighthouse |
| Time to Interactive | <2.5s | Lighthouse |
| Frame Rate (animations) | 60 FPS | Chrome DevTools |
| Memory Usage (idle) | <30MB | Chrome Task Manager |
| Bundle Size | <500KB (uncompressed) | Network tab |

### 10.3 Browser Support

- **iOS Safari**: 14.0+
- **Android Chrome**: 90+
- **Desktop Chrome**: Latest 2 versions
- **Desktop Firefox**: Latest 2 versions
- **Desktop Safari**: Latest 2 versions
- **Desktop Edge**: Latest 2 versions

**Not Supported**:
- Internet Explorer (any version)
- Browsers with JavaScript disabled

### 10.4 Dependencies

| Library | Version | Purpose | CDN |
|---------|---------|---------|-----|
| React | 18.x | UI framework | unpkg.com/react@18 |
| React DOM | 18.x | DOM rendering | unpkg.com/react-dom@18 |
| Babel Standalone | 7.x | JSX transpilation | unpkg.com/@babel/standalone |

**No build system required** - All dependencies loaded via CDN in single HTML file.

### 10.5 Data Storage

- **Primary**: Server-side database (TBD - PostgreSQL, MongoDB, etc.)
- **Cache**: LocalStorage (for offline support)
- **Capacity**: <5MB per player (localStorage limit safe)

### 10.6 API Endpoints (If Server Implemented)

| Endpoint | Method | Purpose | Request Body | Response |
|----------|--------|---------|--------------|----------|
| `/api/player/state` | GET | Load player data | - | Full player_state JSON |
| `/api/player/state` | POST | Save player data | Full player_state JSON | Success/error |
| `/api/player/collect_mine` | POST | Collect mine gems | `{ gems_collected, timestamp }` | Updated player_state |
| `/api/player/upgrade_level` | POST | Upgrade level | `{ new_level, gems_spent, timestamp }` | Updated player_state |
| `/api/player/start_deposit` | POST | Start deposit | `{ tier_id, amount, timestamp }` | Updated player_state + deposit_id |
| `/api/player/collect_deposit` | POST | Collect deposit | `{ deposit_id, timestamp }` | Updated player_state + payout |
| `/api/telemetry` | POST | Send analytics events | `{ events: [...] }` | 200 OK |

### 10.7 Security

- **HTTPS**: Required for production
- **CORS**: Configure server to allow origin
- **Authentication**: JWT or session-based (if multi-player)
- **Rate Limiting**: Prevent spam requests (e.g., 100 req/min per IP)
- **Input Validation**: Server validates all gem amounts, timestamps, levels

---

## 11. GLOSSARY

| Term | Definition |
|------|------------|
| **Active Deposit** | An investment that has been started but not yet collected |
| **Base Interest** | The minimum interest rate for a deposit tier (before level bonuses) |
| **Collection** | The act of manually claiming gems from mine or completed deposits |
| **Completed Deposit** | A deposit whose timer has reached zero, ready for collection |
| **Current Storage** | Number of gems currently in mine storage (can have decimals) |
| **Deposit Tier** | One of four time-locked investment options (7/14/30/60 day) |
| **Expected Payout** | Calculated return: principal × (1 + total interest) |
| **Gem Balance** | Total gems player owns and can spend (excludes locked deposit gems) |
| **Investment Cap** | Maximum gems that can be invested per deposit at current level |
| **Level Bonus** | Additional interest percentage added to all deposits based on player level |
| **Locked Gems** | Gems currently invested in active deposits (cannot be spent) |
| **Max Storage Capacity** | Maximum gems mine can hold before production is wasted |
| **Passive Production** | Automatic gem generation from mine over time |
| **Payout** | Final amount received when collecting a completed deposit |
| **Player Level** | Progression tier (1-10) that determines all system rates |
| **Principal** | Amount of gems invested in a deposit |
| **Production Interval** | Time (in seconds) to generate 1 gem |
| **Tab Badge** | Red notification indicator on inactive tabs |
| **Time-Locked** | Cannot be withdrawn or cancelled until duration elapses |
| **Total Interest** | Base interest + level bonus (used for payout calculation) |

---

## 12. BALANCE TABLES

### 12.1 Configuration Constants

| Variable Name | Value | Type | Description |
|---------------|-------|------|-------------|
| `max_player_level` | 10 | integer | Highest achievable level |
| `starting_gem_balance` | 10000 | integer | Gems player starts with |
| `minimum_investment_amount` | 10 | integer | Smallest allowed deposit |
| `investment_slider_step_size` | 10 | integer | Slider increment size |
| `viewport_max_width_pixels` | 414 | integer | Portrait container width |
| `tooltip_display_duration` | 2000 | integer (ms) | How long tooltips show |
| `level_up_animation_duration` | 2000 | integer (ms) | Level up overlay duration |
| `particle_animation_duration` | 2000 | integer (ms) | Particle float time |
| `particle_float_distance` | 300 | integer (px) | How far particles travel upward |
| `particle_spread_horizontal` | 100 | integer (px) | Random horizontal offset range |
| `particle_spread_vertical` | 50 | integer (px) | Random vertical offset range |
| `particle_count_mine` | 15 | integer | Particles on mine collection |
| `particle_count_deposit` | 15 | integer | Particles on deposit collection |
| `particle_count_level_up` | 20 | integer | Particles on level up |
| `mine_production_tick_interval` | 100 | integer (ms) | How often storage updates |
| `deposit_timer_tick_interval` | 1000 | integer (ms) | How often deposit timers update |
| `seconds_per_day` | 86400 | integer | Seconds in one day |
| `button_border_radius` | 12 | integer (px) | Border radius for buttons |
| `card_border_radius` | 12 | integer (px) | Border radius for cards |
| `card_border_width` | 2 | integer (px) | Card border thickness |
| `slider_track_height` | 8 | integer (px) | Investment slider track height |
| `slider_thumb_size` | 24 | integer (px) | Investment slider thumb size |

### 12.2 Level Progression Table

| Level | max_storage_capacity_for_level | gem_production_interval_in_seconds_for_level | level_upgrade_cost_for_level | level_deposit_bonus_for_level | max_investment_cap_for_level |
|-------|-------------------------------|---------------------------------------------|----------------------------|------------------------------|------------------------------|
| 1 | 10 | 18000 (5h) | 500 | 0.00 (0%) | 100 |
| 2 | 20 | 14400 (4h) | 600 | 0.02 (2%) | 250 |
| 3 | 30 | 10800 (3h) | 700 | 0.04 (4%) | 500 |
| 4 | 40 | 9000 (2.5h) | 800 | 0.06 (6%) | 1000 |
| 5 | 50 | 7200 (2h) | 900 | 0.08 (8%) | 2000 |
| 6 | 60 | 5400 (1.5h) | 1000 | 0.10 (10%) | 4000 |
| 7 | 70 | 3600 (1h) | 1100 | 0.12 (12%) | 7000 |
| 8 | 80 | 2700 (45min) | 1200 | 0.14 (14%) | 12000 |
| 9 | 90 | 1800 (30min) | 1300 | 0.16 (16%) | 20000 |
| 10 | 100 | 900 (15min) | null | 0.20 (20%) | 50000 |

**Note**: level_upgrade_cost_for_level is null at Level 10 (cannot upgrade further).

### 12.3 Deposit Tiers Table

| deposit_tier_id | deposit_display_name | deposit_duration_in_days | deposit_base_interest | deposit_unlock_level | deposit_tier_icon |
|-----------------|---------------------|-------------------------|----------------------|---------------------|-------------------|
| 7day | 7-Day Deposit | 7 | 0.15 (15%) | 3 | ⛰️ |
| 14day | 14-Day Deposit | 14 | 0.35 (35%) | 5 | 🏔️ |
| 30day | 30-Day Deposit | 30 | 0.70 (70%) | 7 | 🗻 |
| 60day | 60-Day Deposit | 60 | 1.20 (120%) | 9 | ⚡ |

### 12.4 Formula Reference

**Mine Production**:
```
gems_produced_per_tick = mine_production_tick_interval / 1000 / gem_production_interval_in_seconds_for_level
new_storage = min(current_mine_storage + gems_produced_per_tick, max_storage_capacity_for_level)
```

**Time Until Full**:
```
remaining_capacity = max_storage_capacity_for_level - current_mine_storage
time_until_full_in_seconds = remaining_capacity * gem_production_interval_in_seconds_for_level
```

**Deposit Payout**:
```
total_interest_rate = deposit_base_interest_for_tier + level_deposit_bonus_for_level
expected_payout = floor(deposit_principal * (1 + total_interest_rate))
actual_gems_earned = expected_payout - deposit_principal
```

**Deposit Progress**:
```
total_duration_in_seconds = deposit_duration_in_days * seconds_per_day
time_elapsed_in_seconds = current_timestamp - deposit_start_timestamp
time_remaining_in_seconds = max(0, deposit_end_timestamp - current_timestamp)
progress_percentage = (time_elapsed_in_seconds / total_duration_in_seconds) * 100
```

**Level Up Requirements**:
```
can_upgrade = (player_level < max_player_level) AND (player_gem_balance >= level_upgrade_cost_for_level)
```

**Investment Constraints**:
```
min_allowed_investment = minimum_investment_amount
max_allowed_investment = min(player_gem_balance, max_investment_cap_for_level)
investment_amount must be multiple of investment_slider_step_size
investment_amount must satisfy: min_allowed_investment <= investment_amount <= max_allowed_investment
```

---

## APPENDIX: STATE MACHINE DIAGRAMS

### Mine Collection State Machine
```
┌─────────────┐
│   EMPTY     │ (storage < 1)
│ Button: OFF │
└─────────────┘
      ↓ (production tick)
      ↓ storage >= 1
┌─────────────┐
│  READY      │ (storage >= 1)
│ Button: ON  │
│ Animation:  │
│ Pulse+Spark │
└─────────────┘
      ↓ (tap Collect)
┌─────────────┐
│ COLLECTING  │
│ Particles   │
│ Spawn       │
└─────────────┘
      ↓ (instant)
      ↓ storage = 0
┌─────────────┐
│   EMPTY     │
└─────────────┘
```

### Deposit Lifecycle State Machine
```
┌─────────────┐
│   LOCKED    │ (player_level < unlock_level)
│ Show: 🔒    │
└─────────────┘
      ↓ (player reaches unlock_level)
┌─────────────┐
│  AVAILABLE  │
│ Show: Card  │
│ Button:     │
│ "Invest"    │
└─────────────┘
      ↓ (tap card → confirm → execute)
┌─────────────┐
│   ACTIVE    │
│ Timer: Run  │
│ Progress:   │
│ Updating    │
└─────────────┘
      ↓ (time >= end_timestamp)
┌─────────────┐
│  COMPLETED  │
│ Animation:  │
│ Pulse Green │
│ Badge: Tab  │
└─────────────┘
      ↓ (tap to collect)
┌─────────────┐
│ COLLECTING  │
│ Particles   │
│ Spawn       │
└─────────────┘
      ↓ (instant)
      ↓ deposit deleted
┌─────────────┐
│  AVAILABLE  │ (back to start)
└─────────────┘
```

### Level Progression State Machine
```
┌─────────────┐
│  CAN'T      │ (gems < cost OR level = 10)
│  UPGRADE    │
│ Button:     │
│ Disabled    │
└─────────────┘
      ↓ (earn enough gems)
┌─────────────┐
│    CAN      │ (gems >= cost AND level < 10)
│  UPGRADE    │
│ Button:     │
│ Enabled     │
└─────────────┘
      ↓ (tap Upgrade)
┌─────────────┐
│ UPGRADING   │
│ Animation:  │
│ 2s Overlay  │
└─────────────┘
      ↓ (after 2s)
      ↓ level += 1
┌─────────────┐
│  NEW LEVEL  │
│ Stats       │
│ Updated     │
└─────────────┘
      ↓ (check if can upgrade again)
      ↓ (loop back to CAN'T or CAN)
```

---

**END OF DOCUMENT**

---

## Document Metadata

- **Author**: Generated from Primal Trove V3 Implementation
- **Date**: January 2026
- **Version**: 3.0
- **Status**: Final
- **Next Review**: After playtesting results

---