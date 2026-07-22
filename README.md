# Dragon Quest Monsters : Joker 2 Professional — Traduction française

Traduction française intégrale de *Dragon Quest Monsters: Joker 2 Professional*
(Nintendo DS, 2011), jamais sorti hors du Japon.

Ce dépôt est un fork du projet [DQMJ2 Professional Translation](https://github.com/saneezore07/DQMJ2Pro_Translation)
de **Ceris White** et de son équipe, qui a rendu le jeu modifiable et l'a traduit
en anglais après quinze ans de blocage technique. Nous en reprenons l'outillage
et le patcheur pour proposer une version **française**.

---

## Fonctionnalités

- **Menus, dialogues de la trame et du post-game traduits** du japonais vers le
  français (terminologie officielle Square Enix, voir plus bas).
- **Nouvelles recettes de synthèse** pour des monstres présents dans les fichiers
  du jeu mais normalement inobtenables (exclusifs à des évènements Wi-Fi fermés).
- **Interface de patch** conviviale : sélectionnez votre ROM, cochez vos options,
  patchez.
- **Patch anti-piracy appliqué automatiquement** par le patcheur : jouable sur
  console réelle (DS / 3DS) sans aucune manipulation supplémentaire.
- **Options de confort** facultatives : multiplicateur d'XP, niveau minimum de
  synthèse, assouplissement des conditions de dressage.
- **Randomiseur** avec filtrage par rang, famille et taille, XP aléatoire et
  options de challenge.

### La terminologie officielle Square Enix

*Joker 2* (le premier du nom) est sorti en français en 2011. Sa traduction
officielle a été extraite de la version européenne du jeu, puis **transplantée**
automatiquement dans *Professional* : monstres, sorts, objets et lieux portent les
noms que les joueurs français connaissent déjà (« gluant de métal »,
« médigluant », « Championnat du Monde ») plutôt que des inventions. Seul le
contenu inédit à *Professional* a été traduit à la main. (Le projet anglais fait
exactement de même avec la traduction officielle anglaise.)

---

## Bases de données (référence)

Tableaux consultables, générés en **mappant** chaque terme vers la terminologie
officielle française du jeu (donc cohérents avec ce que vous voyez à l'écran) :

- Recettes de [synthèse](Database_FR/synthesis_database.csv)
- [Stats et traits](Database_FR/monster_database.csv) des monstres
- [Résistances](Database_FR/monster_resistance_database.csv) élémentaires et de statut

---

## Installation

1. Procurez-vous **votre propre ROM japonaise** de *Joker 2 Professional*
   (fichier `.nds`, environ 256 Mo). **Aucune ROM n'est distribuée ici**, ni avant
   ni après patch.
2. Lancez le patcheur (`DQMJ2PPatcher-FR.exe`), sélectionnez votre ROM, choisissez
   vos options, patchez. **La traduction, le patch mémoire (arm9) et l'anti-piracy
   sont appliqués automatiquement.**
3. Jouez sur émulateur en **mode DS** (et non DSi) — melonDS ou DeSmuME — ou sur
   une vraie DS / 3DS (custom firmware requis pour la 3DS).

Le guide joueur détaille la procédure pas à pas.

---

## Problèmes connus

- Le **randomiseur** déséquilibre le jeu par nature : trois gluants remplacés par
  trois monstres occupant chacun trois emplacements font neuf adversaires, ce que
  le jeu ne supporte pas toujours. Filtrez les monstres à trois emplacements, ou
  fuyez les combats facultatifs.
- Avec la **randomisation des recettes**, certains résultats de la famille ??? ne
  sont pas synthétisables et peuvent faire planter le jeu à l'affichage. Si un
  résultat apparaît sans nom, ne le sélectionnez pas.
- **Renommer un monstre** dont le nom d'origine dépasse 13 caractères provoque un
  défaut d'affichage du clavier tactile. C'est purement visuel, la saisie
  fonctionne.

---

## Crédits

**Développement technique**
- **Ceris White** : boîte à outils Python de modification de la ROM + injection du
  texte anglais.
- **Saneezore** : interface graphique du patcheur.
- **Mow** : patch anti-piracy (overlay 4).
- **WireOn** : randomiseur, adapté de son [projet DQMJ2](https://github.com/Wire0n-misc/dqmj2-randomizer).

**Traduction et localisation anglaise**
- **Ceris White, Ilario, Reflex** : import du texte anglais et traduction des menus.
- **Gerb** : traduction du post-game.
- **GemSlimee** : relecture et corrections.

**Nouvelles recettes de synthèse**
- **Darko, Hoodiniebobeenie, Anthcny** : création des recettes et équilibrage.

**Traduction française**
- **Sfynx** : traduction (également auteur des VF de *Joker 3*, *Joker 3
  Professional* et *DQM2 3DS*).
- **TheFrenchPepeAI** : traduction, outillage d'extraction et de contrôle,
  transplantation de la version officielle, écrans personnalisés.
- **Reflex** : accueil du projet français au sein du dépôt.

**Testeurs (version anglaise)** : Ilario, Reflex, Gerb, Darko, GemSlimee,
Hoodinibobeenie, Mad Raigo, Matthew McConville, Nurfed, Chris, diortememirp,
Nightaura, Sloppydeck, Anti-Tank Guided Missile, Ghostface, Blark.

**Le serveur Discord [Dragon Quest Translations](https://discord.gg/aX6Ac8cC84)**
pour héberger la collaboration.

---

## Notes techniques (pour les traducteurs)

Le fichier **[ERREURS_CORRIGEES.md](ERREURS_CORRIGEES.md)** détaille, en français
et en anglais, tous les pièges du moteur rencontrés et leur correction (tampon de
combat caché, débordements de tampons, codes de variables, etc.). De quoi gagner
des dizaines d'heures pour une traduction vers une autre langue. En résumé :

- **`grow_msg_pool` est obligatoire** (sinon le jeu se fige après la saisie du nom
  du héros : les textes traduits, plus longs, débordent la mémoire d'origine).
- **Les codes de variables `{E3xx}` doivent être conservés** : le moteur les
  utilise pour résoudre les variables (nom du héros, nom d'objet, accord des
  articles). Les retirer casse leur affichage. `{E32A}` (nom d'objet côté européen)
  doit être réécrit `{E328}` (côté *Professional*).
- **`msg_battle` a un tampon de combat dédié** calibré ~taille anglaise : s'il
  déborde, tout le menu et les messages de combat s'affichent vides. À condenser.

Pour patcher **manuellement** (sans le patcheur graphique), ou sur console réelle
par une autre méthode, un patch anti-piracy doit être pré-appliqué, sinon un
personnage nécessaire à la progression (« Eugene Pool », le vieil homme du
zeppelin) reste introuvable. Ce mécanisme ne se déclenche pas en émulation. **Le
patcheur graphique fourni s'en charge déjà automatiquement.**

---

## Cadre légal

Aucune ROM n'est distribuée, patchée ou non. Ce dépôt ne contient que des outils
et des fichiers de traduction, inutilisables sans une copie du jeu que vous
possédez légalement.

Les outils originaux sont ceux de Ceris White et de l'équipe anglaise, dont la
seule condition d'usage est d'être crédités : elle est respectée ici.
