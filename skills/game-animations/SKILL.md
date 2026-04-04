---
name: game-animations
description: Game animations - sprite animation, character animations, tweening, particle effects, and screen transitions.
metadata:
  priority: 7
  docs:
    - "https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame"
  pathPatterns:
    - "**/animation/**"
    - "**/sprites/**"
  bashPatterns:
    - '\banimation\b'
    - '\bsprite\b'
  promptSignals:
    phrases:
      - "game animation"
      - "sprite animation"
      - "tweening"
    anyOf:
      - "animation"
      - "sprite"
      - "tween"
---

## Game Animations

### Sprite Animation

```typescript
// Sprite sheet animation
interface SpriteAnimation {
  name: string;
  frames: number[];
  frameDuration: number; // ms per frame
  loop: boolean;
}

class SpriteAnimator {
  private currentAnimation: SpriteAnimation | null = null;
  private currentFrame = 0;
  private elapsed = 0;
  private playing = true;

  play(animation: SpriteAnimation, forceRestart = false) {
    if (this.currentAnimation?.name === animation.name && !forceRestart) return;

    this.currentAnimation = animation;
    this.currentFrame = 0;
    this.elapsed = 0;
    this.playing = true;
  }

  pause() { this.playing = false; }
  resume() { this.playing = true; }
  stop() { this.playing = false; this.currentFrame = 0; }

  update(deltaTime: number) {
    if (!this.playing || !this.currentAnimation) return;

    this.elapsed += deltaTime;
    if (this.elapsed >= this.currentAnimation.frameDuration) {
      this.elapsed = 0;
      this.currentFrame++;

      if (this.currentFrame >= this.currentAnimation.frames.length) {
        if (this.currentAnimation.loop) {
          this.currentFrame = 0;
        } else {
          this.currentFrame = this.currentAnimation.frames.length - 1;
          this.playing = false;
        }
      }
    }
  }

  getCurrentFrame() {
    return this.currentAnimation?.frames[this.currentFrame] ?? 0;
  }
}
```

### Tweening

```typescript
// Easing functions
const Easing = {
  linear: (t: number) => t,

  // Quadratic
  easeInQuad: (t: number) => t * t,
  easeOutQuad: (t: number) => t * (2 - t),
  easeInOutQuad: (t: number) => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,

  // Cubic
  easeInCubic: (t: number) => t * t * t,
  easeOutCubic: (t: number) => (--t) * t * t + 1,
  easeInOutCubic: (t: number) =>
    t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1,

  // Elastic
  easeInElastic: (t: number) => {
    const c4 = (2 * Math.PI) / 3;
    return t === 0 ? 0 : t === 1 ? 1 :
      -Math.pow(2, 10 * t - 10) * Math.sin((t * 10 - 10.75) * c4);
  },
  easeOutElastic: (t: number) => {
    const c4 = (2 * Math.PI) / 3;
    return t === 0 ? 0 : t === 1 ? 1 :
      Math.pow(2, -10 * t) * Math.sin((t * 10 - 0.75) * c4) + 1;
  },

  // Bounce
  easeOutBounce: (t: number) => {
    const n1 = 7.5625, d1 = 2.75;
    if (t < 1 / d1) return n1 * t * t;
    if (t < 2 / d1) return n1 * (t -= 1.5 / d1) * t + 0.75;
    if (t < 2.5 / d1) return n1 * (t -= 2.25 / d1) * t + 0.9375;
    return n1 * (t -= 2.625 / d1) * t + 0.984375;
  },
};

// Tween system
interface Tween<T> {
  target: T;
  property: keyof T;
  from: number;
  to: number;
  duration: number;
  easing: (t: number) => number;
  onUpdate: (value: number) => void;
  onComplete?: () => void;
}

class TweenManager {
  private tweens: Map<string, Tween<any>> = new Map();

  create<T>(
    target: T,
    property: keyof T,
    to: number,
    duration: number,
    options: Partial<Tween<T>> = {}
  ): string {
    const id = crypto.randomUUID();
    const from = target[property] as number;

    this.tweens.set(id, {
      target,
      property,
      from,
      to,
      duration,
      easing: Easing.easeOutQuad,
      onUpdate: () => {},
      ...options,
    });

    return id;
  }

  update(deltaTime: number) {
    this.tweens.forEach((tween, id) => {
      const elapsed = (tween as any)._elapsed || 0;
      (tween as any)._elapsed = elapsed + deltaTime;

      const progress = Math.min((tween as any)._elapsed / tween.duration, 1);
      const easedProgress = tween.easing(progress);
      const value = tween.from + (tween.to - tween.from) * easedProgress;

      tween.target[tween.property] = value;
      tween.onUpdate(value);

      if (progress >= 1) {
        tween.onComplete?.();
        this.tweens.delete(id);
      }
    });
  }

  remove(id: string) {
    this.tweens.delete(id);
  }
}
```

### Particle System

```typescript
// Game particles
interface Particle {
  x: number;
  y: number;
  vx: number;
  vy: number;
  life: number;
  maxLife: number;
  size: number;
  color: string;
  alpha: number;
  gravity?: number;
  friction?: number;
  rotation?: number;
  rotationSpeed?: number;
}

class ParticleSystem {
  private particles: Particle[] = [];
  private emiters: Emiter[] = [];

  emit(config: EmitConfig) {
    for (let i = 0; i < config.count; i++) {
      const angle = config.angle + (Math.random() - 0.5) * config.spread;
      const speed = config.speed * (0.5 + Math.random() * 0.5);

      this.particles.push({
        x: config.x + (Math.random() - 0.5) * config.positionSpread,
        y: config.y + (Math.random() - 0.5) * config.positionSpread,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        life: config.life * (0.5 + Math.random() * 0.5),
        maxLife: config.life,
        size: config.size * (0.5 + Math.random() * 0.5),
        color: config.colors[Math.floor(Math.random() * config.colors.length)],
        alpha: 1,
        gravity: config.gravity,
        friction: config.friction,
      });
    }
  }

  update(deltaTime: number) {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      const p = this.particles[i];

      p.x += p.vx * deltaTime;
      p.y += p.vy * deltaTime;
      p.vy += (p.gravity || 0) * deltaTime;
      p.vx *= (p.friction || 1);
      p.vy *= (p.friction || 1);
      p.life -= deltaTime;
      p.alpha = Math.max(0, p.life / p.maxLife);

      if (p.rotation !== undefined) {
        p.rotation += (p.rotationSpeed || 0) * deltaTime;
      }

      if (p.life <= 0) {
        this.particles.splice(i, 1);
      }
    }
  }

  render(ctx: CanvasRenderingContext2D) {
    for (const p of this.particles) {
      ctx.save();
      ctx.globalAlpha = p.alpha;
      ctx.fillStyle = p.color;

      if (p.rotation !== undefined) {
        ctx.translate(p.x, p.y);
        ctx.rotate(p.rotation);
        ctx.fillRect(-p.size / 2, -p.size / 2, p.size, p.size);
      } else {
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size / 2, 0, Math.PI * 2);
        ctx.fill();
      }

      ctx.restore();
    }
  }
}

// Emit configurations for common effects
const EmitConfig = {
  explosion: (x: number, y: number): EmitConfig => ({
    x, y,
    count: 30,
    speed: 200,
    angle: 0,
    spread: Math.PI * 2,
    life: 0.8,
    size: 8,
    colors: ['#ff6b35', '#f7c59f', '#efefd0', '#ff6b35'],
    gravity: 300,
    friction: 0.98,
  }),
  spark: (x: number, y: number): EmitConfig => ({
    x, y,
    count: 10,
    speed: 150,
    angle: -Math.PI / 2,
    spread: Math.PI / 4,
    life: 0.4,
    size: 4,
    colors: ['#ffd700', '#ffec8b', '#fff'],
    gravity: 400,
    friction: 0.95,
  }),
};
```

### Character Animation States

```typescript
// Animation state machine for characters
type AnimState =
  | 'idle'
  | 'walk'
  | 'run'
  | 'jump'
  | 'fall'
  | 'attack'
  | 'hurt'
  | 'death';

interface AnimTransition {
  from: AnimState;
  to: AnimState;
  condition: () => boolean;
}

class CharacterAnimator {
  private state: AnimState = 'idle';
  private animations: Map<AnimState, SpriteAnimation>;
  private transitions: AnimTransition[];
  private sprite: SpriteAnimator;

  constructor(animations: Map<AnimState, SpriteAnimation>) {
    this.animations = animations;
    this.sprite = new SpriteAnimator();
    this.transitions = [];
    this.playAnimation(this.state);
  }

  addTransition(from: AnimState, to: AnimState, condition: () => boolean) {
    this.transitions.push({ from, to, condition });
  }

  private playAnimation(state: AnimState) {
    const anim = this.animations.get(state);
    if (anim) {
      this.state = state;
      this.sprite.play(anim);
    }
  }

  update(deltaTime: number, velocity: Vector2, isGrounded: boolean) {
    // Check transitions first
    for (const t of this.transitions) {
      if (t.from === this.state && t.condition()) {
        this.playAnimation(t.to);
        break;
      }
    }

    // Auto-transition based on physics
    if (this.state === 'idle' || this.state === 'walk' || this.state === 'run') {
      if (!isGrounded && velocity.y < 0) this.playAnimation('jump');
      else if (!isGrounded && velocity.y > 0) this.playAnimation('fall');
      else if (Math.abs(velocity.x) > 0.1) {
        this.playAnimation(Math.abs(velocity.x) > 8 ? 'run' : 'walk');
      } else if (this.state !== 'idle') {
        this.playAnimation('idle');
      }
    }

    this.sprite.update(deltaTime);
  }

  trigger(state: AnimState) {
    this.playAnimation(state);
  }

  getCurrentFrame() {
    return this.sprite.getCurrentFrame();
  }
}
```

### Screen Transitions

```typescript
// Screen transition effects
class ScreenTransition {
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;
  private progress = 0;
  private duration = 0.5;
  private effect: TransitionEffect;
  private callback?: () => void;

  constructor(canvas: HTMLCanvasElement) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d')!;
  }

  async start(effect: TransitionEffect, duration = 0.5): Promise<void> {
    this.effect = effect;
    this.duration = duration;
    this.progress = 0;

    return new Promise(resolve => {
      this.callback = resolve;
      this.animate();
    });
  }

  private animate = () => {
    const deltaTime = 1 / 60; // Assume 60fps
    this.progress += deltaTime / this.duration;

    if (this.progress >= 1) {
      this.progress = 1;
      this.callback?.();
      return;
    }

    this.render();
    requestAnimationFrame(this.animate);
  };

  private render() {
    const { width, height } = this.canvas;

    switch (this.effect) {
      case 'fade':
        this.ctx.fillStyle = `rgba(0, 0, 0, ${this.progress})`;
        this.ctx.fillRect(0, 0, width, height);
        break;

      case 'slide-left':
        this.ctx.save();
        this.ctx.fillStyle = '#000';
        this.ctx.fillRect(0, 0, width * this.progress, height);
        this.ctx.restore();
        break;

      case 'slide-right':
        this.ctx.save();
        this.ctx.fillStyle = '#000';
        this.ctx.fillRect(width * (1 - this.progress), 0, width, height);
        this.ctx.restore();
        break;

      case 'zoom':
        const scale = 1 + this.progress;
        this.ctx.save();
        this.ctx.translate(width / 2, height / 2);
        this.ctx.scale(scale, scale);
        this.ctx.translate(-width / 2, -height / 2);
        this.ctx.fillStyle = '#000';
        this.ctx.fillRect(0, 0, width, height);
        this.ctx.restore();
        break;

      case 'wipe':
        this.renderWipe(width, height);
        break;
    }
  }

  private renderWipe(width: number, height: number) {
    // Horizontal wipe with jagged edge
    this.ctx.fillStyle = '#000';
    this.ctx.beginPath();
    this.ctx.moveTo(0, 0);

    for (let y = 0; y <= height; y += 20) {
      const x = width * this.progress + Math.sin(y * 0.1) * 30;
      this.ctx.lineTo(x, y);
    }

    this.ctx.lineTo(0, height);
    this.ctx.closePath();
    this.ctx.fill();
  }
}
```

### Animation Hooks (React)

```typescript
// useGameAnimation hook
function useGameAnimation(
  spriteSheet: string,
  animations: Map<AnimState, SpriteAnimation>
) {
  const [currentAnimation, setCurrentAnimation] = useState<AnimState>('idle');
  const [frame, setFrame] = useState(0);
  const animatorRef = useRef(new CharacterAnimator(animations));

  useEffect(() => {
    animatorRef.current.trigger(currentAnimation);
  }, [currentAnimation]);

  useGameLoop((deltaTime) => {
    const character = animatorRef.current;
    character.update(deltaTime, { x: 0, y: 0 }, true);
    setFrame(character.getCurrentFrame());
  });

  return { currentAnimation, setCurrentAnimation, frame };
}

// Staggered list animation
function useStaggeredAnimation<T>(
  items: T[],
  config: { delay: number; duration: number; offset: number }
) {
  const [visibleItems, setVisibleItems] = useState<T[]>([]);

  useEffect(() => {
    items.forEach((item, index) => {
      setTimeout(() => {
        setVisibleItems(prev => [...prev, item]);
      }, index * config.delay);
    });
  }, [items]);

  return visibleItems;
}

// Card flip animation
function useCardFlip() {
  const [isFlipped, setIsFlipped] = useState(false);
  const [rotation, setRotation] = useState(0);

  const flip = () => {
    const target = isFlipped ? 0 : 180;
    animateValue(rotation, target, 300, (v) => setRotation(v));
    setIsFlipped(!isFlipped);
  };

  return { isFlipped, rotation, flip };
}
```

### Best Practices

1. **Frame rate independence** - Always use deltaTime
2. **Object pooling** - Reuse particles and objects
3. **Easing** - Natural-feeling motion
4. **Stagger** - Add delay between items
5. **Squash/stretch** - Juice for game feel
6. **Sound sync** - Match audio to visuals
7. **Performance** - Limit particle counts
