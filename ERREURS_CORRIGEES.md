# Erreurs rencontrées & corrigées / Bugs Encountered & Fixed
### Dragon Quest Monsters: Joker 2 Professional (NDS) — traduction / translation

**🇫🇷** Ce document liste tous les bugs sur lesquels on est tombé en traduisant DQMJ2 Pro en français, et **comment on les a corrigés**. Il s'adresse à quiconque voudrait traduire ce jeu dans une **autre langue** : la plupart de ces pièges se reproduiront, car ils viennent du **moteur du jeu**, pas de la langue. Basé sur le projet de traduction anglaise de Ceris White & co (crédit à eux pour les outils et la base technique).

**🇬🇧** This document lists every bug we hit while translating DQMJ2 Pro to French, and **how we fixed each one**. It is meant for anyone translating this game into **another language**: most of these traps will happen again, because they come from the **game engine**, not the language. Built on Ceris White & co's English translation project (credit to them for the tools and technical groundwork).

---

## Principe général / General principle

**🇫🇷** La cause n°1 des bugs : **le texte traduit est plus long que l'original** (japonais, ou même anglais). Le jeu a des **tampons mémoire de taille fixe** calibrés pour l'original. Un texte trop long les **déborde** → soit ça plante, soit ça s'affiche **vide**. Réflexe : comparer la taille de chaque fichier/entrée à la version **anglaise** (référence prouvée qui fonctionne), et condenser si on dépasse.

**🇬🇧** The #1 cause of bugs: **translated text is longer than the original** (Japanese, or even English). The game uses **fixed-size memory buffers** sized for the original. Text that is too long **overflows** them → either it crashes, or it renders **blank**. Rule of thumb: compare every file/entry size against the **English** version (a proven-working reference), and condense if you go over.

---

## 1. Le jeu se fige après la saisie du nom / Freeze after name entry

**🇫🇷**
- **Symptôme** : écran figé juste après avoir entré le nom du héros (au tout début).
- **Cause** : les textes traduits (plus longs que le japonais) débordent le **pool de messages** (pool 0), dont la taille d'origine `0x2a800` est calibrée pour le japonais.
- **Correction (OBLIGATOIRE)** : patch arm9 à l'offset RAM **`0x02026EC8`**, agrandir la taille du pool 0 à **`0x35000`** (instruction `MOV R1, #imm`). Sans ce patch, aucune traduction ne démarre.

**🇬🇧**
- **Symptom**: frozen screen right after entering the hero's name (very start of the game).
- **Cause**: translated text (longer than Japanese) overflows the **message pool** (pool 0), whose original size `0x2a800` is calibrated for Japanese.
- **Fix (MANDATORY)**: arm9 patch at RAM offset **`0x02026EC8`**, grow pool 0's size to **`0x35000`** (a `MOV R1, #imm` instruction). Without this patch, no translation boots past the name screen.

---

## 2. Plantage du Manuel du dresseur / Trainer's Manual crash

**🇫🇷**
- **Symptôme** : plantage en ouvrant/faisant défiler certaines rubriques du Manuel (`msg_traveler`), typiquement une rubrique débloquée plus tard.
- **Cause** : chaque **entrée** du manuel est affichée dans un tampon dédié dont la taille max ≈ **la plus grande entrée anglaise (~259 octets)**. Une entrée traduite plus longue déborde et plante à l'affichage. (À ne pas confondre avec le total du fichier : ici c'est **par entrée**.)
- **Correction** : condenser **toute entrée > 259 octets** (le plafond = la plus grande entrée EN du fichier).

**🇬🇧**
- **Symptom**: crash when opening/scrolling certain Manual topics (`msg_traveler`), typically one unlocked later in the game.
- **Cause**: each manual **entry** is drawn into a dedicated buffer whose max size ≈ **the largest English entry (~259 bytes)**. A longer translated entry overflows and crashes on display. (Not to be confused with the file total: this is **per entry**.)
- **Fix**: condense **any entry > 259 bytes** (ceiling = the largest EN entry in that file).

---

## 3. Menu de combat + messages de combat VIDES / Battle menu + messages BLANK

**🇫🇷**
- **Symptôme** : en combat, le **menu de commandes** (Combattre/Ordres/Tactique/Dresser/Objets/Fuir) **ET** les messages de combat s'affichent **vides**, alors que d'autres textes (les tactiques, qui sont dans `message0`) marchent.
- **Cause** : en combat, **`msg_battle` est chargé dans un tampon dédié CACHÉ** calibré ~taille anglaise (**payload EN ≈ 3295 o**). Si le `msg_battle` traduit dépasse (le FR faisait **4181 o**), **TOUT `msg_battle` échoue à se charger** → menu + messages vides. Piège : le **pool général (`0x35000`) n'est PAS le coupable** (il a largement la marge, ~66 Ko de libre). Ce tampon de combat est séparé et invisible dans le pool.
- **Correction** : **condenser `msg_battle` sous la taille anglaise** (on est descendus à 2939 o). Contrepartie assumée : messages de combat plus secs (ex. « Attaque rétablie. » au lieu de « L'attaque du monstre redevient normale »). Les 6 libellés du menu (entrées 57-62) sont courts, on les garde tels quels ; c'est le **reste** de `msg_battle` qu'on raccourcit.

**🇬🇧**
- **Symptom**: in battle, the **command menu** (Fight/Orders/Tactics/Scout/Items/Flee) **AND** the battle messages render **blank**, while other text (the tactics, which live in `message0`) works fine.
- **Cause**: in battle, **`msg_battle` is loaded into a HIDDEN dedicated buffer** sized ~English size (**EN payload ≈ 3295 bytes**). If the translated `msg_battle` is bigger (French was **4181 bytes**), the **WHOLE `msg_battle` fails to load** → blank menu + messages. Trap: the **general pool (`0x35000`) is NOT the culprit** (it has plenty of room, ~66 KB free). This battle buffer is separate and invisible inside the pool.
- **Fix**: **condense `msg_battle` below the English size** (we got down to 2939 bytes). Accepted tradeoff: terser battle messages (e.g. "Attack restored." instead of "The monster's attack is back to normal"). The 6 menu labels (entries 57-62) are short, keep them as-is; it's the **rest** of `msg_battle` you shorten.

---

## 4. Chaque ramassage affiche le MÊME objet (faux) / Every pickup shows the SAME (wrong) item

**🇫🇷**
- **Symptôme** : quel que soit l'objet ramassé, le message dit toujours le même (chez nous « obtient le sceptre Moins »), alors que le **bon objet arrive bien dans le sac**.
- **Cause** : le transplant de dialogues a **aligné des ramassages génériques japonais** — qui utilisent la variable **`{E328}`** (= « l'objet réellement obtenu ») — **sur des répliques européennes qui nommaient un objet précis en dur**. Résultat : **191 lignes** avec un nom d'objet **figé** au lieu de `{E328}`. Le japonais d'origine écrit : `{NAME}は {E328}を 手に入れた`.
- **Correction** : partout où la ligne japonaise d'origine utilise `{E328}`, **remettre `{E328}`** à la place du nom figé. (Chez nous : script `repare_ramassage_objets.py` qui compare chaque ligne au script JP.) Attention à ne PAS toucher les vrais coffres à objet unique, où le japonais nomme l'objet en dur lui aussi (ex. via une icône de polarité `{232}`).

**🇬🇧**
- **Symptom**: whatever item you pick up, the message always shows the same one (for us "obtains the Minus Sceptre"), while the **correct item is actually added to the bag**.
- **Cause**: the dialogue transplant **aligned generic Japanese pickups** — which use the **`{E328}`** variable (= "the item actually obtained") — **onto European lines that named a specific item literally**. Result: **191 lines** with a **hardcoded** item name instead of `{E328}`. The original Japanese reads: `{NAME}は {E328}を 手に入れた`.
- **Fix**: wherever the original Japanese line uses `{E328}`, **restore `{E328}`** in place of the hardcoded name. (For us: a `repare_ramassage_objets.py` script that compares each line to the JP script.) Be careful NOT to touch genuine unique-item chests, where Japanese hardcodes the item too (e.g. via a `{232}` polarity icon).

---

## 5. Codes de contrôle {E3xx} : garder ceux du moteur / {E3xx} control codes: keep the engine's

**🇫🇷**
- **Symptôme** : nom du héros, nom d'objet, accords d'articles cassés dans les dialogues (glyphes bizarres, variables non résolues).
- **Cause** : le moteur de Pro **comprend des codes variables « européens »** (`{E321}`, `{E323}`, `{E326}`, `{E327}`, `{E328}`, `{E32A}`) **même si les scripts japonais ne les utilisent pas** ; le **patch anglais les AJOUTE** et ça fonctionne. Les **retirer** casse la résolution des variables.
- **Correction** : conserver **exactement le jeu de codes que le patch anglais emploie** (relevé EN : `E321`×157, `E322`, `E323`×21, `E324`, `E326`×21, `E327`×21, `E328`×256, `E32A`, `E30B`×48). Ne retirer **que** les codes présents **uniquement** côté européen et **jamais** employés par l'anglais (`E33A`, `E36A/B/C`, `E391`, `E39E`, `E3DF`, `E3F6`). Le `{E32A}` européen (nom d'objet) doit être **réécrit `{E328}`** (le code objet côté Pro/anglais).

**🇬🇧**
- **Symptom**: hero name, item name, article agreement broken in dialogue (weird glyphs, unresolved variables).
- **Cause**: the Pro engine **understands "European" variable codes** (`{E321}`, `{E323}`, `{E326}`, `{E327}`, `{E328}`, `{E32A}`) **even though the Japanese scripts don't use them**; the **English patch ADDS them** and it works. **Removing** them breaks variable resolution.
- **Fix**: keep **exactly the set of codes the English patch uses** (EN counts: `E321`×157, `E322`, `E323`×21, `E324`, `E326`×21, `E327`×21, `E328`×256, `E32A`, `E30B`×48). Only remove codes present **only** on the European side and **never** used by English (`E33A`, `E36A/B/C`, `E391`, `E39E`, `E3DF`, `E3F6`). The European `{E32A}` (item name) must be **rewritten `{E328}`** (the Pro/English item-name code).

---

## 6. Caractères interdits par l'encodeur / Characters the encoder rejects

**🇫🇷**
- **Symptôme** : l'encodeur de texte plante sur certains caractères.
- **Cause** : ils sont **absents de la table d'encodage** de Pro (table en RAM arm9 à **`0x020dbc16`**).
- **Correction** : remplacer avant encodage : `’` → `'` ; `« »` → `"` ; `—` `–` → `-` ; `ÿ æ Æ` → reformuler. (Prévoir une table de repli automatique.)

**🇬🇧**
- **Symptom**: the text encoder crashes on certain characters.
- **Cause**: they are **absent from Pro's encoding table** (table in arm9 RAM at **`0x020dbc16`**).
- **Fix**: replace before encoding: curly `'` → straight `'` ; `« »` → `"` ; em/en dash `—` `–` → `-` ; `ÿ æ Æ` → rephrase. (Set up an automatic fallback table.)

---

## 7. La sauvegarde a une somme de contrôle / The save has a checksum

**🇫🇷**
- **Symptôme** : après modification manuelle d'octets de la save (ex. changer le nom du héros), le jeu affiche « les données de sauvegarde sont corrompues » et refuse de charger.
- **Cause** : la save (64 Ko, **2 copies** à `0x0` et `0x7100`) est **protégée par une somme de contrôle**. L'éditeur de save officiel ne la recalcule pas non plus.
- **Contournement** : **ne pas patcher la save octet par octet.** Pour changer un élément bloquant (ex. un nom saisi en japonais), faire une **nouvelle partie**. Toujours **sauvegarder** la save avant toute manip. (melonDS lie le `.sav` au **nom du fichier ROM** : garder un nom de ROM constant pour ne pas perdre la save à chaque rebuild.)

**🇬🇧**
- **Symptom**: after manually editing save bytes (e.g. changing the hero name), the game shows "the save data is corrupted" and refuses to load.
- **Cause**: the save (64 KB, **2 copies** at `0x0` and `0x7100`) is **protected by a checksum**. The official save editor doesn't recompute it either.
- **Workaround**: **don't patch the save byte-by-byte.** To change something blocking (e.g. a name typed in Japanese), start a **new game**. Always **back up** the save before any tinkering. (melonDS ties the `.sav` to the **ROM filename**: keep a constant ROM name so the save survives every rebuild.)

---

## 8. Décalage d'un cran dans les fichiers de correction / Off-by-one in correction files

**🇫🇷**
- **Symptôme** : une correction tombe une ligne à côté (corrige/casse la mauvaise entrée).
- **Cause** : les fichiers de correction `.tsv` sont **indexés à partir de 1** (ligne N → tableau `[N-1]`), alors que les index Python (`fr[i]`) partent de **0**.
- **Correction** : écrire **`i+1`** dans le `.tsv` pour viser `fr[i]`. (Ce piège nous a mordus deux fois : Manuel puis msg_battle.)

**🇬🇧**
- **Symptom**: a correction lands one line off (fixes/breaks the wrong entry).
- **Cause**: correction `.tsv` files are **1-based** (line N → array `[N-1]`), while Python indices (`fr[i]`) are **0-based**.
- **Fix**: write **`i+1`** in the `.tsv` to target `fr[i]`. (This trap bit us twice: Manual then msg_battle.)

---

## 9. Décalage d'affichage des menus / Menu display shift

**🇫🇷**
- **Symptôme** : libellés de menu affichés au mauvais endroit (ex. « OK » dans une barre de titre, texte dupliqué).
- **Cause** : des **lignes vides manquantes AU MILIEU** d'un fichier de menu décalent **toutes** les entrées suivantes.
- **Correction** : aligner sur les **lignes non vides** ; règle : **1 ligne = 1 entrée**, la 1re ligne est un **sentinel vide** à conserver, jamais de saut de ligne à l'intérieur d'une entrée.

**🇬🇧**
- **Symptom**: menu labels shown in the wrong place (e.g. "OK" in a title bar, duplicated text).
- **Cause**: **missing empty lines IN THE MIDDLE** of a menu file shift **all** subsequent entries.
- **Fix**: align on **non-empty lines**; rule: **1 line = 1 entry**, the first line is an **empty sentinel** to keep, never put a line break inside an entry.

---

## 10. Fausse piste & contraintes / False lead & constraints

**🇫🇷**
- **⚠️ Fausse piste (ne PAS reproduire notre erreur)** : au début on a cru qu'un **nom de héros en japonais** cassait l'affichage de `{NAME}` (glyphe bizarre dans « [glyphe] obtient... »). **C'était FAUX.** Le vrai coupable était le code **`{E321}` qu'on avait retiré à tort** (voir bug 5) : sans lui, `{NAME}` ne se résout pas. Une fois `{E321}` conservé, `{NAME}` s'affiche correctement — **même avec un nom japonais** (l'octet de « さ » se rend d'ailleurs très bien dans les menus). Bref : ne blâmez pas le nom du joueur, blâmez les codes de variables retirés.
- **Longueur des noms de monstres** : au-delà de **13 caractères affichés**, bug du clavier de nommage. Garder les noms ≤ 13.

**🇬🇧**
- **⚠️ False lead (do NOT repeat our mistake)**: at first we thought a **Japanese hero name** broke `{NAME}` rendering (weird glyph in "[glyph] obtains..."). **That was WRONG.** The real culprit was the **`{E321}` code we had wrongly removed** (see bug 5): without it, `{NAME}` doesn't resolve. Once `{E321}` is kept, `{NAME}` renders correctly — **even with a Japanese name** (the "さ" byte actually renders fine in menus). In short: don't blame the player's name, blame the removed variable codes.
- **Monster name length**: beyond **13 displayed characters**, the naming keyboard bugs out. Keep names ≤ 13.

---

*🇫🇷 Rédigé au fil de la traduction française (2026). Réutilisez et complétez librement pour votre langue. / 🇬🇧 Written during the French translation (2026). Reuse and extend freely for your language.*
