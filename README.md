# Laboratorio 4 - Segmentación de imágenes con técnicas de agrupación

ISIS2611 - Aprendizaje de Máquina · Caso DermAlpes

*Shaiel Jiménez - 202323846*
*Juan Esteban Triviño - 202315338*

DermAlpes recibe fotografías dermatoscópicas de centros de salud de municipios pequeños y quiere una herramienta que
delimite automáticamente la lesión para que el dermatólogo mida su tamaño y forma. Este laboratorio segmenta la lesión
sin entrenar un modelo supervisado: cada píxel se representa con su color y su posición, ponderados por β y λ, se
agrupa con K-medias eligiendo K con la silueta, y un criterio decide qué grupo es la lesión. El método se ajusta con
15 imágenes de ISIC 2016 y se evalúa sin cambios en otras 5.

## Estructura del repositorio

```
.
├── data/
│   └── raw/                         # Datos originales (no se modifican)
│       ├── ajuste/                  # 15 imágenes con su máscara (*_Segmentation.png)
│       └── prueba/                  # 5 imágenes con su máscara
├── sm.jimenezp1_j.trivinon.ipynb    # Notebook principal (con las celdas ejecutadas)
├── requirements.txt
└── README.md
```

## Cómo correrlo

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

jupyter notebook sm.jimenezp1_j.trivinon.ipynb
```

Hay que ejecutarlo desde la raíz del repositorio, porque lee las imágenes de `data/raw/`. Una ejecución completa
tarda unos 3 minutos (el barrido de configuraciones y valores de K de la Actividad 2 es lo más lento).

Para exportar a HTML:

```bash
jupyter nbconvert --to html sm.jimenezp1_j.trivinon.ipynb
```

El notebook se probó con Python 3.14, scikit-learn 1.9.1, pandas 3.1.0, numpy 2.5.4 y Pillow 13.0.0.

## Contenido del notebook

| Sección | Qué hace |
|---|---|
| Exploración | Mosaico de las imágenes de ajuste, descriptivos de la lesión frente a la piel y efecto de reducir la resolución |
| Actividad 1 | `Pipeline` que convierte la imagen en una matriz [βR, βG, βB, λx, λy] y aplica K-medias |
| Actividad 2 | Diez combinaciones de β y λ más la recomendada por la IA de verificación, K entre 2 y 8 elegido con la silueta |
| Actividad 3 | Tres criterios para decidir qué grupo es la lesión |
| Actividad 4 | Dice, V-measure, precisión y recall en ajuste, regla de decisión y configuración final |
| Actividad 5 | Evaluación de la configuración congelada en las imágenes de prueba |
| Análisis de resultados | Respuesta a las 5 preguntas del enunciado |
| Uso de IA generativa | Declaración, prompts, análisis crítico y actividad de verificación (historial en el anexo) |

## Decisiones principales

- **Una sola resolución.** Todas las imágenes se reducen a 128 píxeles en su lado mayor; las máscaras reducidas
  conservan casi todo el contorno (Dice medio de 0.9885 frente a las originales).
- **Solo importa λ/β.** Con color y posición en [0, 1], la distancia queda β² · (color² + (λ/β)² · posición²), así
  que las combinaciones se comparan por esa razón.
- **Regla escrita antes de las métricas.** Mayor Dice medio; si dos configuraciones quedan a no más de dos errores
  estándar, se desempata por menos fallos, mayor V-measure y la más simple.
- **Prueba intacta.** Las imágenes de prueba no se abrieron hasta congelar la configuración.

## Resultados

La configuración elegida es solo color (β = 1, λ = 0) con el grupo más oscuro como lesión y K elegido por silueta.

| Conjunto | Imágenes | Dice medio | Desviación |
|---|---|---|---|
| Ajuste | 15 | 0.848 | 0.101 |
| Prueba | 5 | 0.848 | 0.124 |

Las configuraciones con un peso moderado de la posición quedaron empatadas con ella. Los errores más comunes son
zonas oscuras que no son lesión, como las esquinas del lente (el peor caso de prueba, con Dice de 0.629), y bordes
claros de la lesión que quedan fuera de la máscara.

## Entrega

- Notebook (`.ipynb` y `.html`) con las celdas ejecutadas.
- Video explicativo para DermAlpes (máximo 3 minutos).
- Fecha límite: **2 de noviembre, 8:00 p.m.** (penalización del 30% hasta el 3 de noviembre, 2:00 a.m.; después de
  esa hora, nota 0).
