# Optimización de Campañas Publicitarias con Multi-Armed Bandits

### Un estudio comparativo: A/B Testing, ε-greedy, UCB1 y Thompson Sampling

## Objetivo

Comparar de forma rigurosa, mediante simulación Monte Carlo, el rendimiento de cuatro
estrategias de asignación de tráfico en un escenario de marketing digital, donde varias
creatividades de anuncio compiten por conversión.

## Pregunta de investigación

¿Qué algoritmo minimiza el *regret* acumulado, y cómo cambia el rendimiento relativo de
cada uno según la dificultad del escenario (diferencias entre tasas de conversión,
número de brazos, horizonte temporal)?

Los cuatro algoritmos se evalúan con el mismo nivel de rigor, sin asumir de antemano cuál
debería funcionar mejor. Las conclusiones se basan en las métricas obtenidas (regret
acumulado, significancia estadística y velocidad de convergencia), no en la reputación de
cada método en la literatura.

## Metodología

- **Modelo:** bandits Bernoulli, cada brazo (creatividad de anuncio) tiene una
  probabilidad de conversión real, desconocida para los algoritmos.
- **Escenarios simulados** (dificultad creciente):
  - **Fácil**: diferencias grandes entre tasas de conversión (2%, 5%, 10%)
  - **Difícil**: diferencias mínimas (4.8%, 5.0%, 5.2%)
  - **Pocos brazos**: k = 3
  - **Muchos brazos**: k = 15, con un único brazo claramente superior
- **Horizonte temporal:** T = 5,000 rondas por simulación
- **Repeticiones Monte Carlo:** 500 corridas independientes por combinación
  algoritmo-escenario, para obtener resultados estadísticamente robustos con bandas de
  confianza
- **Métricas de evaluación:**
  - Regret acumulado (media ± desviación estándar)
  - Significancia estadística de las diferencias (test t de Welch, con corrección de
    Bonferroni por comparaciones múltiples)
  - Velocidad de convergencia (ronda a partir de la cual el algoritmo elige el brazo
    óptimo de forma sostenida en ≥95% de las corridas)

## Algoritmos implementados

Todos implementados desde cero en Python (sin librerías de bandits), documentados con
explicación matemática y de código en el propio notebook:

| Algoritmo | Estrategia |
|---|---|
| **A/B Testing** | Reparto de tráfico equitativo (round-robin) durante todo el experimento, sin adaptación |
| **ε-greedy** (ε=0.1) | Exploración aleatoria un 10% de las rondas, explotación del mejor brazo observado el resto |
| **UCB1** | Elige el brazo con mayor límite superior de confianza (media + término de incertidumbre) |
| **Thompson Sampling** | Enfoque bayesiano: mantiene una distribución Beta por brazo y muestrea de ella en cada ronda |

## Resultados principales

- **Fácil y Pocos brazos:** Thompson Sampling obtiene el menor regret de forma
  estadísticamente significativa frente a los otros tres (p < 0.0001), y es el único que
  converge de forma sostenida al 95% dentro del horizonte simulado.
- **Difícil:** Thompson Sampling tiene la media de regret más baja y es estadísticamente
  distinto de los demás, pero A/B Testing, ε-greedy y UCB1 no son distinguibles entre sí
  con la evidencia disponible. Ningún algoritmo converge de forma sostenida, el ruido
  domina sobre la señal con diferencias tan pequeñas entre brazos.
- **Muchos brazos:** hallazgo principal del proyecto. Thompson Sampling y ε-greedy no
  muestran diferencia significativa en regret medio (p = 0.87), pero Thompson Sampling
  tiene un tercio de la varianza de ε-greedy, lo que lo hace más consistente y predecible.
- **UCB1** no domina en ningún escenario del proyecto, pese a sus garantías teóricas.
  Su término de exploración resulta más conservador de lo esperado en este horizonte
  temporal.
- **ε-greedy con epsilon fijo nunca converge en sentido estricto**, por diseño: sigue
  explorando aleatoriamente un 10% de las rondas incluso tras miles de iteraciones.

Detalle completo de resultados, gráficas y tablas en el notebook.

## Estructura del notebook

1. Entorno de simulación (`BanditEnvironment`) y escenarios predefinidos
2. Implementación de los cuatro algoritmos, cada uno probado individualmente
3. Métrica de regret y motor de simulación Monte Carlo
4. Ejecución completa: 4 algoritmos × 4 escenarios × 500 repeticiones
5. Curvas de regret acumulado con bandas de confianza
6. Tabla resumen de regret final por algoritmo y escenario
7. Tests de significancia estadística (test t de Welch + corrección de Bonferroni)
8. Velocidad de convergencia
9. Conclusiones, limitaciones y extensiones futuras

## Requisitos

```
numpy
matplotlib
scipy
pandas
```

Instalación:

```bash
pip install numpy matplotlib scipy pandas
```

## Uso

El notebook guarda los resultados de la simulación Monte Carlo (`bandit_results.pkl`)
tras la primera ejecución, para no tener que repetir la simulación completa (~40 millones
de tiradas simuladas) cada vez que se abre el proyecto. Este archivo no se versiona en el repositorio (ver `.gitignore`).

## Limitaciones y extensiones futuras

- Solo se probó ε-greedy con epsilon fijo; sería interesante un análisis de sensibilidad o una
  variante con epsilon decreciente (annealing).
- El escenario Difícil sugiere que un horizonte mayor a 5,000 rondas podría ser necesario
  para distinguir algoritmos cuando las diferencias entre brazos son mínimas.
- El proyecto usa bandits estacionarios (las probabilidades no cambian con el tiempo);
  extenderlo a un entorno no estacionario (ej. fatiga publicitaria) sería un paso natural.
- No se implementaron bandits contextuales (LinUCB, Thompson Sampling contextual), que
  incorporarían información del usuario a la decisión.

## Autor

Hugo Rodríguez Tristancho