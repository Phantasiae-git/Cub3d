# Cub3d

**A raycasting engine inspired by Wolfenstein 3D (with a trippy twist)**

A 42 curriculum project that started as a classic raycasting renderer but ended up with two extra modes that use the engine's own output as wall textures, creating recursive and feedback-loop visuals that are slightly cursed.

Built from scratch in **C** with **miniLibX**.

---

### Mode 1 — Classic

![](https://github.com/Phantasiae-git/Cub3d/blob/main/cub3d_0.gif)

The OG raycasting with pixel art textures mapped to each wall face. You can walk around the maze and look around

---

### Mode 2 — Recursive (Current Frame)

![](https://github.com/Phantasiae-git/Cub3d/blob/main/cub3d_1.gif)

Instead of static pixel art, the wall textures are the current frame itself. Each cardinal wall direction gets a version of the rendered frame rotated to match its orientation (0°, 90°, 180°, 270° for N, E, S, W). The result is a mirror-house effect.

**How it works:** the DDA raycasting pass runs once for each wall direction to generate 4 rotated frame textures, then one final pass composites the scene using those generated textures as wall surfaces.

---

### Mode 3 — Feedback Loop (Previous Frame)

![](https://github.com/Phantasiae-git/Cub3d/blob/main/cub3d_2.gif)

Instead of the current frame, the wall textures come from the **previous frame**. This creates a temporal feedback loop, and things get very trippy.

In my head it looked like a really cool idea, and it is, but in practice, after a few seconds, the walls practically disappear.Why?

Each frame, the wall texture is a snapshot of the last frame, and that last frame had walls occupying some percentage of its pixels (like 40%), so the new wall TEXTURES are 40% wall and 60% ceiling/floor. Next frame, those textures get rendered on the walls again, and now the wall-within-wall percentage is 40% of 40% = 16%. And so on it shrinks exponentially toward zero, so after some seconds, the textures are almost entirely ceiling and floor colors, and the walls fade into a ghostly soup (the pattern is really cool tho).

**How it works:** same multi-pass DDA approach as Mode 2, but each frame samples from the previous frame buffer instead of the current one.

---

## DDA Raycasting

The rendering engine uses the **DDA (Digital Differential Analyzer)** algorithm, the same technique used in the original Wolfenstein 3D.

For each vertical column of pixels on screen:

1. **Cast ray** from the player's position into the map at the corresponding angle
2. **Walk through the grid** - the DDA algo efficiently walks the ray through map cells, checking for wall hits at each grid cell EDGE
3. **Calculate wall height** - when a wall is hit, the perpendicular distance to the wall determines is inversely proportional to how tall it is
4. **Project texture** - the exact point where the ray hit the wall determines which column of the texture to draw

For Modes 2 and 3, this process runs **multiple times per frame** so it gets pretty laggy

---

## Controls

| Key | Action |
|-----|--------|
| `W` `A` `S` `D` | Move around |
| `Mouse or arrows` | Look around |
| `ESC` | Quit |

---

## Map Format

Maps use the `.cub` file format. A `.cub` file contains texture paths, colors, and the map layout:

```
NO ./textures/north.xpm
SO ./textures/south.xpm
WE ./textures/west.xpm
EA ./textures/east.xpm

F 139,0,0
C 180,150,200

        1111111111111111111111111
        1000000000110000000000001
        1011000001110000000000001
        1001000000000000000000001
111111111011000001110000000000001
100000000011000001110111111111111
11110111111111011100000010001
11110111111111011101010010001
11000000110101011100000010001
10000000000000001100000010001
10000000000000001101010010001
11000001110101011111011110N0111
11110111 1110101 101111010001
11111111 1111111 111111111111
```

- `1` — wall
- `0` — empty space
- `N` `S` `E` `W` — player spawn position and facing direction
- `F` — floor color (R,G,B)
- `C` — ceiling color (R,G,B)
- Texture paths point to `.xpm` files for each wall face

---

## Build & Run

### Prerequisites

- `gcc`
- `make`
- `miniLibX` (included or linked depending on your setup)

### Compile

```bash
make
```

### Run

```bash
./cub3d path/to/map.cub
```
