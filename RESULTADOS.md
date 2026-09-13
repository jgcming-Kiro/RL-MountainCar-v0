# Comparacion de Q-Learning y DQN en MountainCar-v0

## Metodo

La ejecucion uso la semilla 42. Q-Learning se entreno durante 20.000 episodios y DQN durante 2.500. La evaluacion posterior uso 100 episodios con una politica determinista y semillas distintas a las del entrenamiento. La recompensa equivale al negativo de la cantidad de pasos; por esta razon, un valor cercano a cero es mejor. Un episodio tiene exito cuando el auto llega a la bandera antes del limite de 200 pasos.

Los tiempos corresponden a una CPU Apple Silicon y pueden cambiar en otro equipo.

## Hiperparametros

| Parametro | Q-Learning | DQN |
|---|---:|---:|
| Tasa de aprendizaje | 0,1 | 0,001 |
| Gamma | 0,99 | 0,99 |
| Epsilon inicial | 1,0 | 1,0 |
| Epsilon minimo | 0,01 | 0,01 |
| Decaimiento de epsilon | 0,9995 | 0,995 |
| Discretizacion | 20 x 20 celdas | No aplica |
| Capas ocultas | No aplica | 2 capas de 128 unidades |
| Tamano del lote | No aplica | 64 |
| Memoria de repeticion | No aplica | 100.000 transiciones |
| Copia a la red objetivo | No aplica | Cada 10 episodios |
| Persistencia de accion exploratoria | No aplica | 20 pasos |

## Resultados medidos

| Medida | Q-Learning | DQN |
|---|---:|---:|
| Episodios de entrenamiento | 20.000 | 2.500 |
| Tiempo de entrenamiento | 37,51 s | 317,85 s |
| Mejor media movil de 100 episodios | -125,39 | -121,32 |
| Episodio de esa media | 15.860 | 1.506 |
| Recompensa media en evaluacion | -149,23 | -108,32 |
| Desviacion estandar | 16,84 | 9,92 |
| Mejor episodio evaluado | -118 | -83 |
| Peor episodio evaluado | -171 | -115 |
| Exitos | 100/100 | 100/100 |

![Curvas de entrenamiento](results/training_curves.svg)

## Lectura de los resultados

Q-Learning necesito mas episodios, aunque cada episodio fue barato: actualizar una celda de la tabla requiere pocas operaciones. La curva presento oscilaciones y la evaluacion termino en -149,23. Llegó a la bandera en las 100 pruebas, pero uso unos 41 pasos mas que DQN en promedio.

DQN aprendio con 2.500 episodios y obtuvo -108,32 durante la evaluacion. Supero el umbral usual de -110. Su costo por episodio fue mayor debido al muestreo de la memoria y al ajuste de la red en cada paso. El entrenamiento tambien oscilo. Para evitar que una fase posterior reemplazara una politica mejor, el programa conservo los pesos asociados a la mejor media movil de 100 episodios.

La tabla permite inspeccionar cada valor y necesita pocos recursos, pero pierde informacion al convertir posicion y velocidad en celdas. La red usa los valores continuos y alcanzo una politica mas rapida. A cambio, requiere PyTorch, una memoria de repeticion y una red objetivo. Tambien depende mas de los hiperparametros.

La exploracion aleatoria independiente no fue suficiente para DQN. El auto necesita empujes sostenidos en una direccion para acumular impulso. El agente mantiene una accion exploratoria durante 20 pasos, con lo cual puede descubrir trayectorias que llegan a la bandera. La evaluacion desactiva esta conducta y usa la accion con mayor valor estimado.

## Archivos de evidencia

- `results/qlearning_training.csv`: recompensa y media movil por episodio.
- `results/dqn_training.csv`: recompensa y media movil por episodio.
- `results/summary.json`: tiempos y evaluacion de 100 episodios.
- `results/training_curves.svg`: comparacion grafica.
- `saves/qlearning_mountaincar.pkl` y `saves/dqn_mountaincar.pt`: modelos locales, excluidos de Git por tamano y portabilidad.

## Reproduccion

```bash
uv sync
uv run python scripts/run_experiments.py
uv run mountaincar load qlearning --eval
uv run mountaincar load dqn --eval
```
