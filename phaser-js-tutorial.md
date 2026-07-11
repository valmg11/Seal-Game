# Phaser Crash Course: Games in HTML/JS

Phaser is a free, open-source JavaScript framework for building 2D games that run directly in a browser tab — no installs, no app store, just an HTML file. This guide covers the fastest path to a working game: setup, loading assets, sprites, motion, and input.

**Version note:** As of mid-2026, Phaser 4 is the current major version. The core API covered here (scenes, sprites, physics, input) is nearly identical to Phaser 3, so this guide — and the huge pile of Phaser 3 tutorials online — works either way.

---

## 1. Setup: One HTML File Is All You Need

Phaser loads from a CDN like any other script. No build tools required to get started.

```html
<!DOCTYPE html>
<html>
<head>
  <title>My Phaser Game</title>
  <script src="https://cdn.jsdelivr.net/npm/phaser@4.2.1/dist/phaser.js"></script>
</head>
<body>
<script>
  const config = {
    type: Phaser.AUTO,        // use WebGL, fall back to Canvas automatically
    width: 800,
    height: 600,
    physics: {
      default: 'arcade',       // simple, fast physics — good for most 2D games
      arcade: { gravity: { y: 300 }, debug: false }
    },
    scene: {
      preload: preload,
      create: create,
      update: update
    }
  };

  const game = new Phaser.Game(config);

  function preload() { }
  function create() { }
  function update() { }
</script>
</body>
</html>
```

Save this, open it in a browser, and you have a blank Phaser canvas ready to go. Everything below just fills in `preload`, `create`, and `update`.

---

## 2. The Three Functions That Run Your Game

Every Phaser scene is built around three lifecycle functions:

| Function | Runs | Use it for |
|---|---|---|
| `preload()` | Once, before anything else | Loading images, spritesheets, audio |
| `create()` | Once, right after preload finishes | Building your world: adding sprites, setting up input |
| `update()` | Every frame (~60x per second) | Movement, input checks, anything ongoing |

**The #1 beginner mistake:** checking keyboard input inside `create()`. It runs once, so your character will never move. Input checks belong in `update()`.

---

## 3. Handling Assets

`preload()` is where every image, spritesheet, and sound gets loaded. Phaser waits for loading to finish before `create()` runs, so you never have to manage that yourself.

```js
function preload() {
  this.load.image('sky', 'assets/sky.png');
  this.load.image('player', 'assets/player.png');

  // A spritesheet is one image file cut into equal-size frames,
  // used for animation (walk cycles, etc.)
  this.load.spritesheet('dude', 'assets/dude.png', {
    frameWidth: 32,
    frameHeight: 48
  });
}
```

- The first argument (`'sky'`, `'player'`, `'dude'`) is a **key** — a nickname you make up, used to reference that asset later.
- The second argument is a file path, relative to your HTML file. Put an `assets/` folder next to it and drop your images in.

> **Gotcha:** if you just double-click the HTML file, some browsers block local image loading for security reasons (`file://` restrictions) and your assets silently fail. Run a tiny local server instead — `python -m http.server` in the project folder, or the VS Code "Live Server" extension both work.

---

## 4. Sprites

A **sprite** is anything drawn on screen that can move — your player, enemies, coins. Phaser gives you three ways to put an image on screen, and the difference matters:

| Method | Has a physics body? | Use for |
|---|---|---|
| `this.add.image()` | No | Static background art |
| `this.add.sprite()` | No | Animated but non-physical (menu decorations) |
| `this.physics.add.sprite()` | **Yes** | Anything that moves, falls, or collides |

```js
function create() {
  this.add.image(400, 300, 'sky');                            // background, never moves
  this.player = this.physics.add.sprite(100, 450, 'player');   // has a physics body
}
```

**Coordinates:** `(0, 0)` is the top-left corner. X increases to the right, and **Y increases downward** — the opposite of a math-class graph. This trips everyone up at least once, so commit it to memory now.

**Bonus — animating a spritesheet:**

```js
this.anims.create({
  key: 'walk',
  frames: this.anims.generateFrameNumbers('dude', { start: 0, end: 3 }),
  frameRate: 10,
  repeat: -1          // -1 = loop forever
});

this.player.play('walk');
```

---

## 5. Motion

Anything made with `this.physics.add.sprite()` has a `.body` with velocity, gravity, and collision. You move things by setting **velocity**, not by manually changing x/y — that way the physics engine handles smooth, frame-rate-independent motion for you.

```js
player.setVelocityX(200);      // move right at 200 px/sec
player.setVelocityY(-330);     // move up (negative = up, since Y is flipped)
player.setVelocity(0, 0);      // stop completely

player.setCollideWorldBounds(true);   // can't walk off the edge of the screen
player.setBounce(0.2);                // bounce a little on landing
```

**Gravity** is set globally in your config (`arcade: { gravity: { y: 300 } }`), so every physics sprite falls automatically — perfect for platformers. For a top-down game where nothing should fall, just set gravity to `0`.

**Platforms that don't move or fall:**

```js
const platforms = this.physics.add.staticGroup();
platforms.create(400, 584, 'ground');

this.physics.add.collider(player, platforms);   // makes them solid to each other
```

---

## 6. User Input

The most common input is the keyboard, read every frame inside `update()`.

```js
function create() {
  this.cursors = this.input.keyboard.createCursorKeys();   // arrow keys, ready to use
}

function update() {
  if (this.cursors.left.isDown) {
    this.player.setVelocityX(-200);
  } else if (this.cursors.right.isDown) {
    this.player.setVelocityX(200);
  } else {
    this.player.setVelocityX(0);
  }

  // Only allow jumping when standing on something (stops mid-air jumps)
  if (this.cursors.up.isDown && this.player.body.touching.down) {
    this.player.setVelocityY(-380);
  }
}
```

**Want WASD too?**

```js
this.wasd = this.input.keyboard.addKeys('W,A,S,D');
// then check this.wasd.A.isDown, this.wasd.D.isDown, etc.
```

**Mouse/touch**, if you need it:

```js
this.input.on('pointerdown', (pointer) => {
  console.log('Clicked at', pointer.x, pointer.y);
});
```

---

## 7. Putting It All Together

A complete, runnable example — a square that walks left/right and jumps on a platform. Copy this into an HTML file and open it. It needs zero image files, since it draws its own shapes on startup instead of loading them.

```html
<!DOCTYPE html>
<html>
<head>
  <title>My First Phaser Game</title>
  <script src="https://cdn.jsdelivr.net/npm/phaser@4.2.1/dist/phaser.js"></script>
  <style> body { margin: 0; display: flex; justify-content: center; background: #1a1a1a; } </style>
</head>
<body>
<script>
  const config = {
    type: Phaser.AUTO,
    width: 800,
    height: 400,
    physics: {
      default: 'arcade',
      arcade: { gravity: { y: 500 }, debug: false }
    },
    scene: { create, update }
  };

  let player, cursors, platforms;
  new Phaser.Game(config);

  function create() {
    // Draw two colored rectangles and register them as textures —
    // this stands in for this.load.image() so the example needs no asset files.
    // In a real project, swap this block for this.load.image() inside preload().
    const g = this.add.graphics();

    g.fillStyle(0x3aa8ff, 1);
    g.fillRect(0, 0, 32, 48);
    g.generateTexture('player', 32, 48);
    g.clear();

    g.fillStyle(0x2ecc71, 1);
    g.fillRect(0, 0, 800, 32);
    g.generateTexture('ground', 800, 32);
    g.destroy();

    platforms = this.physics.add.staticGroup();
    platforms.create(400, 384, 'ground');

    player = this.physics.add.sprite(100, 300, 'player');
    player.setCollideWorldBounds(true);
    player.setBounce(0.1);

    this.physics.add.collider(player, platforms);

    cursors = this.input.keyboard.createCursorKeys();
  }

  function update() {
    if (cursors.left.isDown) {
      player.setVelocityX(-200);
    } else if (cursors.right.isDown) {
      player.setVelocityX(200);
    } else {
      player.setVelocityX(0);
    }

    if (cursors.up.isDown && player.body.touching.down) {
      player.setVelocityY(-380);
    }
  }
</script>
</body>
</html>
```

Arrow keys move it, up jumps, gravity pulls it back down onto the green platform. That's the core loop underneath a huge number of 2D games.

---

## Quick Reference / Cheat Sheet

| I want to... | Code |
|---|---|
| Load an image | `this.load.image('key', 'path.png')` |
| Load a spritesheet | `this.load.spritesheet('key', 'path.png', {frameWidth, frameHeight})` |
| Add a static image | `this.add.image(x, y, 'key')` |
| Add a physics sprite | `this.physics.add.sprite(x, y, 'key')` |
| Move right / left | `sprite.setVelocityX(200)` / `(-200)` |
| Jump | `sprite.setVelocityY(-330)` |
| Stop at screen edges | `sprite.setCollideWorldBounds(true)` |
| Make two things solid | `this.physics.add.collider(a, b)` |
| Detect overlap, no bounce | `this.physics.add.overlap(a, b, callback)` |
| Read arrow keys | `this.input.keyboard.createCursorKeys()` |
| Check if grounded | `sprite.body.touching.down` |
| Check a key each frame | `cursors.left.isDown` |
| Loop an animation | `this.anims.create({key, frames, frameRate, repeat: -1})` |

---

## Where to Go Next

- **Official examples:** [labs.phaser.io](https://labs.phaser.io) — hundreds of tiny runnable demos, easy to copy-paste from.
- **Docs:** [docs.phaser.io](https://docs.phaser.io)
- Once a single HTML file starts feeling cramped, look into a bundler (Vite is the current standard) and split your scenes into separate files. Same API, better organization.
