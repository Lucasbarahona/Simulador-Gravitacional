# 🌌 Simulador Gravitacional N-Cuerpos

> Simulador gravitacional interactivo construido con JavaScript puro y HTML5 Canvas, implementando **integración numérica Velocity Verlet** y la **Ley de Gravitación Universal de Newton** para dinámica orbital físicamente precisa.

![Demo](https://img.shields.io/badge/demo-en_vivo-brightgreen) ![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-yellow) ![HTML5](https://img.shields.io/badge/HTML5-Canvas-orange) ![Física](https://img.shields.io/badge/Física-Newtoniana-blue)

---

## 🔭 Demo en Vivo

> Abre `gravity-simulator.html` directamente en cualquier navegador moderno — sin instalación, sin dependencias.

**Pruébalo:** Clona el repositorio → abre el archivo → haz click para añadir cuerpos y arrastra para establecer su velocidad inicial.

---

## ✨ Características

- **Simulación N-cuerpos en tiempo real** — cada cuerpo atrae a todos los demás simultáneamente
- **Integración Velocity Verlet** — integrador simpéctico que conserva la energía mucho mejor que el método de Euler simple
- **Suavizado gravitacional (softening)** — técnica estándar en astrofísica computacional para evitar singularidades numéricas cuando los cuerpos se acercan demasiado
- **Monitor de energía en vivo** — gráfico de energía cinética que valida la precisión del integrador en tiempo real
- **Controles interactivos** — añade cuerpos con click+arrastre para establecer velocidad inicial; ajusta G, timestep (dt), suavizado (ε) y masa al vuelo
- **Tres presets** — Sistema Binario, Sistema Solar y modo Caos
- **Renderizado de trayectorias orbitales** — efecto de desvanecimiento usando repintado semitransparente del canvas

---

## 🧮 La Física

### Ley de Gravitación Universal de Newton

La fuerza entre dos cuerpos cualesquiera $i$ y $j$ es:

$$\vec{F}_{ij} = G \frac{m_i m_j}{r^2} \hat{r}_{ij}$$

donde $G$ es la constante gravitacional, $m_i, m_j$ son las masas y $r$ es la distancia entre ellos.

Para obtener la fuerza como **vector**, la descomponemos a lo largo del vector desplazamiento $\vec{d} = \vec{r}_j - \vec{r}_i$:

$$F_x = G \cdot \frac{m_i m_j}{r^3} \cdot dx \qquad F_y = G \cdot \frac{m_i m_j}{r^3} \cdot dy$$

Por la Tercera Ley de Newton, la fuerza sobre $j$ es igual y opuesta — lo que nos permite calcular ambas en un solo paso del bucle, obteniendo un algoritmo O(n²/2).

### Suavizado Gravitacional

Para evitar la división por cero (y las fuerzas infinitas resultantes) cuando $r \to 0$, añadimos el parámetro de suavizado $\varepsilon$:

$$r_{\text{suav}}^2 = r^2 + \varepsilon^2$$

Esta es una técnica estándar en astrofísica N-cuerpos, usada en códigos como GADGET y AREPO. Físicamente equivale a tratar los cuerpos como distribuciones de masa extendidas en lugar de masas puntuales.

### Integración Velocity Verlet

El integrador de Euler simple tiene un error acumulado O(dt) — la energía se desvía sin límite. El método **Velocity Verlet** es un integrador simpéctico de segundo orden con error O(dt²):

$$\vec{v}\left(t + \tfrac{dt}{2}\right) = \vec{v}(t) + \vec{a}(t)\cdot\tfrac{dt}{2}$$

$$\vec{x}(t + dt) = \vec{x}(t) + \vec{v}\!\left(t+\tfrac{dt}{2}\right)\cdot dt$$

$$\text{[recalcular } \vec{a}(t+dt)\text{]}$$

$$\vec{v}(t + dt) = \vec{v}\!\left(t+\tfrac{dt}{2}\right) + \vec{a}(t+dt)\cdot\tfrac{dt}{2}$$

Este enfoque conserva la **estructura simpéctica** de la mecánica hamiltoniana, lo que significa que la energía total ($EC + EP$) permanece acotada en integraciones largas — visible en el gráfico de energía en tiempo real.

### Conservación de Energía (Validación)

La energía mecánica total debe permanecer aproximadamente constante:

$$E_{\text{total}} = \underbrace{\sum_i \frac{1}{2} m_i v_i^2}_{EC} - \underbrace{\sum_{i<j} \frac{G m_i m_j}{r_{ij}}}_{|EP|} = \text{const}$$

El gráfico de EC en vivo permite observar cómo cambiar `dt` o `softening` afecta la estabilidad numérica.

---

## 🏗️ Arquitectura

```
gravity-simulator.html
│
├── Variables CSS & Layout       → sistema de diseño, grilla responsiva
│
├── Sección 1 — Setup            → inicialización del canvas, estado de cámara
│
├── Sección 2 — clase Body       → posición, velocidad, aceleración, trayectoria
│
├── Sección 3 — Motor de Física
│   ├── computeAccelerations()   → cálculo de fuerzas por pares O(n²)
│   ├── integrateVerlet()        → integrador Velocity Verlet de dos pasos
│   └── computeEnergy()          → EC + EP para validación
│
├── Sección 4 — Renderizador
│   ├── render()                 → bucle de dibujo con desvanecimiento de trayectorias
│   └── renderEnergyGraph()      → serie temporal de EC en vivo
│
└── Sección 5 — Loop Principal & UI
    ├── loop()                   → requestAnimationFrame a ~60fps
    ├── Presets                  → Estrella Binaria, Sistema Solar, Caos
    └── Event listeners          → mouse, scroll, sliders, botones
```

**Sin dependencias.** Sin librerías, sin compilación, sin framework. Un solo archivo.

---

## 🎮 Cómo Usarlo

| Acción | Efecto |
|---|---|
| **Click** en el canvas | Añade un cuerpo con velocidad cero |
| **Click + Arrastrar** | Añade un cuerpo; dirección y longitud del arrastre definen la velocidad inicial |
| **Scroll** | Zoom in / out |
| **Slider G** | Aumenta la constante gravitacional (atracción más fuerte) |
| **Slider dt** | Cambia el paso de tiempo (menor = más preciso, más lento) |
| **Slider ε (softening)** | Aumentar para estabilizar encuentros cercanos |
| **Slider Masa** | Define la masa del próximo cuerpo a añadir |

---

## 🔬 Aspectos Técnicos Destacados

Este proyecto demuestra:

- **Métodos numéricos** — Velocity Verlet, integrador simpéctico usado en dinámica molecular y astrofísica
- **Conciencia de complejidad computacional** — cálculo de fuerzas O(n²) con optimización por Tercera Ley de Newton (n²/2 evaluaciones)
- **Álgebra vectorial** — descomposición de la fuerza gravitacional en componentes cartesianas
- **Conservación de energía como métrica de corrección** — usando invariantes físicos para validar el código
- **Pipeline de renderizado en Canvas** — equivalente a doble búfer mediante fill semitransparente, gradientes radiales y `requestAnimationFrame`
- **Diseño orientado a objetos** — clase `Body` encapsulando estado y comportamiento
- **Ajuste de parámetros en tiempo real** — los sliders modifican el estado de la simulación en vivo sin reinicio

---

## 🚀 Posibles Extensiones

- [ ] **Algoritmo Barnes-Hut** — cálculo de fuerzas O(n log n) usando partición espacial con quadtree
- [ ] **Colisiones elásticas / inelásticas** — fusionar cuerpos conservando momento lineal $\vec{p} = m\vec{v}$
- [ ] **Integrador Runge-Kutta 4** — comparar precisión vs. Verlet con igual coste computacional
- [ ] **Bloqueo al centro de masa** — restar la deriva para mantener el sistema centrado
- [ ] **Exportar datos orbitales como CSV** — registrar posiciones a lo largo del tiempo para análisis externo

---

## 📚 Referencias

- Verlet, L. (1967). *Computer Experiments on Classical Fluids*. Physical Review, 159(1).
- Springel, V. (2005). *The cosmological simulation code GADGET-2*. MNRAS, 364(4).
- Newton, I. (1687). *Philosophiæ Naturalis Principia Mathematica*.

---

## 👤 Autor

Desarrollado como proyecto de portafolio que demuestra simulación física aplicada y computación numérica en el navegador.

---

*Implementación en un solo archivo · Sin dependencias · HTML5 + CSS3 + ES6+ puro*

---

## 🧠 Lo que Aprendí Construyendo Este Proyecto

Tengo certificación en fundamentos de HTML, CSS y JavaScript, y experiencia previa con C++, Arduino y Docker. Sin embargo, este proyecto me llevó a territorios que ningún curso introductorio cubre.

### De saber programar a entender física computacional

El mayor salto fue darme cuenta de que **programar una simulación física no es lo mismo que programar una aplicación**. En una app web normal, si algo falla, lo ves en pantalla. En una simulación, el código puede ejecutarse perfectamente y aun así estar mal — porque la física se rompe de formas silenciosas.

**El problema concreto:** Mi primera versión usaba el integrador de Euler simple:
```javascript
// ❌ Lo que hice primero — parece correcto, pero no lo es
body.vx += body.ax * dt;
body.vy += body.ay * dt;
body.x  += body.vx * dt;
body.y  += body.vy * dt;
```
Las órbitas se espiralizaban hacia afuera lentamente. Los planetas ganaban energía de la nada y salían disparados. Tardé un tiempo en entender que el problema no era un bug de código — era un **error matemático de primer orden**. Euler acumula error en cada frame y la energía del sistema "se escapa".

La solución fue aprender el **integrador Velocity Verlet**, que divide cada paso en dos mitades y recalcula la aceleración en el medio. El resultado: la energía total oscila pero no diverge. Eso fue el momento en que entendí que los métodos numéricos no son un detalle académico — son la diferencia entre una simulación que funciona y una que miente.

### Álgebra vectorial aplicada, no abstracta

En clase uno aprende que una fuerza tiene módulo y dirección. Aquí tuve que implementarlo: dado que la fuerza gravitacional apunta *de un cuerpo hacia otro*, necesité descomponerla en sus componentes $F_x$ y $F_y$ usando el vector desplazamiento normalizado. El concepto de **dividir por $r$ para normalizar** pasó de ser una fórmula a ser algo que entiendo geométricamente.

### Complejidad computacional deja de ser teórica

Sabía que O(n²) era "malo". Aquí lo *sentí*: con 5 cuerpos el simulador vuela a 60fps; con 30 empieza a trabajar; con 100 se arrastra. Cada cuerpo nuevo no suma trabajo — lo multiplica. Eso me llevó a investigar el **algoritmo Barnes-Hut** (O(n log n)), que está en mi lista de extensiones futuras. Nunca había tenido una razón tan concreta para querer optimizar algo.

### El canvas como sistema de coordenadas independiente

Aprender a separar el **espacio de simulación** (donde viven los planetas) del **espacio de pantalla** (píxeles del canvas) fue un cambio de mentalidad importante. La función `simToScreen()` que implementé es una transformación afín básica — el mismo concepto que usan los motores de videojuegos y las aplicaciones de CAD.

### Lo que este proyecto me confirmó

Que quiero estudiar ingeniería no solo para escribir software, sino para entender los sistemas que el software modela. La frontera entre matemática, física y código es exactamente donde quiero trabajar.

