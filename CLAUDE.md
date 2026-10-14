# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Clon del clásico arcade **Asteroids** implementado en canvas HTML5 puro, sin dependencias ni bundler. El juego consiste en una nave espacial que se mueve en un campo de asteroides con envolvimiento toroidal de bordes (el espacio es un toro).

## Architecture

### File Structure
- `index.html` — Entrada del proyecto, crea elemento `<canvas>` y carga `game.js`
- `game.js` — Todo el código del juego en un solo archivo (~420 líneas)
- `favicon.svg` — Icono de pestaña del navegador

### High-Level Architecture

El juego sigue un patrón de **game loop** tradicional con separación entre actualización (`update`) y renderizado (`draw`):

```
┌─────────────────────────────────────┐
│              Main Loop              │
│            requestAnimationFrame    │
├─────────────────────────────────────┤
│  update(dt)                         │
│  ├─ Process input                   │
│  ├─ Update entities                 │
│  │   ├─ Ship (nave)                │
│  │   ├─ Bullets (balas)            │
│  │   ├─ Asteroids                  │
│  │   └─ Particles (explosiones)    │
│  └─ Collision detection             │
├─────────────────────────────────────┤
│       draw()                        │
│  └─ Render all entities + HUD       │
└─────────────────────────────────────┘
```

### Core Classes (en `game.js`)

| Clase | Líneas | Descripción |
|-------|--------|-------------|
| `Bullet` | 32-58 | Proyectiles disparados por la nave |
| `Asteroid` | 65-119 | Asteroides de diferentes tamaños (grande/mediano/pequeño) |
| `Ship` | 122-204 | La nave espacial del jugador |
| `Particle` | 207-236 | Efectos de explosión |

### Game State Machine

El juego tiene tres estados principales (líneas 241, 294-308):

```javascript
state: 'playing' | 'dead' | 'gameover'

// playing: Nave activa, procesar input y colisiones
if (state === 'playing') {
  // Update entidades, colisiones bala-asteroide, nave-asteroide
}

// dead: Nave destruida, espera de recuperación
if (state === 'dead') {
  deadTimer -= dt;
  if (deadTimer <= 0) { state = 'playing'; ship.reset(); }
}

// gameover: Fin del juego, reiniciar con Espacio
if (state === 'gameover') {
  if (pressed('Space')) initGame();
}
```

### Key Constants

| Constante | Valor(s) | Descripción |
|-----------|----------|-------------|
| `W`, `H` | 800, 600 | Dimensiones del canvas |
| `RADII` | [0, 16, 30, 50] | Radio por tamaño de asteroide |
| `SPEEDS` | [0, 85, 55, 32] | Velocidad base por tamaño |
| `POINTS` | [0, 100, 50, 20] | Puntos por destruir |

## Common Development Tasks

### Run the Game

Abrir directamente en navegador:
```bash
open index.html
```

O usar un servidor local:
```bash
npx serve .
```

Luego visitar `http://localhost:3000`.

### Code Structure Notes

- El código es **ES6+** con módulos ES (`'use strict'`)
- Todo el juego está en un solo archivo JavaScript cargado directamente en HTML
- No hay dependencias externas, frameworks o bundlers
- Uso de clases ES6 para entidades del juego
- Sistema de coordenadas: mundo 2D con envolvimiento toroidal

### Utils Helper Functions

Las funciones utilitarias en `game.js` (líneas 27-30):

```javascript
const wrap    = (v, max) => ((v % max) + max) % max;  // Envolvimiento toroidal
const dist    = (a, b)   => Math.hypot(a.x - b.x, a.y - b.y);  // Distancia
const rand    = (min, max) => min + Math.random() * (max - min);  // Float aleatorio
const randInt = (min, max) => Math.floor(rand(min, max + 1));     // Int aleatorio
```

### Input Handling

El sistema de input usa un estado actual y estados recién presionados:

```javascript
const keys = {};      // Estado actual
const justPressed = {};  // Estados recién presionados (single-frame true)

function pressed(code) {
  const val = justPressed[code];
  justPressed[code] = false;
  return val;
}
```

### Collision Detection

Colisión simple basada en distancia entre centro de balas/asteroides y su radio:

```javascript
// Bala vs asteroide (línea 327)
if (dist(b, a) < a.radius) { /* hit */ }

// Nave vs asteroide (línea 342)  
if (dist(ship, a) < ship.radius + a.radius * 0.82) { /* hit */ }
```