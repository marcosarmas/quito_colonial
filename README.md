# Leyendas de Quito — Una noche en la Real Audiencia

Un juego en primera persona en pixel art nativo, ambientado en el Centro Histórico de Quito en una época colonial legendaria (siglos XVI–XVIII). Es de noche, hay neblina, se prenden los faroles y el Panecillo se ve al fondo.

Todo está en **un solo archivo**, `index.html`: JavaScript puro con Canvas 2D y Web Audio. No carga nada de afuera: las texturas, los sprites, la fuente, los efectos de sonido y la música se generan por código al arrancar.

## Cómo jugar

Abre `index.html` en un navegador moderno (Chrome, Firefox, Safari o Edge). No hace falta servidor.

| Acción | Teclado y ratón | Móvil |
|---|---|---|
| Caminar | W A S D o flechas (Shift para apurarse) | Joystick izquierdo |
| Mirar | Ratón (haz clic en el juego para capturarlo) o ← → | Arrastra en la mitad derecha |
| Hablar / usar | E o Espacio | Botón **E** |
| Elegir respuesta | 1–3, o flechas y Enter | Toca la opción |
| Plano de pergamino | M o Tab | Botón **M** |
| Diario de misiones | J | Botón **J** |
| Pausa, volumen y guardar | Esc o P | Botón **II** |

La partida se guarda en `localStorage` al cumplir cada leyenda o desde el menú de pausa. Luego se retoma con **Continuar** en la pantalla de título.

## Las cinco leyendas (se hacen en cualquier orden)

1. **Cantuña y el atrio** (atrio de San Francisco). Hay que traer la última piedra desde el Callejón del Cantero y esconderla bajo las gradas antes de la campanada doce, sin que te vean los diablillos que rondan la plaza.
2. **El Chulla quiteño** (La Ronda). Consigue sombrero, leva y bastón con una cadena de trueques entre Doña Rosario, Don Lucho el sastre y Don Efraín el sombrerero. Después escoge los piropos y lleva el compás de la serenata bajo el balcón azul.
3. **El Padre Almeida** (convento de San Diego). Misión de sigilo: llévalo hasta el Cristo sin que lo vean los frailes, acompáñalo a la chichería y regresa antes del amanecer. Termina con «¿Hasta cuándo, padre Almeida?» y «Hasta la vuelta, Señor».
4. **Don Ramón Ayala y el Gallo de la Catedral** (calle Chile y Plaza Grande). Guía al borrachito escondiéndote de los alguaciles en los zaguanes, hasta que el gallo de la torre le saque la promesa.
5. **La Dama Tapada** (calle de las Siete Cruces). Síguela calle abajo, junta las señales (el pañuelo, el olor a nardos, las huellas y lo que cuenta la beata) y no caigas en su trampa.

Cuando cumples las cinco, amanece sobre Quito: se ve una panorámica desde lo alto, suenan las campanas y pasan los créditos.

## El mapa

Es una cuadrícula de 84×62 celdas (1 celda ≈ 3,5 m), con el norte arriba. El damero se reconstruyó a mano porque la API de OpenStreetMap no estaba accesible, así que es una versión simplificada que respeta las posiciones relativas:

```
          Imbabura  Cuenca  Benalcázar  G.Moreno  Venezuela  Guayaquil  Flores
 Chile    ──┼────────┼─────────┼──[Arzobispal]─────┼─────────┼──[mistelas]
            │        │  [Audiencia] PLAZA GRANDE [Cabildo]   │
 Espejo   ──┼────────┼─────────┼──[CATEDRAL + gallo]────────┼─────────┼──
            │ [S.FCO │[LA COMPAÑÍA] ↓ Siete Cruces           │
 Sucre    ──┼─convento]──ATRIO─┼──────────────┼──────────────┼─────────┼──
            │ [iglesia] PLAZA  │   [Sagrario]               │  PLAZA
 Bolívar  ──┼── S.FCO ─┼───────┼──────────────┼──────────────┼─STO.DOMINGO
  SAN DIEGO │     Callejón del │ [Carmen Alto]              │ [iglesia]
 Rocafuerte (subida)──Cantero──┼──────────────┼──────────────┼─────────┼──
                               ╲____ LA RONDA (curva, hondonada) ____╱
                                 Mirador de la quebrada → El Panecillo
```

El terreno sube hacia el oeste (la loma de San Diego, las faldas del Pichincha) y baja hacia la quebrada del sur. Por eso hay pendientes y gradas en García Moreno, La Ronda y la subida a San Diego.

## Arquitectura del código

Todo va en un solo `<script>`, dividido en secciones comentadas:

- **CORE / PALETA / LUT**: paleta fija de 47 colores. Una tabla de búsqueda `LUT[niebla][luz][color]` convierte cada píxel a un color de la paleta, con tinte de luna o de farol y niebla por distancia, y el dithering Bayer 4×4 suaviza los degradados.
- **FUENTE**: fuente bitmap 5×7 proporcional, hecha a mano, con tildes, ñ, ¿ y ¡.
- **TEXTURAS / SPRITES**: generadores procedurales de texturas de 32×32 (cal con zócalo, balcones azules y verdes, portones, rejas, sillería, la portada dorada de La Compañía, campanarios, cúpulas, adoquín voronoi, lajas, tejas…). Un generador paramétrico de personajes arma todos los sprites billboard.
- **MUNDO**: la cuadrícula de alturas, con el tipo de cada celda, la altura del piso, la altura de las paredes y las texturas por cara y nivel. También coloca los faroles y precalcula un mapa de luz con línea de visión.
- **RENDER**: un raycaster de campo de alturas. Cada columna recorre la cuadrícula con DDA de adelante hacia atrás y dibuja pisos, contrahuellas y fachadas con recorte vertical. Así se ven pendientes y gradas, y las torres asoman por encima de los tejados. Encima van el cielo panorámico procedural (estrellas, luna, Pichincha, Panecillo), los sprites con prueba de profundidad, los halos de farol con dithering y la neblina que se arrastra.
- **AUDIO / MÚSICA**: el sonido se sintetiza con Web Audio (pasos sobre adoquín, viento con ráfagas, campanas con parciales inarmónicos, perros, búho, grillos y reverb por convolución). La música es generativa y original: cadenas de Markov sobre la pentatónica andina, en yaraví y sanjuanito, con quena, rondador, charango (Karplus-Strong) y bombo.
- **UI / DIÁLOGOS**: cajas retro con retrato y efecto de máquina de escribir, respuestas que ramifican cada misión, plano de pergamino, diario, pausa y título.
- **MISIONES**: una máquina de estados por leyenda. Las escenas se escriben como funciones `async`, con `await talk()`, `wait()` y `fadeTo()`. Hay guardias con cono de visión y seguidores que caminan sobre tu rastro.

`window.__debug` expone ganchos para las pruebas automatizadas (teletransporte, estado, etc.).
