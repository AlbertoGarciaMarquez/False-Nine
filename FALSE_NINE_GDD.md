# FALSE NINE — Game Design Document

**Versión del documento:** 0.1  
**Estado:** Documento vivo  
**Proyecto:** False Nine  
**Tipo de juego:** Web game / Football draft roguelike  
**Plataforma actual:** GitHub Pages  
**Idioma:** Español  

---

## 1. Concepto general

**False Nine** es un juego web de fútbol tipo **draft roguelike**, donde el jugador crea un club, elige una plantilla mediante draft y avanza por una temporada con partidos, eventos, fichajes, entrenamientos, sponsors y decisiones estratégicas.

El juego está pensado para partidas rápidas, con decisiones sencillas pero con suficiente profundidad para que cada temporada sea diferente.

La idea principal es que el jugador actúe como entrenador y director deportivo, tomando decisiones que afecten directamente al rendimiento del equipo.

---

## 2. Objetivo de diseño

El objetivo principal del juego es que cada decisión tenga consecuencias visibles.

El jugador debe:

- Elegir bien su plantilla inicial.
- Gestionar el cansancio de los jugadores.
- Rotar titulares y suplentes.
- Decidir el estilo de juego antes de cada partido.
- Mejorar el equipo mediante fichajes.
- Reaccionar ante eventos aleatorios.
- Mantener al equipo competitivo durante toda la temporada.
- Intentar ganar el título evitando caer en una mala racha.

---

## 3. Estado actual del juego

### Ya implementado

- Pantalla inicial con nombre de club.
- Draft inicial de jugadores.
- Plantilla con estadísticas.
- Mapa de temporada.
- Nodos de partido.
- Nodos de entrenamiento.
- Nodos de fichaje.
- Nodos de sponsor.
- Nodos de evento.
- Checkpoints.
- Simulación de partidos.
- Pantalla de resultado.
- Progreso de temporada.
- Game Over.
- Pantalla de campeón.
- Autoguardado con `localStorage`.
- Botón para continuar partida.
- Botón para borrar partida guardada.

---

## 4. Filosofía de gameplay

False Nine debe sentirse como una mezcla entre:

- Draft deportivo.
- Gestión ligera de plantilla.
- Roguelike de decisiones.
- Simulador rápido de temporada.

El jugador no debe sentirse obligado a gestionar demasiados detalles, pero sí debe notar que sus decisiones importan.

La profundidad debe venir de sistemas simples combinados entre sí:

- Calidad de los jugadores.
- Energía.
- Estilo de juego.
- Rotaciones.
- Fichajes.
- Eventos.
- Presupuesto.
- Riesgo/recompensa.

---

# 5. Mecánicas actuales

## 5.1 Draft inicial

El jugador elige una plantilla inicial mediante una serie de elecciones tipo draft.

Actualmente el draft genera varias opciones por posición y el jugador escoge una.

### Objetivo actual

Construir una primera plantilla funcional para empezar la temporada.

### Mejora prevista

Ampliar el draft inicial para que tenga sentido con el futuro sistema de cansancio y rotaciones.

---

## 5.2 Plantilla

Cada jugador tiene estadísticas principales:

- `VEL` — Velocidad.
- `PAX` — Pase.
- `DIS` — Disparo.
- `DEF` — Defensa.
- `FÍS` — Físico.

La media del jugador se calcula a partir de estas estadísticas.

### Mejora prevista

Añadir una nueva propiedad:

```js
energy: 100
```

Esta energía representará el cansancio del jugador a lo largo de la temporada.

---

## 5.3 Prepartido

Antes de cada partido, el jugador puede seleccionar formación y estilo.

### Problema actual detectado

Cuando se cambia el estilo de juego en el prepartido, se selecciona visualmente, pero el jugador no percibe claramente qué cambia.

### Mejora necesaria

El prepartido debe mostrar en directo:

- Poder del equipo.
- Poder del rival.
- Probabilidad estimada de victoria.
- Bonus del estilo seleccionado.
- Penalización o riesgo del estilo.
- Impacto previsto en el cansancio.

---

# 6. Mecánicas previstas

## 6.1 Prepartido mejorado

El prepartido debe convertirse en una pantalla de decisión real.

### Información que debería mostrar

```text
Tu equipo: 78
Rival: 82

Probabilidad estimada:
Victoria 32% · Empate 28% · Derrota 40%

Estilo seleccionado:
Presión alta
+ Ataque
+ Ocasiones
- Más cansancio
```

### Estilos previstos

#### Posesión

- Cansancio bajo.
- Más control.
- Menos riesgo.
- Menor explosividad ofensiva.

#### Presión

- Más ataque.
- Más generación de ocasiones.
- Más riesgo.
- Mucho más cansancio.

#### Contra

- Cansancio medio.
- Mejor contra rivales fuertes.
- Puede reducir el control del partido.
- Aumenta la posibilidad de sorprender.

---

## 6.2 Energía / cansancio

Cada jugador tendrá una barra de energía de 0 a 100.

```js
energy: 100
```

La energía afectará al rendimiento del jugador durante los partidos.

### Rangos de energía

```text
100-80 → rendimiento normal
79-60  → pequeña penalización
59-40  → penalización notable
39-20  → rendimiento bajo
0-19   → rendimiento muy bajo
```

### Penalización propuesta

```js
function energyModifier(player){
  if(player.energy >= 80) return 1;
  if(player.energy >= 60) return 0.96;
  if(player.energy >= 40) return 0.90;
  if(player.energy >= 20) return 0.82;
  return 0.72;
}
```

### Visualización

La energía debe mostrarse en plantilla y prepartido.

Ejemplo:

```text
Bellingham  89  Energía ███████░░░ 72%
```

---

## 6.3 Cansancio según FÍS

El cansancio dependerá del atributo `FÍS`.

Cuanto más físico tenga un jugador, menos energía perderá tras jugar un partido.

### Fórmula propuesta

```js
function fatigueLoss(player, style){
  let base = 12;

  if(style === 'presion') base = 18;
  if(style === 'posesion') base = 10;
  if(style === 'contra') base = 13;

  const fisReduction = Math.round(player.fis / 10);
  const loss = base - fisReduction;

  return Math.max(5, loss);
}
```

### Ejemplo

```text
Jugador con FÍS 90 usando presión:
18 - 9 = pierde 9 energía

Jugador con FÍS 50 usando presión:
18 - 5 = pierde 13 energía
```

### Objetivo

Hacer que `FÍS` sea una estadística importante y no solo decorativa.

---

## 6.4 Recuperación de energía

Los jugadores que no juegan deben recuperar energía.

### Propuesta

```text
Titulares:
- Pierden energía tras el partido.
- Recuperan poco si siguen acumulando minutos.

Suplentes:
- Recuperan energía.
- Recuperan algo más si tienen buen FÍS.
```

### Fórmula propuesta

```js
function energyRecovery(player, played){
  const base = played ? 4 : 12;
  const fisBonus = Math.round(player.fis / 20);

  return base + fisBonus;
}
```

---

## 6.5 Once titular editable

### Problema actual

El once titular se basa en los primeros jugadores de la plantilla/draft, por lo que el jugador no puede rotar manualmente.

### Mejora propuesta

Añadir una sección de once titular en el prepartido.

Ejemplo:

```text
Once titular

[PT] Courtois      Cambiar
[CB] Militão       Cambiar
[CB] Rüdiger       Cambiar
[LI] Mendy         Cambiar
[LD] Carvajal      Cambiar
...
```

Al pulsar **Cambiar**, se mostrarán jugadores compatibles con esa posición.

### Primera versión

No usar drag & drop inicialmente.

La primera versión debe ser simple, con botones.

Ejemplo:

```text
Cambiar portero:

- Courtois
- Lunin
```

### Objetivo

Permitir rotaciones y hacer que el cansancio tenga impacto real.

---

## 6.6 Plantilla ampliada

### Problema

Con cansancio, una plantilla de 16 jugadores se queda corta.

### Propuesta

Aumentar el draft inicial a 20 jugadores.

```text
11 titulares
9 suplentes
```

### Distribución recomendada

#### Titulares

```text
1 PT
2 CB
1 LI
1 LD
1 MCD
2 MC
1 EI
1 ED
1 DC
```

#### Suplentes

```text
1 PT suplente
2 defensas
3 centrocampistas
3 atacantes
```

### Plantilla máxima

```text
22 jugadores
```

### Motivo

20 jugadores de inicio permiten rotar sin hacer el draft demasiado largo.

El límite máximo de 22 permite que durante la temporada se puedan fichar 2 jugadores adicionales sin descontrolar la plantilla.

---

## 6.7 Mercado de fichajes mejorado

### Problema actual

El mercado permite añadir jugadores, pero si la plantilla crece demasiado se vuelve difícil de gestionar.

### Mejora propuesta

En los nodos de fichajes, ofrecer dos opciones:

```text
Fichar jugador
Cambiar por jugador de mi plantilla
```

### Si hay hueco en plantilla

```text
Plantilla: 19 / 22

[ Fichar ]
[ Cambiar por jugador ]
```

### Si la plantilla está llena

```text
Plantilla: 22 / 22

Solo disponible:
[ Cambiar por jugador ]
```

### Cambio de jugador

El jugador debe elegir a quién sacar de la plantilla.

Ejemplo:

```text
Entra: Nico Williams
Sale: Suso
Coste final: 28.000 €
```

---

## 6.8 Valor de venta

Al cambiar un jugador, el jugador saliente reducirá el coste del nuevo fichaje.

### Fórmula propuesta

```js
function playerValue(player){
  const rarityMult = {
    n: 80,
    b: 150,
    e: 300,
    l: 600
  };

  return ovr(player) * rarityMult[player.rare];
}
```

### Coste final

```js
costeFinal = Math.max(0, costeNuevo - valorVentaJugadorSaliente);
```

### Objetivo

Hacer que el mercado sea más estratégico.

El jugador debe decidir si merece la pena cambiar a un jugador útil por otro mejor, teniendo en cuenta presupuesto, posición, media, energía y rareza.

---

# 7. Roadmap

## v0.2 — Guardado automático

**Estado:** Implementado

Incluye:

- Autoguardado con `localStorage`.
- Continuar partida.
- Borrar partida guardada.
- Recuperación segura desde GitHub Pages.
- Guardado durante draft, temporada, fichajes, eventos y resultados.

---

## v0.3 — Prepartido real

**Objetivo:** que el prepartido tenga impacto visible.

Incluye:

- Estilos con efectos visibles.
- Cálculo de poder actualizado.
- Probabilidad estimada de victoria.
- Explicación de bonus y penalizaciones.
- Guardado del estilo seleccionado.

---

## v0.4 — Energía y cansancio

Incluye:

- Energía en cada jugador.
- Barras de energía.
- Cansancio según estilo.
- Cansancio reducido por `FÍS`.
- Recuperación de suplentes.
- Penalización de rendimiento por baja energía.

---

## v0.5 — Plantilla ampliada

Incluye:

- Draft inicial de 20 jugadores.
- Más suplentes por línea.
- Plantilla máxima de 22 jugadores.
- Adaptación de pantalla de plantilla.
- Ajuste del autoguardado a plantillas más grandes.

---

## v0.6 — Once titular editable

Incluye:

- Cambio manual de titulares.
- Jugadores compatibles por posición.
- Aviso de jugador cansado.
- Recomendación básica de rotación.
- Guardado del once titular.

---

## v0.7 — Mercado inteligente

Incluye:

- Fichar si hay hueco.
- Cambiar jugador si la plantilla está llena.
- Venta parcial del jugador saliente.
- Coste final calculado.
- Límite máximo de plantilla.
- Decisiones de fichaje más estratégicas.

---

# 8. Prioridad actual

La siguiente prioridad debería ser:

```text
v0.3 + v0.4
Prepartido real + energía/cansancio
```

## Orden recomendado de implementación

1. Añadir energía a los jugadores.
2. Mostrar barra de energía en plantilla.
3. Hacer que el estilo afecte al cansancio.
4. Hacer que `FÍS` reduzca el cansancio.
5. Mostrar impacto del estilo en prepartido.
6. Aplicar penalización de energía al cálculo de poder.
7. Guardar todo con el sistema de autosave.
8. Después añadir once titular editable.

---

# 9. Ideas futuras

Estas ideas no son prioridad inmediata, pero pueden dar más personalidad al juego.

## 9.1 Historial de resultados

Guardar una lista de partidos jugados:

```text
J1 — False Nine 2-1 Valencia
J2 — Betis 0-0 False Nine
J3 — False Nine 1-3 Barcelona
```

## 9.2 Logros

Ejemplos:

- Ganar 3 partidos seguidos.
- Ganar a Real Madrid.
- Ganar a Barcelona.
- Fichar un jugador élite.
- Sobrevivir una temporada.
- Ganar la liga.
- Ganar con una plantilla de media baja.

## 9.3 Identidad del club

Añadir personalización:

- Color principal del club.
- Escudo simple generado con iniciales.
- Nombre del estadio.
- Ciudad del club.

## 9.4 Noticias ficticias

Después de partidos o eventos, mostrar titulares.

Ejemplo:

```text
"False Nine sorprende a toda La Liga tras vencer al Atlético"
```

## 9.5 Moral

Añadir una variable de moral de equipo.

```js
morale: 75
```

La moral podría afectar al rendimiento general.

```text
Moral alta → bonus leve
Moral baja → penalización leve
```

## 9.6 Lesiones y sanciones

Añadir disponibilidad de jugadores.

```js
injuryWeeks: 0
suspended: false
```

Primera versión simple:

```text
Jugador no disponible durante 1-3 jornadas
```

---

# 10. Notas de diseño

## Mantener el juego simple

Cada nueva mecánica debe ser entendible en pocos segundos.

Evitar pantallas demasiado cargadas.

## Priorizar feedback visual

Cuando una decisión tenga efecto, el jugador debe verlo.

Ejemplos:

- Cambio de probabilidad.
- Barra de energía.
- Bonus del estilo.
- Aviso de jugador cansado.
- Coste final de fichaje.

## Evitar sistemas demasiado complejos al principio

No implementar todavía:

- Contratos complejos.
- Sueldos.
- Cláusulas.
- Moral individual avanzada.
- Entrenamientos demasiado detallados.
- Drag & drop de alineación.

Primero deben funcionar bien:

- Energía.
- Estilos.
- Rotaciones.
- Fichajes.

---

# 11. Resumen de decisiones tomadas

- El autoguardado funciona y queda como sistema base.
- El estilo de juego debe tener impacto visible.
- El cansancio debe depender del estilo.
- El cansancio debe depender también del atributo `FÍS`.
- Cuanto más `FÍS`, menos energía pierde el jugador.
- La energía debe afectar al rendimiento.
- La plantilla debe ampliarse porque con cansancio hacen falta más suplentes.
- El draft inicial recomendado pasa a 20 jugadores.
- La plantilla máxima recomendada será de 22 jugadores.
- En fichajes se podrá fichar o cambiar por un jugador existente.
- Si la plantilla está llena, solo se podrá cambiar.
- El jugador saliente reducirá el coste final del fichaje.
- El once titular editable se implementará después del sistema de energía.

---

# 12. Próximo paso recomendado

Implementar la versión:

```text
v0.3 — Prepartido real
```

Concretamente:

1. Mostrar efectos del estilo de juego.
2. Calcular poder del equipo según estilo.
3. Mostrar probabilidad estimada.
4. Guardar estilo elegido.
5. Preparar la base para energía/cansancio.

Después implementar:

```text
v0.4 — Energía y cansancio
```

Con:

1. `energy` en cada jugador.
2. Barra visual de energía.
3. Cansancio según estilo.
4. Reducción de cansancio por `FÍS`.
5. Penalización por energía baja.
