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
