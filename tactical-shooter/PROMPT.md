# Tactical Shooter: build prompt

Paste the prompt below into a coding agent that can run the game in a browser to test it (for example Claude Code or Cursor). Run it from the repo root. All output goes into this folder.

Target: browser game (HTML5 Canvas, plain JavaScript, no install, no asset files).

```text
ROLE
You are a senior gameplay programmer, pixel artist and sound designer who ships complete, polished indie games. Build a finished, fully playable game in one pass. No placeholders, no TODOs, no stubbed features, no "left for later".

GAME
Working title: NIGHT BREACH (you may keep it).
A tactical top-down shooter in pixel art, played in real time. The player is a lone operator who breaks into hostile buildings at night. Every room is a puzzle: see before you are seen, control noise and light, and win fights in fractions of a second. Deaths are fast and restarts are instant.

REFERENCES (take these specific ideas; do not copy assets, names or levels)
- Hotline Miami: high lethality (the player and basic enemies die in 1 to 2 hits), one-key instant restart, bold readable visuals, combo score for chained kills, camera leans toward the cursor.
- Door Kickers: fog of war from line of sight (the player sees only what the operator sees). Doors are the key tactical space: open quietly, or kick for a loud breach that stuns enemies behind. Flashbangs.
- Intravenous: stealth built on light and noise. Darkness hides the player. Enemies have visible vision cones, hear shots and footsteps, investigate, and call for backup. Suppressed weapons matter.
- Teleglitch: real shadow casting. Walls block vision. Unseen areas are dark and desaturated; remembered layout stays dim.
- Synthetik: weapon depth. Magazine ammo, manual reload, a tactical reload drops the rounds left in the mag, an active reload window gives a faster reload.
- Nuclear Throne (Vlambeer, "The Art of Screenshake"): game feel. Screen shake, hit stop, muzzle flash, recoil kick, casings that stay on the floor, knockback, white hit flash, persistent blood and debris.
- Enter the Gungeon / Hyper Light Drifter: pixel art quality. Clear silhouettes, a strong palette, readable bullets, lively idle animations, detailed rooms.

TECH (strict)
- HTML5 Canvas 2D and plain JavaScript. Zero dependencies, zero build step, zero external asset files, zero network requests.
- Must run by double-clicking index.html (file://) and on GitHub Pages. Use classic <script src> tags loaded in order and one global namespace (window.G). Do not use ES modules or fetch.
- Put every file in /tactical-shooter/ at the repo root. Do not change anything outside this folder, except the entry for this game in the Games table of the root README (set its status to Playable).
- Layout: index.html, style.css, README.md, src/ (config.js, core.js, input.js, audio.js, palette.js, font.js, sprites.js, render.js, lighting.js, map.js, levels.js, player.js, weapons.js, enemies.js, ai.js, fx.js, ui.js, save.js, main.js).
- Fixed 60 Hz simulation with render interpolation. Object pools for bullets, particles and casings. Spatial grid for collisions.
- 60 FPS on a mid-range laptop with 40 enemies and 300 particles active.
- All tuning values live in one CONFIG object in config.js.

ART DIRECTION
- Internal resolution 480x270. Scale to the window by the largest integer factor, letterbox the rest, image smoothing off. Snap sprites to whole pixels when drawing.
- One fixed 32-color palette in palette.js, in the spirit of ENDESGA 32: deep navy and purple shadows, warm skin and wood tones, teal and cyan tech, hot orange muzzle light, blood red. All art and effects use only palette colors.
- All sprites are drawn in code: text pixel maps (one character per pixel, mapped to palette indexes) baked into offscreen canvases at startup. Characters are 16x16 with a 1 px dark outline; legs and upper body are separate so the upper body aims in 16 directions. 4-frame walk, 2-frame idle breathing, death animation, and a corpse that stays.
- 16x16 tiles: floors (concrete, carpet, tile, wood, wet asphalt), walls with a top face and a front face for depth, doors (closed, open, broken), breakable windows, props (desks, crates, shelves, sofas, cars, server racks, plants).
- Lighting: dark night scenes. Lamps, neon signs, muzzle flashes, explosions and enemy flashlights add colored light. Use a light buffer with multiply blending and 4x4 Bayer dithering on falloff so gradients stay pixel-styled.
- Line of sight: each frame, build a visibility polygon from the player by ray casting to wall segment endpoints. Outside it, draw the remembered map dark and desaturated, and do not draw enemies.
- Persistent decals: blood, scorch marks, bullet holes, casings, broken glass.
- Post effects, each toggleable: soft vignette, light scanlines, short chromatic flash on large explosions.
- A built-in bitmap pixel font (define the glyphs in font.js). A custom pixel crosshair that widens with spread; hide the system cursor over the canvas.

CONTROLS
Keyboard and mouse:
- WASD move, mouse aim, left click fire.
- Hold right click: aim mode (camera pushes toward the cursor, tighter spread, slower walk).
- Hold Shift: sprint (loud). Hold C: sneak (silent, slow). Do not use Ctrl (browser shortcuts).
- R: reload; press R again inside the active reload window for a fast reload.
- E: context action (open or close a door quietly, pick up, silent takedown from behind an unaware enemy).
- F: kick door. G: hold to show the throw arc and landing point, release to throw. Q: swap grenade type.
- 1 primary, 2 sidearm, mouse wheel swaps.
- Space: tactical pause (time freezes; you can look around and see enemy last known positions; you cannot act).
- After death, R restarts the mission in under 0.5 s. Esc opens the pause menu.
- preventDefault on all game keys.
Gamepad (Gamepad API): left stick move, right stick aim, RT fire, LT aim mode, standard mapping for the rest. Detect the active device and swap on-screen button prompts.

CORE LOOP
- Session: pick a mission, pick a loadout, breach, complete objectives, reach extraction, get a grade, unlock gear, next mission.
- Every 30 seconds: scout a door, choose quiet or loud, clear a room.
- Missions take 2 to 5 minutes. Death restarts the mission. Long missions get one mid-point checkpoint.

MECHANICS
Player:
- 3 hits of health. When hit: red edge flash, 60 ms hit stop, a damage direction marker. No regen; some maps have medkits (+1).
- Optional armor vest in the loadout: +1 hit, -10% move speed.
- Speeds (starting values, tune): sneak 45 px/s, walk 80, sprint 125.
Noise:
- Every action emits noise with a radius in tiles: sneak 0, walk 2, sprint 5, suppressed shot 3, normal shot 14, door kick 10, glass break 8, explosion 20.
- Noise spreads by flood fill through open tiles and open doors, so it does not pass through solid walls.
Weapons (6, unlocked over the campaign): suppressed pistol, revolver, SMG, pump shotgun (pellets, knockdown, breaks doors), assault rifle, DMR (goes through 1 enemy).
- Each has damage, fire rate, magazine size, reload time, spread, recoil, noise and penetration in CONFIG.
- Fast visible projectiles (not hitscan) with a 1-frame tracer. Sparks and bullet holes on walls. Glass breaks.
- Sustained fire widens spread; it recovers when you stop firing. Aim mode reduces spread.
- A tactical reload drops the remaining rounds. Active reload: a window of 20% of the reload bar; hitting it makes the reload 40% faster, missing it adds a short delay. Show a small bar next to the player.
Grenades:
- Flashbang: blinds and stuns enemies within 5 tiles who have line of sight to it, for 3 s. It blinds the player too if they face it (white fade) and muffles audio with a low-pass filter.
- Frag: kills within 2.5 tiles, damages up to 4 tiles, blocked by walls, breaks glass, very loud.
Doors:
- A quiet open takes 0.4 s. A kick is instant and loud and stuns enemies within 2 tiles behind the door for 1.5 s. Closed doors block vision and bullets.
Stealth takedown:
- Hold E for 0.8 s behind an unaware enemy. Small noise. Enemies who find a body become alerted.
Civilians and hostages:
- Civilians panic and flee. Hurting one costs score and lowers the grade. Killing a hostage fails the mission.
Score and grade:
- Points for kills, a combo multiplier for kills within 2.5 s of each other, a stealth bonus (never detected), a time bonus (under par) and an accuracy bonus. Grades S, A, B, C, D.

ENEMIES AND AI
Types:
1. Thug: pistol, patrols, weak.
2. Rifleman: burst fire, uses cover.
3. Shotgunner: rushes when alerted, deadly up close.
4. Heavy: armored (4 small-arms hits), slow, suppressive fire; shotgun and DMR counter it.
5. Marksman: long range, shows a red laser for 0.8 s before each shot.
States: Patrol (waypoints, looks around) > Suspicious ("?" icon, investigates the noise or glimpse) > Alerted ("!" icon, engages, shouts to alert others nearby) > Searching (goes to the last known position, sweeps nearby rooms, returns to patrol after 20 s). Also Stunned and Blinded.
Vision:
- 90 degree cone, 9 tiles in light, 4 tiles if the player stands in darkness, 12 tiles for the marksman.
- A detection meter fills faster when the player is closer and lit; show it as a small arc above the enemy.
- Draw vision cones on the floor, clipped by walls, translucent, only for enemies the player can see (toggleable).
Fairness:
- Reaction time before the first shot after spotting: thug 0.35 s, rifleman 0.25 s.
- Enemy bullets are visible and slightly slower than player bullets. Enemies firing from darkness always show a muzzle flash.
Pathfinding: A* on the tile grid with doors as nodes. Limit path recalculations per frame.

CONTENT
- Tutorial mission with short contextual prompts: movement, doors, stealth, reload, grenades.
- 8 handcrafted missions stored as ASCII tile maps with a legend in levels.js, each 40x30 to 70x50 tiles:
  1. Warehouse: eliminate all hostiles.
  2. Office floor: recover the intel laptop, then extract.
  3. Neon nightclub: loud music halves all noise radii.
  4. Docks in the rain: rain particles, puddles that reflect lights, destroy 3 weapon crates.
  5. Apartment block: rescue 2 hostages who then follow the player to the exit.
  6. Server farm: a power switch darkens the whole level; enemies switch to flashlights.
  7. Mansion, stealth: an alarm panel brings reinforcements if triggered.
  8. Penthouse finale: a boss (armored leader with 3 phases), then a win screen and credits.
- Each mission has an objective, a par time and an optional challenge (for example "No alarms").

PROGRESSION AND SAVE
- Missions unlock in order. Grades unlock weapons and gear; show what unlocks where.
- Loadout screen: primary, sidearm, 2 grenade slots, armor on or off, with stat bars.
- Save to localStorage: unlocked missions, best score and grade per mission, unlocks and settings. "Reset progress" in Settings, with confirmation.

UI AND HUD
- Title screen: an animated pixel city skyline at night with rain and a pixel logo. Menu: Play, Settings, Credits.
- Mission select with mission cards, grade stamps and lock icons.
- HUD (minimal, pixel font): health pips, ammo "12 / 48" with magazine icons, grenade icons, current objective, combo counter, stealth status (Hidden, Suspicious, Alerted), hit markers.
- Pause menu: Resume, Restart, Settings, Quit to menu.
- Results screen: time, kills, accuracy, stealth, civilians, score, and an animated grade stamp.
- Settings: master, music and SFX volume; screen shake 0 to 100%; hit flash on or off; each post effect on or off; vision cones on or off; gamepad aim assist; fullscreen. All settings persist.
- Every screen works with mouse, keyboard and gamepad.

AUDIO (WebAudio, fully synthesized, no files)
- A distinct shot per weapon (noise burst + low sine thump + short tail; suppressed = filtered click), reload clicks, casing tinkles, footsteps per floor type, door open and kick, glass break, explosion, flashbang ring, short synthesized enemy shouts, UI blips, grade stamp.
- Procedural dark synthwave music: menu, stealth and combat layers. Crossfade to combat when any enemy is alerted.
- A master compressor to prevent clipping. Start audio on the first user input.

GAME FEEL (all required)
Screen shake with decay and per-weapon strength; 20 to 50 ms hit stop on kills; muzzle flash sprite plus a 1-frame light burst; 2 px weapon recoil kick; casings that bounce and stay; enemy knockback and white hit flash; blood spray plus persistent decals; wall sparks and dust; camera lead toward the cursor (stronger in aim mode); smooth camera follow; 0.3 s slow motion on the last kill of a mission; a death animation for every enemy type.

WORK PLAN (in order; run and test after each step)
1. Core: loop, input, camera, integer scaling, palette, font, tile map, collisions.
2. Vertical slice: player, one weapon, the thug with all AI states, doors, lighting, line of sight, full game feel, one small map. Make this feel great before you continue.
3. All weapons, grenades, enemy types, stealth, noise.
4. All missions, objectives and the boss.
5. Menu flow, progression, save, settings, audio and music.
6. Polish: visual consistency, tuning balance, performance, bug sweep.
7. Final check against the Definition of Done. Fix every failure before you finish.

DEFINITION OF DONE
- Runs from index.html with zero console errors or warnings.
- The tutorial and all 8 missions are playable and beatable; the finale ends with a win screen and credits.
- No placeholder shapes, no TODOs, no unused stubs, no lorem ipsum.
- Every menu is reachable, every setting works and persists, restart is instant.
- Stable 60 FPS at the target load.
- /tactical-shooter/README.md covers how to run, controls, features and credits.

If you must cut scope to finish, cut in this order and list what you cut: missions 6 to 8 (keep the tutorial plus 5), gamepad support, post effects. Never cut game feel, lighting, line of sight, AI states or the menu flow.
```
