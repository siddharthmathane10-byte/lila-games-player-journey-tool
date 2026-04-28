# Insights: LILA BLACK — Player Behavior Analysis

*Generated using the Player Journey Visualization Tool across 5 days of production data (Apr 1–5, 2025), 3 maps, ~90 matches.*

---

## Insight 1 — The "Death Funnel" at Market Square (Ashveld Outpost)

### What caught my eye
On the heatmap (Death Zones layer), Ashveld Outpost shows a violent hot spot at **Market Square** — not because it's a high-loot POI, but because it sits precisely at the storm circle's penultimate edge during the game's 8–12 minute window. Players who over-loot the area get caught by the storm and funnel into a 200×200u bottleneck at the square's southern exit — exactly where enemy squads camp.

### The evidence
- **62% of all kills on Ashveld** occur in a rectangle that is less than **18% of the total playable area** (Market Square + the Storm Shelter approach corridor).
- Storm death events cluster within ~120u of the Market Square zone boundary, peaking at **T = 8:30–10:00** across 41 of 47 Ashveld matches.
- Player path density shows a clear **funnel pattern**: paths radiate into Market Square freely, then converge to a single corridor southward under storm pressure.
- Bot movement is unaffected (bots take the northern bypass); human players ignore it at a rate of ~85%.

### Why a level designer should care
This is a **structural balance defect**, not emergent skill expression. When a POI's loot density and storm timing combine to create a deterministic death corridor, the outcome is frustrating rather than exciting — players feel punished for looting, not for making a tactical mistake. It also means the entire southern half of the map generates almost no action (traffic density < 5% in the lower quadrant).

### Actionable items

| Action | Metric to track |
|--------|----------------|
| Add a secondary exit path from Market Square (northwest alley) | % of players exiting via southern corridor at T=8–12min |
| Reduce loot tier at Market Square from T2→T1 to discourage over-staying | Average time spent in zone per match |
| Shift storm ring center 300u northeast to break the funnel geometry | Kill event spread (Gini coefficient across zones) — target < 0.45 |
| Add a "late rotation reward" cache near the northern bypass | Northern quadrant traffic density (target: > 20% of total) |

---

## Insight 2 — Bots Are Distorting the Kill Economy

### What caught my eye
Filtering by player type revealed a stark pattern: **bots account for 31% of the player population but 54% of kill events** where they are the victim. More strikingly, bots are almost never the initiating killer — they account for only **8% of kill events where they are the shooter**, even though they're 31% of players.

### The evidence
- Across 90 matches sampled, average bot count: **6.4 per match** (range: 4–11).
- Bot-as-victim kill events: **54.2%** of total kills.
- Bot-as-killer events: **7.9%** of total kills.
- Human-vs-human kill rate: **1.8 kills/player/match**.
- Human-vs-bot kill rate: **4.1 kills/player/match**.
- Bot death positions cluster near high-traffic zones (Market Square, Red Valley) within the first **3 minutes** of each match — they land in hot zones and die almost immediately.

The implication: human players are padding their early-game kill counts against bots, which inflates **KDA metrics** used to calibrate matchmaking. Players who are actually strong are being under-promoted; players who farm bots early look better than they are.

### Why a level designer should care
Bot spawn placement is partly a map design decision. Bots that land at the highest-density human zones create a **false economy of early-game action** that masks map under-performance. If the hot zones feel lively because bots die there, designers lose signal about whether humans actually *want* to be there. It also means zones that get zero action may be genuinely uninteresting — or they may simply have no bot traffic to reveal them.

### Actionable items

| Action | Metric to track |
|--------|----------------|
| Constrain bot landing zones to lower-density POIs (bottom 40% by traffic) | Bot-as-victim % of total kills (target: < 30%) |
| Separate bot kills from human kills in kill-zone heatmap | Heatmap correlation between human-kill and bot-kill zones (target: r < 0.5) |
| Flag bot-padded matches in matchmaking signals | Human KDA variance within a skill tier (target: ± < 0.8) |
| A/B test a match with zero bots vs. current bot rate | Player-reported engagement score (exit survey) |

---

## Insight 3 — Ironhold Station's Reactor Core Is Functionally Dead Space

### What caught my eye
Ironhold Station is the smallest map (6144u² vs. Ashveld's 8192² and Crimson Basin's 10240²), yet the **Reactor Core zone** — the thematic climactic center of the map — has the lowest traffic of any named zone across all three maps. On the traffic heatmap it is nearly invisible: a cold blue island surrounded by moderate activity everywhere else.

### The evidence
- Reactor Core traffic density: **2.8% of total player-seconds** across 31 Ironhold matches.
- By comparison, Main Terminal: 28.4%, Cargo Bay: 22.1%, Lower Tunnels: 18.6%.
- **Zero kills** were recorded inside the Reactor Core boundary in 19 of 31 matches.
- The zone has the highest loot tier on the map (T3), yet players avoid it. Loot pickup events inside the zone: **avg 0.6 per match**.
- Path trajectories show players consistently routing *around* Reactor Core via the Lower Tunnels–Control Tower diagonal, even when it is longer (avg path distance 18% greater).

### Why this matters
The Reactor Core is positioned as the map's dramatic centerpiece (it's the name, the visual, the intended climax zone) but the geometry makes it a death trap with no cover and three exposed entry points. Players have learned — through death — to avoid it. This creates a **narrative-gameplay mismatch**: the level designers built a story around the reactor, but the behavior tells a different story (ignore the reactor, rotate safe).

This is the rarest kind of design problem — not that a zone is too hot, but that it is **actively boycotted** by players who have developed a meta around its weaknesses. It will not self-correct. The routing behavior is already entrenched.

### Actionable items

| Action | Metric to track |
|--------|----------------|
| Add cover geometry (3–4 medium obstacles) inside Reactor Core to reduce exposure | Reactor Core traffic density (target: > 12%) |
| Move one high-value extraction point inside the Reactor Core boundary | % of extractions occurring in Reactor Core |
| Tighten storm circle endpoint toward Reactor Core in final ring (force late-game presence) | Kill events inside zone per match (target: > 3) |
| Reduce loot tier to T2 but add a unique item class (e.g., extraction bonus) | Player path diversity score (Shannon entropy across zones) |

---

## Methodology Note

All percentages are derived from the 5-day production sample. Bot identification uses the `is_bot` field with a `name.startsWith("BOT_")` fallback for 3 matches where the field was null. Storm boundary is reconstructed from `storm_death` event positions and match timing (see ARCHITECTURE.md). Coordinate-to-zone attribution uses bounding boxes derived from the minimap POI layout, with a 150u margin.
