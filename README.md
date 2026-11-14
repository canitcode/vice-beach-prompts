# Vice Beach: the prompts

Every system in the film, as the prompt we wish we had on day one. Each one is what many small steps taught us, so
treat it as a first step in your own game: run it, look at the result, and tell the model what is wrong. They are not
tied to Unreal: say your engine in the first line.

The film shows one card a chapter; these are the full ones. Four of them (the sea, the sand, the footprints, the
crashes) are a little more exact here than on their cards: we read every prompt against the game's code after the
film was cut.

The step-by-step version of each system, with the numbers, the mistakes and the code, is the skills pack for Claude
Code and Codex on Patreon: https://www.patreon.com/CanItCode

## Act 1: the postcard

### The sky

Match my game's sky to a reference frame. Measure the reference first: the open sky between the clouds, the lit cloud
tops, the haze toward the sun and the sunlit ground, in sRGB. Use a physical sky atmosphere and volumetric clouds with
multiple scattering (a two-lobe phase, two scattering octaves, ground bounce) and a warm white balance. Film our sky
from the reference's camera, compare the same spots side by side, fix it by the numbers first, then by eye.

### The sea

Make an ocean for my game: a flat grid mesh whose vertices are moved in the vertex shader by 16 Gerstner waves, 24 m
long down to about 2 m (each 16% shorter), directions within about 45 degrees of the way the sea comes in, random
phases, steepness 0.075 times a height scale I can turn (start at 0.35). Fade the short waves out in shallow water,
and add a separate surf model near the shore where the waves grow and break.

### The sea in pieces

My ocean is one huge mesh, drawn every frame even when the camera looks away. Split it into about 30 tiles the engine
can cull one by one. Keep small cells only near the player and let them grow further out. Grow each tile's bounds by
the wave height so tiles don't pop, and share the edge vertices between tiles so no cracks open. Measure the frame time
before and after.

### The sand

Write one sand material for my whole beach, no texture files, only noise. Drive everything by the height above sea
level: wet, dark and glossy under about 10 cm, drying out by 30 cm, a sparse line of seaweed clumps around 50 cm,
dry sand above with soft wind ripples only in wind-swept patches. Make the small relief real displacement
(tessellation) and fade it out between 15 and 40 m from the camera, so it costs a millisecond or two. Measure the
frame time before and after.

### The footprints

Add real footprints to my sand. Keep a render target around the player, about 1.2 cm per pixel, as a pressure map. When a
foot lands, stamp a foot-shaped mask into it. In the sand material, displace the surface down by up to 3 cm where it is
pressed (tessellation or a dense mesh), and let the waves slowly fade the map back to zero.

### If the shallow water goes dark near the camera (the dark stripe)

In Unreal the sky atmosphere is a planet, and its ground can sit at sea level. With "per pixel atmosphere
transmittance" on the sun, everything below that ground counts as shadowed by the planet, so the sea bed and the wave
troughs lose the sun in patches that change with the camera. Ask the model: "The sand under my shallow water goes dark
and light again as the camera moves. Check whether the sun's per-pixel atmosphere transmittance treats everything below
the sky atmosphere's ground level as shadowed, and show me the same frame with it off." Then look at the frame yourself.

## Act 2: the city

### The prop factory

Build me a prop factory for my game. I keep a list: for every prop its name, its real size in metres, a short
description, a few reference photos and where it goes. For each one, have an image model draw it from the front and
from the side on a plain background (the same scale in both views, no shadows), send those views to Tripo to get a clean
low-poly model with the face count I ask for, then in Blender scale it to the real size, turn its front the right way,
put its origin on the ground and pack all props of one zone into one texture atlas at one texel density. Make every
step resume where it stopped, so a night of many props survives one failure.

### Placing the props by rules

Place my props from their list by rules, never by hand: "at" a point, in a "row" every N metres, "scatter" N in an
area, "near" another thing. Stand every prop where a trace down hits the ground, keep a footprint circle round it so
nothing lands on anything, and log how many of each kind landed. Typical rules: a parking meter at every painted
parking stall, two chairs at every cafe table, a trash can every block.

### Buildings from a kit, not from a generator

Do not generate whole buildings. Build a small kit of wall pieces in Blender on a 3 m grid (a ground floor piece with
a shop window, one with a door, floor pieces with different windows, a roof edge), with named anchor points for
details. A building is then a recipe: rows of piece names and a few colours. Use one tinted stucco material for every
building. Generate only the small things for the anchors (air conditioners, lamps, signs, chairs) and place them by
the recipe.

### Neon from a drawing

Turn a drawing of a neon sign into real 3D neon. Take the picture (lit tubes on black), keep only the bright cores,
thin them to one-pixel lines, walk those lines into paths between ends and junctions, drop the short spurs and smooth
the paths. Sweep a round tube along every path and give it the colour of the glow around it in the picture. Make the
tubes emissive, bright enough to read in daylight, and dim them when the scene gets dark (tie their strength to the
exposure), or they will light the whole street in their colour.

### A room behind every window

Put a fake room behind every window of my buildings (interior mapping). Use one picture of a room in one-point
perspective, its back wall the middle half of the picture. In the glass material, follow the camera ray into a box
behind the window and sample the picture where the ray hits a wall, the floor, the ceiling or the back. Make the box
exactly as wide and tall as the window and about as deep as it is wide (a box as big as the whole wall shows the
street only its ceiling: one flat colour). Pack several rooms into one atlas, pick one per window, leave some dark,
and never let the atlas stream down to small mips.

## Act 3: life

### A walk test that looks at the picture

Write a walk test for the crowd in my game that judges what the player sees, not what the code says. For every
walker, take its position at two moments: that is the way it really goes. Take the way it faces from the model
itself: a point on the nose minus a point on the back of the head. Fail when the two are more than 30 degrees apart. Then
film five walkers from the side and look at the film yourself before you call it a pass.

(Ours all walked backwards, and the first test passed: it had the same mistake as the code it checked.)

### People from AI, all on one skeleton

Make the people of my game with AI, all on one skeleton. Start from one grey T-pose mannequin. For every person,
have an image model draw the clothes, hair and face onto that mannequin from the front, the side and the back, on a
plain background. Send the three views to Tripo's multiview (quads, about 10,000 faces) and build four versions.
Pick one by its head in plain clay, without the colours: a face can come out melted, and the colours hide it. They
all started as the same mannequin, so one skeleton and one set of animations fits everybody.

### Moves from a sentence

Make the animations of my crowd with a text-to-motion model (we used NVIDIA's Kimodo). One sentence for each move,
written like this: "A person lying on their back on the floor holds a book up above their face with both hands and
reads." Put every clip onto the crowd's one skeleton and make it loop. Before a clip goes into the game, play it on a
person and watch the whole loop: a move can come out as something you did not ask for, and one move given to
everybody who lies on a beach looks like a gym class.

### Two thousand people in three milliseconds

Draw two thousand animated people in my game in about 3 ms. Give them no skeleton at run time. Play every animation
once on the shared skeleton and bake it into ONE texture: a row per frame, three texels per bone. Skin the people in
the vertex shader from that texture, so each person is a plain instanced mesh and any person can play any move. Add
three distance levels of every model. Measure the frame time with the people hidden and shown, in one run.

(A texture for every person's every move is the simple way and it does not scale: ours would have been about 9 MB a
pair, 36 people x 33 moves = about 10 GB of graphics card memory. The one bone texture is 13 MB.)

### Getting into a car, GTA style (build it in a lab first)

Add GTA-style car entry to my game, and build it in a lab first: one car, one driver, the player, nothing else. Key
every clip to the car: the hand to the door handle along the door's arc, the hips to the seat, the grab to the
driver's collar. Let the game's own locomotion do every step of walking and play clips only where he stands in place.
Turn foot placement off while a clip holds his body. Add a key that freezes the world and a key that steps one frame,
and watch the whole sequence that way before you call it done.

### A car that drives on four springs

Give my game's car its own small simulation: the body on four springs, tyres that grip up to a limit and slide past
it, the engine through a gearbox to the rear wheels. Step the physics in equal steps that add up to exactly the
frame, and move the chase camera after the car in the same frame. Keep the collision box well above the road: the
wheels carry the car, the box only meets walls and other cars. On keys, turn the front wheels toward where the car is
sliding, so a drift can be held. Drive it with a bot on a fixed route and log its height and its wobble against the
camera.

### Soft-body crashes that sleep until a hit

Add soft-body crash damage to my game's cars without paying for it all the time. Keep the cars on a cheap rigid body.
When one hits something faster than about 8 km/h, hand that car to a node-and-beam solver for a second and a half,
with what stands around it as walls; then freeze the bent shape and drive on. Give an open car a frame under its
floor, or it folds at the firewall. Test it in a lab: one speed, one barrier, measure the dent and how far the cabin
moves.

### A motorbike that leans

Add a motorbike to my game. Do not reuse the car's steering: the keys lean the bike, and the lean sets how fast it
turns (tan(lean) = speed x turn rate / g), inside the tyres' grip. Cut the throttle as it leans, hold it upright when
nobody steers, damp the springs so a kerb cannot launch it. To stop it for the rider use the brakes only, never the
reverse key. Test it with a bot that holds the keys all the way down, not a gentle one.

## Act 4: chaos

### Holding a weapon (the weapon first, the hands after)

Give my game's character a pistol, a rifle and a rocket launcher. Place each weapon by the body, never by an
animation's hand: the rifle's butt in the shoulder, the pistol at arm's length under the eyes, the launcher on the
shoulder. Then reach both arms to its grips with two-bone IK and close the fingers round them. Keep the barrel's line
clear of the head. Test it in a lab: aim, reload and walk, and measure how far the palms are from the grips.

(An animation model gives you a body that holds nothing: its wrists and fingers carry no grip. A weapon seated in the
clip's hand points wherever that wrist points.)

### Weapon effects from a reference, played by particles

Build each weapon effect of my game from a reference. Find the effect in real footage and go through it frame by
frame: what appears, in what order, for how long. Generate its parts as short videos on pure black, cut them into
flipbook sheets, and play the sheets on particles. Never play a whole video on one card: it turns with the camera. A
blast is hundreds of particles in 3D: fire puffs that cool from white to red, smoke, debris meshes, sparks.

(What the frames told us: a muzzle flash lasts one or two frames at 60 a second, a hit on sand 0.7 s, a rocket's
fireball turns from white to red in 0.3 s and its smoke comes after half a second. A muzzle flash is not a flipbook:
take each burst of the clip as a variant and show one per shot.)

### Buildings that break, inside the frame

Make the buildings of my game break. Cut each one into pieces ahead of time, and let the pieces hold each other. A
blast frees the pieces near it; any piece that no longer reaches the ground through held pieces falls. Then keep it
inside the frame: break only the windows a blast can see, freeze rubble that has come to rest, let floors fall as big
pieces. Measure the milliseconds of a frame before and after each cut.

(A model made to be looked at is a shell: loose parts, open edges. Before you cut it, rebuild it as one closed solid
and bake its old texture onto it. Left to itself, the physics engine let our whole wall go at the first hit: the game
decides which pieces are free, the engine only moves them.)

### A crowd that runs from one map

When a shot is fired in my game, the crowd runs. Do not path-find per person. Build one grid of the map where every
cell knows the way to safety: flood it from the exits, walls cost more. Each person follows the cell under them.
Rebuild the grid on a worker thread when the shooter moves, so the way out goes round him. After a minute of quiet,
walk everybody back the same way to where they sat.

(Move each runner exactly as far as their run clip's own root motion, or the feet slide. Check the run clip itself
first: every "panic run" our animation model made glided along the ground.)

### A police car that chases (build it in a lab first)

Write a pursuit driver for my game on the same car physics the player has. Far away it cuts toward him; close, it
follows his trail, brakes for the bends ahead and keeps a gap measured in seconds. Give it a grid of where a car fits,
baked once, and find its way on that grid on a worker thread. Test it in a lab first: a bot drives the player's car
round a loop, the police car starts behind it and has to close the gap.

(Our first police car never closed the gap: it had the same tyres and the same engine as the car it chased. It needs
more grip and more power than the player, or it only ever follows. Aiming at a point ahead of the player cut his
corners and rammed his side: close up, drive his trail.)

### A wanted level from events

Give my game a wanted level made of events, not scripts. Each crime raises the stars: a shot near people, a person
hit, an explosion, an officer hit. Police appear on roads outside the camera's view. Each unit searches its own spot
round where the player was last seen, never the same point. Stars fall only after the player stays unseen outside the
search area for a set time. Test it with a bot that hides, a bot that stands still, and one planted fault each.

(Sent to one point, our police cars piled up on it and climbed onto each other. And "seen" has to forgive: within a
few metres a unit sees the player whatever stands between, or he walks past a police car behind a parasol and is lost.)

### When the frame rate drops, measure

My game's frame rate drops when the police arrive. Do not guess. Measure the game thread, the render thread and the
GPU separately. Change one thing per run and compare inside the same run. Then look for anything that rebuilds or
re-sends data that did not change, every frame: instance buffers, render proxies, physics bodies that never go to
sleep.

(Ours were three different things with one symptom: every police car rebuilt its whole set of smoke and tyre-mark
instances every frame; a three-second freeze was the engine writing an error report; and a hundred bodies lying on
the sand never went to sleep in the physics.)

### Generated characters on the player's skeleton

Put every generated character of my game on the player's own skeleton: animate one hidden body and let the skin
follow its bones. After every import, check each bone's position, rotation and scale against the reference skeleton.
Then look at a rendered picture of the character moving: a check can pass while the picture is wrong.

(Our officers got out of their car as two pairs of hands: every bone was in the right place and a hundred times too
big. The check measured places and angles, not sizes.)
