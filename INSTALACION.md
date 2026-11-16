<!-- coderabbit-ai-generated:v1 -->

# Dynamic — guía de configuración y versiones

## Qué instalar

Geometry Dash **2.2081**, Windows x64, Geode **5.11.2**. Todas estas versiones son experimentales. El paquete más reciente es **Dynamic Pack 0.8.1 — Caché de portales**; reemplaza Assist 0.8.0 y conserva los mismos cuatro complementos del paquete 0.8.0.

| Mod | Versión del paquete 0.8.1 | Características |
| --- | --- | --- |
| Dynamic Assist | 0.8.1 | Predice el resultado de retrasar una entrada; prueba candidatos con el jugador oculto y colisiones nativas. Catálogo de 3.993 IDs, portal verde 2926 y registro de activación reutilizado entre candidatos. |
| Dynamic Optimizer | 0.2.0 | Evita registrar temporizadores de nodos privados de predicción; protocolo V2 evita el callback de propiedad fuera de la simulación. Conserva compatibilidad V1. |
| Dynamic Particles Lite | 0.2.0 | Omite dibujo, preparación de vértices y cargas gráficas de partículas. Conserva la actualización y caducidad por la ruta nativa. |
| Dynamic Glow Lite | 0.1.2 | Oculta capas de glow comprobadas contra el árbol real. El recorrido temporal usa memoria proporcional a su profundidad. |
| Dynamic Effects Lite | 0.2.0 | Omite brillo, bump y el efecto visual de desplazamiento de spider. El movimiento y la colisión siguen activos. |

Los complementos visuales pueden ocultar pistas del nivel. Son opcionales. La comparación de rendimiento real y la compatibilidad con todos los demás mods requieren pruebas dentro del juego.

### Versiones de referencia publicadas

| Release | Nombre | Diferencias |
| --- | --- | --- |
| dynamic-pack-v0.8.0 | Catálogo y render | Assist 0.8.0, Optimizer 0.2.0, Particles 0.2.0, Glow 0.1.2, Effects 0.2.0. Catálogo de IDs, portal verde, búsqueda sin `std::function`, menos callbacks, recorrido de capas con menos memoria y omisión de trabajo gráfico. |
| dynamic-pack-v0.8.1 | Caché de portales | Assist 0.8.1 y los mismos complementos. Elimina reconstrucciones del registro de portales por candidato. Conserva candidatos, orden de colisiones, horizonte, presupuestos y la política de incertidumbre. |

En 0.8.1 el registro se vacía al comenzar cada búsqueda; dentro de ella cada candidato tiene su propia generación. Un portal puede activarse una vez en cada candidato, sin reservar su nodo de hash de nuevo en cada repetición. Se retienen temporalmente las entradas visitadas por esa búsqueda; se liberan al iniciar la siguiente o al salir del nivel. En un caso sintético de 65 candidatos con 24 portales, se comparan las asignaciones y la misma secuencia de activaciones contra la versión anterior. Esto no mide FPS.

No instales Particles/Glow **0.1.0**: hubo un crash por campos de capas con direcciones inválidas. La validación incorporada en 0.1.1 sigue presente en los paquetes publicados actuales.

## Instalación y controles

1. Cierra GD. Retira los archivos anteriores de los mismos cinco mods de `Geometry Dash/geode/mods` para evitar duplicados.
2. Copia los `.geode` nuevos que necesites. Empieza con Assist y Optimizer; añade los visuales si quieres reducir esos efectos.
3. Abre GD. En Geode, abre Dynamic Assist y sus ajustes. Los valores se conservan al actualizar; una instalación nueva empieza con `Maximum delay = 0`, asistencia desactivada.
4. Entra al nivel. Usa **F7** u ON/OFF para activar/desactivar la asistencia. **F6** aumenta la ventana un tick; después de 64 vuelve a 0. Los botones +/− también cambian la ventana.
5. Los ajustes de los tres mods visuales se aplican al volver a entrar al nivel. El log `Dynamic Optimizer: connected` confirma la conexión del complemento.

## Para qué sirve cada ajuste

| Nombre en Geode | Qué controla | Rango y valor inicial |
| --- | --- | --- |
| Maximum delay (ticks) | Retraso máximo permitido a un clic o liberación tempranos. 0 desactiva Assist. No adelanta clics que llegan tarde ni elige por ti toda la secuencia. Más ventana permite más candidatos y también más retraso. | 0–64; inicial 0 |
| Prediction lookahead (ticks) | Pasos simulados después de cada entrada candidata. Más horizonte cuesta más CPU y supone que no llegarán más clics durante ese futuro. | 32–256; inicial 32 |
| Prediction time budget (ms) | Tiempo máximo de búsqueda por entrada. Un límite mayor permite más trabajo, con posible impacto en el fotograma. | 1–66; inicial 12 |
| Prediction budget per frame (ms) | Presupuesto compartido por las búsquedas del mismo fotograma renderizado. **0 elimina este límite compartido, pero conserva el límite por entrada.** | 0–66; inicial 0 |
| Diagnostic log | Explica decisiones y objetos incompatibles en el log de Geode. Apágalo después de investigar. | Inicial desactivado |
| Show in-level controls | Muestra controles y estadísticas dentro del nivel. Puedes ocultarlos tras configurar. | Inicial activado |

Los ticks son pasos físicos, no necesariamente fotogramas renderizados. Retraso en ms = ticks × duración del paso físico × 1000. Por ejemplo, con pasos de 1/240 s, 8 ticks son unos 33 ms.

Los límites de tiempo son cooperativos: una llamada nativa o la limpieza pueden excederlos. Si una predicción queda incompleta, se conserva la entrada original. El límite de 4 ms no es un acelerador; impide seguir buscando cuando se consume ese tiempo compartido.

## Perfiles sugeridos

Son **puntos de partida sin medir en tu equipo**, no presets incluidos en la interfaz. Introduce los valores manualmente.

| Perfil | Ventana | Horizonte | Por entrada | Por fotograma | Cuándo empezar por él |
| --- | ---: | ---: | ---: | ---: | --- |
| Refresco alto | 4 | 32 | 1 ms | 1 ms | Objetivo de 240 FPS o más; puede completar pocas predicciones. |
| Rendimiento | 4 | 32 | 2 ms | 2 ms | Si notas tirones al pulsar; prioriza limitar el trabajo. |
| Equilibrado | 8 | 32 | 4 ms | 4 ms | Primer ensayo a 60–144 FPS si hay margen de CPU. |
| Más exploración | 16 | 64 | 8 ms | 8 ms | Práctica y CPU con margen; mayor coste y posible retraso perceptible. |

**Para tu objetivo de más predicciones con menos caída de FPS:** empieza por Equilibrado, o Refresco alto si juegas a 240 FPS. Deja el horizonte en 32, usa Optimizer y apaga Diagnostic log. No pongas todos los deslizadores al máximo.

A 60 FPS hay 16,67 ms por fotograma; a 144 FPS, 6,94 ms; a 240 FPS, 4,17 ms. La búsqueda comparte ese tiempo con todo el juego: un presupuesto de 4 ms ya consume casi un fotograma de 240 FPS. No es una reserva de CPU garantizada ni permite calcular los FPS finales.

### Cómo ajustar con una comparación útil

1. Usa el mismo nivel y tramo, en práctica. Mantén iguales FPS objetivo y los demás mods.
2. Compara Assist desactivado y activado con el mismo perfil. Observa tirones al pulsar y correcciones, además del contador de FPS.
3. Si hay tirones, baja primero ambos presupuestos y después la ventana. Mantén horizonte 32.
4. Si las búsquedas se agotan, activa el diagnóstico solo unos intentos: `time-budget` indica falta de tiempo; puedes subirlo de 1 en 1 ms si aceptas el coste.
5. `unsupported-object`, `unsupported-activation` o `ring` requieren examinar el objeto o su lógica. Aumentar el presupuesto no añade esa simulación.
6. Si la trayectoria ya aparece como `already-safe`, no retrasar la entrada es el resultado esperado. `no-safe-tick` significa que los candidatos completados no encontraron un retraso seguro dentro de los supuestos del simulador.

## Qué combinación de mods conviene

- **Conservar pistas visuales:** Assist + Optimizer; los tres Lite desactivados o sin instalar.
- **Reducir partículas:** añade Particles Lite. Mantiene su vida y limpieza, pero las hace invisibles.
- **Escenarios con mucho glow:** prueba Glow Lite además, comprobando que distingues los obstáculos.
- **Reducir efectos del jugador:** añade Effects Lite. Desactívalo si necesitas la pista visual del salto de spider.

Los Lite reducen trabajo visual; no amplían la cobertura de física ni garantizan más candidatos en una búsqueda dominada por colisiones.

## ¿Cuáles son los mejores modos de juego?

No hay una clasificación medida por vehículo. Para comprobar primero que funciona, utiliza un tramo sencillo de **cubo o bola**, sin triggers que cambien el mundo. Eso facilita interpretar los resultados; no demuestra que esos modos siempre tengan mejores FPS.

| Vehículos | Orientación práctica |
| --- | --- |
| Cubo, bola, robot, spider | Comienza con ventana 4–8 y horizonte 32. La física se ejecuta en el jugador oculto. Revisa especialmente cambios de modo y pistas ocultadas por los Lite. |
| Nave y wave | Empieza también con horizonte 32 y ventana 4. La trayectoria depende mucho de mantener/soltar; un horizonte largo supone que no harás más entradas. |
| UFO y swing | Usa primero ventana 4 y horizonte 32; compara con tu ritmo de pulsaciones. El mod no planifica una secuencia completa de clics. |
| Dual | Cobertura restringida a las condiciones de independencia admitidas por el código. Puede conservar la entrada original; no equivale a simular cualquier dual. |
| Plataforma | La asistencia predictiva no está habilitada para plataforma. Los Lite pueden reducir efectos visuales en el nivel. |

## Cobertura y verificación

La lista de **3.993 IDs** sirve como referencia de identificación. Los peligros y sólidos siguen pasando por su tipo nativo, incluso con IDs ausentes. Los orbes que activan grupos personalizados, teletransportes y el futuro de objetos movidos por triggers mantienen limitaciones. Reconocer un ID no implementa su lógica.

Las pruebas y compilaciones publicadas no sustituyen una prueba dentro de GD. No se han medido FPS ni verificado todos los vehículos, niveles, colisiones o combinaciones de otros mods en el juego.
