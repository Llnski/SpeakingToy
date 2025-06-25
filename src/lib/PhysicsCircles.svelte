<script>
import { onMount, onDestroy } from 'svelte';

let container;
let app;
let world;
const bodies = [];
const sprites = [];
let dragging = null;

const width = 600;
const height = 400;
const radius = 30;
const circleCount = 5;

onMount(async () => {
  const PIXI = await import('https://cdn.skypack.dev/pixi.js');
  const planck = await import('https://cdn.skypack.dev/planck-js');

  app = new PIXI.Application({ width, height, backgroundColor: 0xffffff });
  container.appendChild(app.view);

  world = new planck.World(planck.Vec2(0, 0));

  for (let i = 0; i < circleCount; i++) {
    const body = world.createBody({
      type: 'dynamic',
      position: planck.Vec2(Math.random() * width, Math.random() * height)
    });
    body.createFixture(planck.Circle(radius), {
      density: 1,
      friction: 0.5,
      restitution: 0.8
    });
    bodies.push(body);

    const g = new PIXI.Graphics();
    g.beginFill(0x3399ff);
    g.drawCircle(0, 0, radius);
    g.endFill();
    g.x = body.getPosition().x;
    g.y = body.getPosition().y;
    g.interactive = true;
    g.cursor = 'pointer';
    g.on('pointerdown', (e) => {
      const pos = e.data.global;
      const bodyPos = body.getPosition();
      dragging = {
        body,
        offset: planck.Vec2(pos.x - bodyPos.x, pos.y - bodyPos.y)
      };
    });
    app.stage.addChild(g);
    sprites.push(g);
  }

  const onMove = () => {
    if (dragging) {
      const pos = app.renderer.plugins.interaction.mouse.global;
      dragging.body.setTransform(
        planck.Vec2(pos.x - dragging.offset.x, pos.y - dragging.offset.y),
        0
      );
      dragging.body.setLinearVelocity(planck.Vec2());
    }
  };

  app.view.addEventListener('pointermove', onMove);
  window.addEventListener('pointerup', () => (dragging = null));

  app.ticker.add(() => {
    world.step(1 / 60);
    bodies.forEach((b, i) => {
      const p = b.getPosition();
      sprites[i].x = p.x;
      sprites[i].y = p.y;
    });
  });

  onDestroy(() => {
    app.view.removeEventListener('pointermove', onMove);
    app.destroy(true);
  });
});
</script>

<div bind:this={container} style="width:{width}px; height:{height}px;"></div>

