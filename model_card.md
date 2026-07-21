# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

**VibeFinder 1.0**

---

## 2. Intended Use  

**Goal / task:** Predict and explain personalized song recommendations from a small catalog, using a user’s preferred genre, mood, and target energy.

**Intended use**
- Classroom / portfolio exploration of content-based recommenders  
- Demo how scoring + ranking turn preference features into a top‑k list with human-readable reasons  
- Compare how different taste profiles (e.g. chill vs high-energy) change results  

**Assumptions**
- The user can name one favorite genre, one favorite mood, and a target energy between 0 and 1  
- Exact label matches (genre/mood strings) are a fair proxy for taste in this toy setting  

**Non-intended use**
- Real production music apps or live user traffic  
- Fairness-critical or commercial ranking decisions  
- Capturing complex taste (multiple genres, playlists, skips, social signals, or long-term history)

---

## 3. How the Model Works  

*(Algorithm summary)*

VibeFinder scores every song for one user, then sorts by score and returns the top results.

**Song features used in scoring:** genre, mood, and energy.  
**User preferences:** `favorite_genre`, `favorite_mood`, `target_energy`.

**Scoring rule (per song)**
1. **+2.0** if the song’s genre matches the user’s favorite genre  
2. **+1.0** if the song’s mood matches the user’s favorite mood  
3. **Energy similarity:** `1.0 - |song energy − target energy|` (closer = higher)

**Ranking rule:** Sort all scored songs from highest to lowest and keep the top *k* (default 5). Each result includes the song, total score, and a short list of reasons (e.g. “genre match (+2.0)”).

Other CSV fields (tempo, valence, danceability, acousticness) are loaded for the catalog but not used in the current score. This is a simple content-based approach—no collaborative filtering from other users.

---

## 4. Data  

**Data used:** 18 songs in `data/songs.csv`.

**Fields:** id, title, artist, genre, mood, energy, tempo_bpm, valence, danceability, acousticness.

**Coverage:** Mix of starter tracks (pop, lofi, rock, ambient, jazz, synthwave, indie pop) plus added diversity (folk, R&B, EDM, country, classical, metal, latin, blues) and moods such as chill, happy, intense, sad, romantic, and energetic.

**Gaps:** Still a tiny catalog—no lyrics, no listening history, limited artists, and uneven depth per genre. Many real taste dimensions (era, language, instrumentation) are missing.

---

## 5. Strengths  

- Separates **chill** vs **high-energy** tastes cleanly via the energy distance score  
- Stacked genre + mood + energy matches (e.g. *Library Rain* for Chill Lofi, *Sunrise City* for High-Energy Pop) feel intuitive  
- Explanations are easy to audit—you can see *why* a track ranked high  
- Works well for users with a clear primary vibe (one genre + one mood + a target energy)

---

## 6. Limitations and Bias 

Where the system struggles or behaves unfairly. 

**Genre filter bubble.** Genre gets the heaviest bonus (+2.0), so once a favorite genre is set, same-genre tracks rise to the top even when mood or energy are only a partial fit. That creates a small filter bubble: the system keeps reinforcing the labeled category instead of exploring nearby vibes. Example: for High-Energy Pop, *Gym Hero* ranks above *Rooftop Lights* largely because it is tagged `pop`, even though *Rooftop Lights* matches the happy mood better and has similar energy.

**Catalog imbalance.** The starter catalog still leans on a few well-covered styles (pop, lofi, chill). Genres we added later (folk, classical, latin, blues) only surface when energy is close and nothing genre/mood-matched competes. Users whose taste sits in thin categories get fewer true matches and more “energy-only” fillers.

**Exact-match rigidity.** Mood and genre only score on exact string equality. Close neighbors like `indie pop` vs `pop`, or `relaxed` vs `chill`, earn zero preference points even when they feel similar to a listener.

**Missing signals.** The scorer ignores valence, tempo, danceability, acousticness, artist, and listening history. Real taste is broader than three preferences, so the model can overfit to a single labeled genre and under-serve mixed or evolving tastes.

**Popularity / coverage bias (toy scale).** With only 18 songs, a few high-energy pop/rock tracks repeatedly appear across energetic profiles. That can look like “everyone gets Gym Hero,” which mirrors how real systems can over-favor dominant catalog slices.

---

## 7. Evaluation  

How you checked whether the recommender behaved as expected. 

We ran `src/main.py` against three distinct profiles and inspected the top 5 for each, including scores and reasons.

### Profiles tested

1. **High-Energy Pop** — `pop` / `happy` / energy `0.85`  
2. **Chill Lofi** — `lofi` / `chill` / energy `0.35`  
3. **Deep Intense Rock** — `rock` / `intense` / energy `0.90`

### What we looked for

- Clear separation between chill and energetic tastes  
- Genre + mood matches ranking above energy-only near-misses  
- Sensible explanations in the reasons list  

### Patterns we saw

**High-Energy Pop** surfaced *Sunrise City* first (genre + mood + near-perfect energy), then *Gym Hero* (genre + high energy, but mood is intense, not happy). *Rooftop Lights* appeared for mood + energy even though its genre is indie pop, not pop. Lower ranks (*Storm Runner*, *Coral Skies*) were mostly energy similarity—close intensity without preference matches.

**Chill Lofi** behaved as expected: *Library Rain* and *Midnight Coding* dominated with full genre + mood + energy alignment. *Focus Flow* stayed high on genre despite a `focused` mood. *Spacewalk Thoughts* (ambient/chill) and *Coffee Shop Stories* (jazz) showed how energy alone can pull in neighboring calm tracks when genre does not match.

**Deep Intense Rock** put *Storm Runner* clearly on top (genre + mood + energy ≈ 0.91). A notable cross-over: *Gym Hero* ranked second because it shares the `intense` mood and high energy, even though it is pop—not rock. High-energy fillers (*Bassline Riot*, *Iron Horizon*) then followed on energy alone.

### Surprises / plain-language comparisons

- **Why *Sunrise City* wins for High-Energy Pop:** It is the rare track that hits all three levers—pop, happy, and energy near 0.85—so the score stacks instead of relying on one signal.  
- **Why *Gym Hero* shows up for both Pop and Rock profiles:** It is high-energy and tagged `intense`. For Pop it rides the genre bonus; for Rock it rides the mood bonus. Same song, different reasons—proof that weighted rules can recycle popular high-energy tracks across “different” users.  
- **Why chill users almost never see Gym Hero / Sunrise City:** Their target energy (~0.35) makes those ~0.8–0.9 tracks score poorly on similarity, so genre/mood bonuses elsewhere easily outrank them.

Overall, the three-profile check confirmed the scorer separates chill from hype well, while also revealing genre overweighting and cross-profile repeats like *Gym Hero*.

---

## 8. Future Work  

*(Ideas for improvement)*

- Use more of the CSV: valence, tempo, danceability, and acousticness in the score  
- Soften exact matches (treat `indie pop` ≈ `pop`, `relaxed` ≈ `chill`)  
- Rebalance weights so genre does not dominate and reduce filter-bubble effects  
- Add diversity rules so the top 5 is not all one genre  
- Support multi-genre or “two vibe” user profiles  
- Grow the catalog so niche tastes have real candidates, not only energy fillers  

---

## 9. Personal Reflection  

Building VibeFinder taught me that recommenders are less “magic AI” and more clear rules: score each item, then rank. AI tools sped up repetitive coding—CSV loading, loops, formatting—while I stayed in control of the core scoring logic (weights, energy distance, and what counts as a match). The most satisfying moment was watching a simple formula like `1 - |energy − target|` make a chill user and a gym user get obviously different playlists. It felt surprisingly smart for how little math was involved—and it made apps like Spotify feel more understandable: under the polish, they’re still turning preferences and features into scores.
