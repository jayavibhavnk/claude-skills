---
name: game-physics
description: Game physics implementation - rigid body dynamics, gravity, friction, springs, projectiles, and 2D physics simulation.
metadata:
  priority: 7
  docs:
    - "https://developer.mozilla.org/en-US/docs/Games/Tutorials/2D_breakout_game_Phaser"
  pathPatterns:
    - "**/physics/**"
    - "**/game/**/physics*.ts"
  bashPatterns:
    - '\bphysics\b'
    - '\bgravity\b'
  promptSignals:
    phrases:
      - "game physics"
      - "rigid body"
      - "collision"
    anyOf:
      - "physics"
      - "gravity"
      - "velocity"
---

## Game Physics

### Basic Physics Types

```typescript
interface Vector2 {
  x: number;
  y: number;
}

interface PhysicsBody {
  position: Vector2;
  velocity: Vector2;
  acceleration: Vector2;
  mass: number;
  restitution: number; // Bounciness 0-1
  friction: number;
  isStatic: boolean;
}

interface RigidBody extends PhysicsBody {
  rotation: number;
  angularVelocity: number;
  inertia: number;
}

interface AABB {
  min: Vector2;
  max: Vector2;
}
```

### Vector Math

```typescript
// Vector2 utilities
const Vec2 = {
  add: (a: Vector2, b: Vector2): Vector2 => ({ x: a.x + b.x, y: a.y + b.y }),
  sub: (a: Vector2, b: Vector2): Vector2 => ({ x: a.x - b.x, y: a.y - b.y }),
  scale: (v: Vector2, s: number): Vector2 => ({ x: v.x * s, y: v.y * s }),
  dot: (a: Vector2, b: Vector2): number => a.x * b.x + a.y * b.y,
  cross: (a: Vector2, b: Vector2): number => a.x * b.y - a.y * b.x,
  length: (v: Vector2): number => Math.sqrt(v.x * v.x + v.y * v.y),
  normalize: (v: Vector2): Vector2 => {
    const len = Vec2.length(v);
    return len > 0 ? Vec2.scale(v, 1 / len) : { x: 0, y: 0 };
  },
  rotate: (v: Vector2, angle: number): Vector2 => ({
    x: v.x * Math.cos(angle) - v.y * Math.sin(angle),
    y: v.x * Math.sin(angle) + v.y * Math.cos(angle),
  }),
  distance: (a: Vector2, b: Vector2): number => Vec2.length(Vec2.sub(a, b)),
  lerp: (a: Vector2, b: Vector2, t: number): Vector2 => ({
    x: a.x + (b.x - a.x) * t,
    y: a.y + (b.y - a.y) * t,
  }),
};
```

### Gravity & Forces

```typescript
interface Force {
  position: Vector2;      // Point of application
  direction: Vector2;    // Direction
  magnitude: number;     // Strength
}

class PhysicsWorld {
  private bodies: PhysicsBody[] = [];
  private readonly gravity: Vector2 = { x: 0, y: 9.81 };

  addBody(body: PhysicsBody): void {
    this.bodies.push(body);
  }

  applyForce(body: PhysicsBody, force: Vector2): void {
    if (body.isStatic) return;
    body.acceleration = Vec2.add(
      body.acceleration,
      Vec2.scale(force, 1 / body.mass)
    );
  }

  update(deltaTime: number): void {
    for (const body of this.bodies) {
      if (body.isStatic) continue;

      // Apply gravity
      body.acceleration = Vec2.add(body.acceleration, this.gravity);

      // Integrate velocity
      body.velocity = Vec2.add(body.velocity, Vec2.scale(body.acceleration, deltaTime));

      // Apply damping
      body.velocity = Vec2.scale(body.velocity, 0.99);

      // Integrate position
      body.position = Vec2.add(body.position, Vec2.scale(body.velocity, deltaTime));

      // Reset acceleration
      body.acceleration = { x: 0, y: 0 };
    }
  }
}
```

### Friction & Drag

```typescript
// Ground friction
function applyFriction(body: PhysicsBody, groundNormal: Vector2, frictionCoeff: number): void {
  // Project velocity onto ground plane
  const verticalComponent = Vec2.scale(groundNormal, Vec2.dot(body.velocity, groundNormal));
  const horizontalComponent = Vec2.sub(body.velocity, verticalComponent);

  // Apply friction to horizontal component
  const frictionForce = Vec2.scale(horizontalComponent, -frictionCoeff);
  body.velocity = Vec2.add(body.velocity, Vec2.scale(frictionForce, 1 / body.mass));
}

// Air resistance / drag
function applyDrag(body: PhysicsBody, dragCoeff: number): void {
  const speed = Vec2.length(body.velocity);
  const dragForce = Vec2.scale(
    Vec2.normalize(body.velocity),
    -dragCoeff * speed * speed
  );
  body.velocity = Vec2.add(body.velocity, Vec2.scale(dragForce, 1 / body.mass));
}
```

### Projectile Motion

```typescript
interface Projectile {
  position: Vector2;
  velocity: Vector2;
  initialPosition: Vector2;
  launchAngle: number;
  launchSpeed: number;
  gravity: number;
}

function calculateProjectile(position: Vector2, angle: number, speed: number): Projectile {
  return {
    position: { ...position },
    velocity: {
      x: Math.cos(angle) * speed,
      y: Math.sin(angle) * speed,
    },
    initialPosition: { ...position },
    launchAngle: angle,
    launchSpeed: speed,
    gravity: 9.81,
  };
}

function updateProjectile(projectile: Projectile, dt: number): void {
  projectile.velocity.y += projectile.gravity * dt;
  projectile.position = Vec2.add(
    projectile.position,
    Vec2.scale(projectile.velocity, dt)
  );
}

function getProjectileRange(speed: number, angle: number, gravity: number): number {
  return (speed * speed * Math.sin(2 * angle)) / gravity;
}

function getTimeOfFlight(speed: number, angle: number, gravity: number): number {
  return (2 * speed * Math.sin(angle)) / gravity;
}

function getMaxHeight(speed: number, angle: number, gravity: number): number {
  return (speed * speed * Math.sin(angle) * Math.sin(angle)) / (2 * gravity);
}
```

### Springs & Joints

```typescript
interface Spring {
  anchorA: Vector2;
  anchorB: Vector2;
  restLength: number;
  stiffness: number;  // k value
  damping: number;
}

function applySpringForce(bodyA: PhysicsBody, bodyB: PhysicsBody, spring: Spring): void {
  const delta = Vec2.sub(bodyB.position, bodyA.position);
  const distance = Vec2.length(delta);
  const direction = Vec2.normalize(delta);

  // Hooke's law with damping
  const displacement = distance - spring.restLength;
  const springForce = spring.stiffness * displacement;

  // Relative velocity for damping
  const relativeVelocity = Vec2.sub(bodyB.velocity, bodyA.velocity);
  const dampingForce = Vec2.dot(relativeVelocity, direction) * spring.damping;

  const totalForce = springForce + dampingForce;
  const forceVector = Vec2.scale(direction, totalForce);

  if (!bodyA.isStatic) {
    bodyA.velocity = Vec2.add(bodyA.velocity, Vec2.scale(forceVector, 1 / bodyA.mass));
  }
  if (!bodyB.isStatic) {
    bodyB.velocity = Vec2.sub(bodyB.velocity, Vec2.scale(forceVector, 1 / bodyB.mass));
  }
}

// Bungee / elastic string
class BungeeCord {
  constructor(
    private readonly stiffness: number = 50,
    private readonly damping: number = 5
  ) {}

  apply(body: PhysicsBody, anchor: Vector2): void {
    const delta = Vec2.sub(anchor, body.position);
    const distance = Vec2.length(delta);

    if (distance > this.stiffness) {
      const direction = Vec2.normalize(delta);
      const displacement = distance - this.stiffness;
      const force = displacement * this.stiffness;

      body.velocity = Vec2.add(
        body.velocity,
        Vec2.scale(direction, force / body.mass)
      );
    }
  }
}
```

### Collision Response

```typescript
interface Collision {
  bodyA: PhysicsBody;
  bodyB: PhysicsBody;
  normal: Vector2;
  penetration: number;
  point: Vector2;
}

function resolveCollision(collision: Collision): void {
  const { bodyA, bodyB, normal, penetration } = collision;

  // Separate bodies
  const totalMass = (bodyA.isStatic ? 0 : 1/bodyA.mass) + (bodyB.isStatic ? 0 : 1/bodyB.mass);
  const separation = penetration / totalMass;

  if (!bodyA.isStatic) {
    bodyA.position = Vec2.sub(bodyA.position, Vec2.scale(normal, separation * (1/bodyA.mass)));
  }
  if (!bodyB.isStatic) {
    bodyB.position = Vec2.add(bodyB.position, Vec2.scale(normal, separation * (1/bodyB.mass)));
  }

  // Relative velocity
  const relativeVelocity = Vec2.sub(bodyB.velocity, bodyA.velocity);
  const velocityAlongNormal = Vec2.dot(relativeVelocity, normal);

  // Don't resolve if separating
  if (velocityAlongNormal > 0) return;

  // Restitution
  const e = Math.min(bodyA.restitution, bodyB.restitution);
  const j = -(1 + e) * velocityAlongNormal / totalMass;

  // Apply impulse
  const impulse = Vec2.scale(normal, j);
  if (!bodyA.isStatic) {
    bodyA.velocity = Vec2.sub(bodyA.velocity, Vec2.scale(impulse, 1/bodyA.mass));
  }
  if (!bodyB.isStatic) {
    bodyB.velocity = Vec2.add(bodyB.velocity, Vec2.scale(impulse, 1/bodyB.mass));
  }
}
```

### Platformer Physics

```typescript
interface PlayerPhysics {
  position: Vector2;
  velocity: Vector2;
  isGrounded: boolean;
  coyoteTime: number;    // Grace period after leaving platform
  jumpBuffer: number;    // Jump input buffer
}

function updatePlayerPhysics(player: PlayerPhysics, input: Input, dt: number): void {
  const GRAVITY = 25;
  const MOVE_SPEED = 8;
  const JUMP_FORCE = 12;
  const COYOTE_TIME = 0.1;
  const JUMP_BUFFER_TIME = 0.1;

  // Horizontal movement
  if (input.left) player.velocity.x = -MOVE_SPEED;
  else if (input.right) player.velocity.x = MOVE_SPEED;
  else player.velocity.x = 0;

  // Coyote time
  if (player.isGrounded) {
    player.coyoteTime = COYOTE_TIME;
  } else {
    player.coyoteTime -= dt;
  }

  // Jump buffer
  if (input.jumpPressed) {
    player.jumpBuffer = JUMP_BUFFER_TIME;
  } else {
    player.jumpBuffer -= dt;
  }

  // Jump
  if (player.jumpBuffer > 0 && player.coyoteTime > 0) {
    player.velocity.y = -JUMP_FORCE;
    player.isGrounded = false;
    player.jumpBuffer = 0;
    player.coyoteTime = 0;
  }

  // Variable jump height
  if (!input.jumpHeld && player.velocity.y < 0) {
    player.velocity.y *= 0.5;
  }

  // Gravity
  player.velocity.y += GRAVITY * dt;

  // Apply velocity
  player.position = Vec2.add(player.position, Vec2.scale(player.velocity, dt));
}
```

### Best Practices

1. **Fixed timestep** - Use constant dt for physics (e.g., 1/60)
2. **Sleep optimization** - Disable static bodies
3. **Broad phase** - Spatial partitioning for collision
4. **Continuous collision** - Prevent tunneling at speed
5. **Resting contact** - Prevent jittering on ground
6. **Float precision** - Use epsilon for comparisons
7. **Tune values** - Friction/restitution are game feel
