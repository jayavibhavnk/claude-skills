---
name: procedural-gen
description: Procedural generation for games - noise algorithms, dungeon generation, terrain, and content creation algorithms.
metadata:
  priority: 7
  docs:
    - "https://procgen.dev/"
  pathPatterns:
    - "**/procedural/**"
    - "**/generation/**"
  bashPatterns:
    - '\bprocedural\b'
    - '\bnoise\b'
  promptSignals:
    phrases:
      - "procedural generation"
      - "noise algorithm"
      - "terrain generation"
    anyOf:
      - "procedural"
      - "noise"
      - "dungeon"
---

## Procedural Generation

### Noise Algorithms

```typescript
// Perlin noise implementation
class PerlinNoise {
  private permutation: number[];

  constructor(seed: number = Math.random()) {
    this.permutation = this.generatePermutation(seed);
  }

  private generatePermutation(seed: number): number[] {
    const p = Array.from({ length: 256 }, (_, i) => i);
    // Seeded shuffle
    let s = seed;
    for (let i = 255; i > 0; i--) {
      s = (s * 16807) % 2147483647;
      const j = s % (i + 1);
      [p[i], p[j]] = [p[j], p[i]];
    }
    return [...p, ...p]; // Double for overflow
  }

  private fade(t: number): number {
    return t * t * t * (t * (t * 6 - 15) + 10);
  }

  private lerp(a: number, b: number, t: number): number {
    return a + t * (b - a);
  }

  private grad(hash: number, x: number, y: number): number {
    const h = hash & 3;
    const u = h < 2 ? x : y;
    const v = h < 2 ? y : x;
    return ((h & 1) ? -u : u) + ((h & 2) ? -v : v);
  }

  noise(x: number, y: number): number {
    const X = Math.floor(x) & 255;
    const Y = Math.floor(y) & 255;

    x -= Math.floor(x);
    y -= Math.floor(y);

    const u = this.fade(x);
    const v = this.fade(y);

    const p = this.permutation;
    const A = p[X] + Y;
    const B = p[X + 1] + Y;

    return this.lerp(
      this.lerp(this.grad(p[A], x, y), this.grad(p[B], x - 1, y), u),
      this.lerp(this.grad(p[A + 1], x, y - 1), this.grad(p[B + 1], x - 1, y - 1), u),
      v
    );
  }

  // Fractal Brownian Motion for more natural terrain
  fbm(x: number, y: number, octaves: number = 6): number {
    let value = 0;
    let amplitude = 1;
    let frequency = 1;
    let maxValue = 0;

    for (let i = 0; i < octaves; i++) {
      value += amplitude * this.noise(x * frequency, y * frequency);
      maxValue += amplitude;
      amplitude *= 0.5;
      frequency *= 2;
    }

    return value / maxValue;
  }
}
```

### Terrain Generation

```typescript
// Heightmap-based terrain
interface TerrainConfig {
  width: number;
  height: number;
  scale: number;
  octaves: number;
  persistence: number;
  lacunarity: number;
  seed: number;
}

function generateHeightmap(config: TerrainConfig): number[][] {
  const noise = new PerlinNoise(config.seed);
  const heightmap: number[][] = [];

  for (let y = 0; y < config.height; y++) {
    heightmap[y] = [];
    for (let x = 0; x < config.width; x++) {
      // Normalized coordinates
      const nx = x / config.width;
      const ny = y / config.height;

      // Combine octaves
      let amplitude = 1;
      let frequency = config.scale;
      let height = 0;

      for (let o = 0; o < config.octaves; o++) {
        height += amplitude * noise.noise(nx * frequency, ny * frequency);
        amplitude *= config.persistence;
        frequency *= config.lacunarity;
      }

      heightmap[y][x] = (height + 1) / 2; // Normalize to 0-1
    }
  }

  return heightmap;
}

// Tile type based on height
function getTileType(height: number): string {
  if (height < 0.3) return 'water';
  if (height < 0.4) return 'sand';
  if (height < 0.6) return 'grass';
  if (height < 0.8) return 'forest';
  if (height < 0.9) return 'mountain';
  return 'snow';
}
```

### Dungeon Generation (BSP)

```typescript
// Binary Space Partitioning for dungeon rooms
class BSPNode {
  x: number;
  y: number;
  width: number;
  height: number;
  left: BSPNode | null = null;
  right: BSPNode | null = null;
  room: Room | null = null;

  constructor(x: number, y: number, width: number, height: number) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
  }

  split(minSize: number): boolean {
    if (this.left || this.right) return false;

    const splitH = Math.random() > 0.5;
    const max = (splitH ? this.height : this.width) - minSize;

    if (max <= minSize) return false;

    const split = Math.floor(Math.random() * (max - minSize)) + minSize;

    if (splitH) {
      this.left = new BSPNode(this.x, this.y, this.width, split);
      this.right = new BSPNode(this.x, this.y + split, this.width, this.height - split);
    } else {
      this.left = new BSPNode(this.x, this.y, split, this.height);
      this.right = new BSPNode(this.x + split, this.y, this.width - split, this.height);
    }

    return true;
  }

  createRooms(): void {
    if (this.left || this.right) {
      this.left?.createRooms();
      this.right?.createRooms();
    } else {
      const roomWidth = Math.floor(Math.random() * (this.width - 6)) + 4;
      const roomHeight = Math.floor(Math.random() * (this.height - 6)) + 4;
      const roomX = this.x + Math.floor(Math.random() * (this.width - roomWidth - 1)) + 1;
      const roomY = this.y + Math.floor(Math.random() * (this.height - roomHeight - 1)) + 1;

      this.room = new Room(roomX, roomY, roomWidth, roomHeight);
    }
  }

  getRoom(): Room | null {
    if (this.room) return this.room;

    const leftRoom = this.left?.getRoom();
    const rightRoom = this.right?.getRoom();

    if (!leftRoom && !rightRoom) return null;
    if (!leftRoom) return rightRoom;
    if (!rightRoom) return leftRoom;

    return Math.random() > 0.5 ? leftRoom : rightRoom;
  }
}
```

### Wave Function Collapse

```typescript
// Simple WFC for tile-based generation
interface Tile {
  id: string;
  weight: number;
  connections: Map<string, Set<string>>; // side -> valid neighbor ids
}

class WaveFunctionCollapse {
  private tiles: Tile[];
  private grid: (string | null)[][];
  private entropy: number[][];

  constructor(tiles: Tile[], width: number, height: number) {
    this.tiles = tiles;
    this.grid = Array.from({ length: height }, () =>
      Array(width).fill(null)
    );
    this.entropy = Array.from({ length: height }, () =>
      Array(width).fill(tiles.length)
    );
  }

  private getValidTiles(x: number, y: number): Tile[] {
    const valid = new Set<string>();

    // Check all 4 sides
    const neighbors = [
      { dx: 0, dy: -1, side: 'north', opposite: 'south' },
      { dx: 1, dy: 0, side: 'east', opposite: 'west' },
      { dx: 0, dy: 1, side: 'south', opposite: 'north' },
      { dx: -1, dy: 0, side: 'west', opposite: 'east' },
    ];

    for (const { dx, dy, side, opposite } of neighbors) {
      const nx = x + dx;
      const ny = y + dy;

      if (nx < 0 || ny < 0 || ny >= this.grid.length || nx >= this.grid[0].length) {
        continue;
      }

      const cellValue = this.grid[ny][nx];
      if (cellValue) {
        const tile = this.tiles.find(t => t.id === cellValue);
        const allowed = tile?.connections.get(opposite);
        if (allowed) {
          allowed.forEach(id => valid.add(id));
        }
      }
    }

    return this.tiles.filter(t => valid.has(t.id) || valid.size === 0);
  }

  collapse(x: number, y: number): void {
    const validTiles = this.getValidTiles(x, y);
    if (validTiles.length === 0) return;

    // Weighted random selection
    const totalWeight = validTiles.reduce((sum, t) => sum + t.weight, 0);
    let random = Math.random() * totalWeight;

    for (const tile of validTiles) {
      random -= tile.weight;
      if (random <= 0) {
        this.grid[y][x] = tile.id;
        return;
      }
    }

    this.grid[y][x] = validTiles[0].id;
  }

  generate(): string[][] {
    while (true) {
      // Find lowest entropy cell
      let minEntropy = Infinity;
      let minCell: [number, number] | null = null;

      for (let y = 0; y < this.grid.length; y++) {
        for (let x = 0; x < this.grid[0].length; x++) {
          if (this.grid[y][x]) continue;

          const valid = this.getValidTiles(x, y);
          if (valid.length === 0) {
            // Backtrack or restart
            return this.generate();
          }
          if (valid.length < minEntropy) {
            minEntropy = valid.length;
            minCell = [x, y];
          }
        }
      }

      if (!minCell) break; // Done

      this.collapse(minCell[0], minCell[1]);
    }

    return this.grid;
  }
}
```

### Cave Generation (Cellular Automata)

```typescript
// Cellular automata caves
function generateCave(
  width: number,
  height: number,
  initialFill: number = 0.45,
  iterations: number = 5
): boolean[][] {
  // Initialize with random fill
  let grid: boolean[][] = Array.from({ length: height }, () =>
    Array(width).fill(false).map(() => Math.random() < initialFill)
  );

  // Ensure borders are walls
  for (let x = 0; x < width; x++) {
    grid[0][x] = true;
    grid[height - 1][x] = true;
  }
  for (let y = 0; y < height; y++) {
    grid[y][0] = true;
    grid[y][width - 1] = true;
  }

  // Apply cellular automata rules
  for (let i = 0; i < iterations; i++) {
    const newGrid = grid.map(row => [...row]);

    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        const neighbors = countWallNeighbors(grid, x, y);

        if (neighbors > 4) {
          newGrid[y][x] = true;
        } else if (neighbors < 4) {
          newGrid[y][x] = false;
        }
      }
    }

    grid = newGrid;
  }

  return grid;
}

function countWallNeighbors(grid: boolean[][], x: number, y: number): number {
  let count = 0;
  for (let dy = -1; dy <= 1; dy++) {
    for (let dx = -1; dx <= 1; dx++) {
      if (dx === 0 && dy === 0) continue;
      if (grid[y + dy][x + dx]) count++;
    }
  }
  return count;
}
```

### Best Practices

1. **Seeded randomness** - Reproducible worlds
2. **Chunking** - Generate in chunks for large worlds
3. **Caching** - Store generated chunks
4. **LOD** - Different detail at different distances
5. **Biomes** - Blend between terrain types
6. **Validation** - Ensure playable output
7. **Authoring** - Allow hand-crafted seeds/parameters
