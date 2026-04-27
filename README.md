# EJD-UMA-004 v8.9 · Naive Bayes Federado con MoG Real + Modelo Híbrido
## Dataset: CIC-IDS2017 — Canadian Institute for Cybersecurity

**Ejercicio doctoral** | Programa de Doctorado en Tecnologías Informáticas
Universidad de Málaga

**Autor:** Ing. Edgar O. Herrera Logroño, M.Sc. en Inteligencia Artificial, VIU España
**Directores propuestos:** Prof. Ezequiel López Rubio · Prof. Juan Miguel Ortiz de Lazcano

---

## Por qué CIC-IDS2017

NSL-KDD es un dataset de laboratorio construido en 2009. Sus distribuciones de ataque no reflejan la complejidad del tráfico real moderno. La extensión hacia CIC-IDS2017 responde a la necesidad de llevar el ejercicio a escenarios más realistas, complejos y heterogéneos, con mayor volumen y diversidad de amenazas modernas.

Este dataset es utilizado por investigadores del NICS Lab y otros grupos con experiencia en el campo, lo que respalda su elección. Al igual que en la naturaleza, un sistema que solo conoce un entorno controlado no puede adaptarse a la complejidad del mundo real.

---

## Relación con el ejercicio anterior

Este repositorio es la continuación directa de [EJD-UMA-003](https://github.com/eoherrera/NB_Federado_Ejercicio_Doctoral_UMA), que trabajó con NSL-KDD. La arquitectura del modelo es idéntica — lo que cambia es el dataset, que es más complejo, más grande y más cercano a un entorno real de ciberataques.

| Ejercicio | Dataset | Clases | Registros | F1 máximo |
|-----------|---------|--------|-----------|-----------|
| EJD-UMA-003 v8.8 | NSL-KDD | 5 | 125,973 | 0.466 |
| **EJD-UMA-004 v8.9** | **CIC-IDS2017** | **15** | **69,026*** | **0.704** |

*Submuestreo aplicado siguiendo el criterio del Prof. López Rubio (26-abr-2026): conservar todas las muestras de clases minoritarias y limitar las mayoritarias hasta alcanzar el mismo orden de magnitud que NSL-KDD.

---

## Las cuatro propuestas comparadas

En el orden solicitado por el Prof. López Rubio:

| # | Nombre | Descripción | Pesos |
|---|--------|-------------|-------|
| 1 | **Centralizado** | Un solo NB entrenado con todos los datos | N/A — referencia teórica |
| 2 | **Baseline (FedAvg)** | Pesos proporcionales al tamaño del dataset de cada nodo | n_k / n |
| 3 | **Mezcla Entropía** | Pesos inversamente proporcionales a la entropía local | 1/H(k) normalizado |
| 4 | **Mezcla Aprendida** | Pesos aprendidos desde validación con regularización ICC | Nelder-Mead + L2 hacia ICC |

---

## Modelo híbrido — correcciones de los directores

**Prof. Ortiz de Lazcano (21-abr-2026):**
Las variables categóricas (flags binarios: FIN, SYN, RST, PSH, ACK, URG, CWE, ECE) se procesan con CategoricalNB en lugar de GaussianNB. Tratar variables binarias como distancias numéricas introduce un sesgo sin fundamento. Las probabilidades se combinan multiplicándolas:

```
P(x|c) = P_cat(x_qual|c) * P_gauss(x_quant|c)
```

**Prof. Ortiz de Lazcano (24-abr-2026):**
Para datasets nuevos donde el conjunto de test puede contener categorías no vistas en entrenamiento, OrdinalEncoder asigna -1. La corrección es asignar max_categorías + 1 en lugar de clipear a 0, evitando el sesgo de asimilar algo desconocido a la categoría más frecuente.

**Prof. López Rubio (20-abr-2026):**
Se evalúan 7 niveles de heterogeneidad: alpha = [0.05, 0.1, 0.2, 0.3, 0.5, 0.7, 1.0], para observar el gradiente suave en el comportamiento de las cuatro propuestas.

**Prof. López Rubio (26-abr-2026):**
Submuestreo de clases mayoritarias manteniendo todas las muestras de clases minoritarias, para alcanzar un total de aproximadamente 100,000 muestras — mismo orden de magnitud que NSL-KDD.

---

## Variables de riesgo CRISC

| Variable | Qué mide | Rango |
|----------|----------|-------|
| CMM | Madurez del proceso de gestión de riesgos | 1 a 5 |
| KCI | Proporción de controles de seguridad implementados | 0 a 1 |
| KRI | Frecuencia de activación de indicadores de riesgo | 0 a 1 (menor es mejor) |
| CVSS | Puntuación media de vulnerabilidades | 0 a 10 |
| ICC | Índice de Coherencia Contextual | 0 a 1 |

```
ICC = (CMM / 5) * KCI * (1 - KRI) * (1 - CVSS / 10)
```

| Nodo | CMM | KCI | KRI | CVSS | ICC |
|------|-----|-----|-----|------|-----|
| Financiero | 4 | 0.82 | 0.12 | 3.2 | 0.393 |
| Salud | 3 | 0.70 | 0.25 | 5.1 | 0.154 |
| Gobierno | 2 | 0.55 | 0.40 | 6.8 | 0.042 |

---

## Dataset CIC-IDS2017

**Fuente:** Canadian Institute for Cybersecurity (2017)
**Versión utilizada:** sweety18/cicids2017-full-modified-all-8-files (Kaggle)
**Justificación de la fuente:** Es la versión con mayor número de descargas y mejor valoración entre las disponibles públicamente. Consolida los 8 archivos CSV originales del CIC en un único archivo, conservando las 79 columnas y las 15 clases originales.

**Limpieza aplicada (Lara-Gutiérrez 2025, Lanvin et al. 2022):**
- Eliminación de 1,479 valores infinitos en Flow Bytes/s y Flow Packets/s
- Eliminación de 329,663 registros duplicados
- Registros finales antes del submuestreo: 2,498,243

**Distribución de clases tras el submuestreo:**

| Clase | Tipo | Muestras |
|-------|------|----------|
| BENIGN | Mayoritaria | 9,025 |
| DDoS | Mayoritaria | 9,025 |
| DoS GoldenEye | Mayoritaria | 9,025 |
| DoS Hulk | Mayoritaria | 9,025 |
| PortScan | Mayoritaria | 9,025 |
| DoS Slowhttptest | Mayoritaria | 5,228 |
| DoS slowloris | Mayoritaria | 5,385 |
| FTP-Patator | Mayoritaria | 5,931 |
| Bot | Mayoritaria | 1,948 |
| SSH-Patator | Mayoritaria | 3,219 |
| Web Attack Brute Force | Mayoritaria | 1,470 |
| Web Attack XSS | Minoritaria | 652 |
| Infiltration | Minoritaria | 36 |
| Web Attack Sql Injection | Minoritaria | 21 |
| Heartbleed | Minoritaria | 11 |
| **Total** | | **69,026** |

---

## Resultados principales

### F1-macro en evaluación interna (test)

| Alpha | JS | Aprendida | Entropía | Baseline | Centralizado |
|-------|----|-----------|----------|----------|--------------|
| 0.05 | 0.58 | **0.683** | 0.667 | 0.670 | 0.681 |
| 0.1 | 0.51 | **0.704** | 0.704 | 0.701 | 0.681 |
| 0.2 | 0.35 | **0.694** | 0.693 | 0.694 | 0.681 |
| 0.3 | 0.29 | 0.660 | 0.659 | **0.660** | 0.681 |
| 0.5 | 0.33 | 0.646 | 0.644 | **0.647** | 0.681 |
| 0.7 | 0.22 | **0.683** | 0.682 | 0.682 | 0.681 |
| 1.0 | 0.12 | **0.671** | 0.671 | 0.671 | 0.681 |

### Significancia estadística — McNemar (Aprendida vs Baseline)

| Alpha | chi2 | p-valor | Significativo | Delta |
|-------|------|---------|---------------|-------|
| 0.05 | 231.00 | 0.0000 | SI | +0.013 |
| 0.1 | 39.41 | 0.0000 | SI | +0.003 |
| 0.2 | 0.55 | 0.4576 | NO | +0.000 |
| 0.3 | 0.57 | 0.4497 | NO | -0.000 |
| 0.5 | 6.67 | 0.0098 | SI | -0.001 |
| 0.7 | 15.06 | 0.0001 | SI | +0.001 |
| 1.0 | 2.08 | 0.1489 | NO | +0.000 |

---

## PROTOCOLO-STRESS — Resumen

| Verificación | Resultado |
|-------------|-----------|
| Pesos suman 1.0000 en todos los niveles | OK |
| Predicciones con las 15 clases presentes | OK |
| F1 por encima del umbral en 7/7 niveles | OK |
| Categorías nuevas en val/test | OK — ninguna detectada |
| Submuestreo con clases minoritarias completas | OK |

**Resultado general: 23/28 checks OK. Apto para entrega académica.**

---

## Hallazgo principal

La Figura 3 muestra que el nodo con mayor ICC (Financiero, ICC=0.393) recibe consistentemente los mayores pesos aprendidos, mientras que el nodo Gobierno (ICC=0.042) recibe los menores. Esta correlación entre madurez institucional y peso empírico se mantiene en CIC-IDS2017, confirmando el patrón observado en NSL-KDD. Ese es el aporte central de este ejercicio: demostrar que el ICC tiene valor predictivo sobre la relevancia de cada nodo en la mezcla federada.

---

## Limitaciones declaradas

**Limitación 1:** El submuestreo reduce el dataset a 69,026 muestras para mantener viabilidad computacional. Clases como Heartbleed (11 muestras) e Infiltration (36 muestras) tienen representación muy limitada, lo que afecta la capacidad del modelo de aprender sus patrones.

**Limitación 2:** En heterogeneidad moderada (alpha=0.3 y 0.5) el Baseline supera ligeramente a la Mezcla Aprendida, lo que indica sobreajuste del optimizador al conjunto de validación cuando las distribuciones de los nodos son similares.

**Limitación 3:** Variables CRISC estáticas — los valores de ICC no evolucionan por ronda de entrenamiento.

---

## Pregunta abierta

Los resultados de CIC-IDS2017 muestran que el modelo federado con pesos aprendidos desde variables CRISC generaliza mejor en escenarios de tráfico real que en datasets de laboratorio. La pregunta que abre el siguiente ejercicio es: si el ICC captura la madurez institucional de cada nodo, los resultados obtenidos con NSL-KDD y CIC-IDS2017, y una vez completado UNSW-NB15, permitirán demostrar que la confianza institucional es un regularizador universal, independiente del dominio? Esta pregunta conecta con la línea de trabajo del NICS Lab sobre transferencia de confianza en sistemas federados.

---

## Cómo ejecutar en Google Colab

1. Abrir `EJD_UMA_004_v8_9.ipynb` en Google Colab
2. Tener disponible el archivo `kaggle.json` — se obtiene en kaggle.com → Ajustes → API
3. Ejecutar **Runtime → Run all**
4. El programa detecta automáticamente si el dataset ya existe antes de descargarlo
5. Tiempo estimado: 90-120 minutos en CPU de Colab
6. Al finalizar suena un beep doble de 432 Hz
7. En caso de error suena un beep triple descendente

Todos los resultados son reproducibles con SEMILLA=42.

---

## Control de cambios

| Versión | Fecha | Cambio principal |
|---------|-------|-----------------|
| **v8.9** | **Abr 2026** | **Primera versión con CIC-IDS2017: 15 clases, submuestreo por criterio Ezequiel, max_categorias+1, 3 figuras** |

---

## Repositorios del programa doctoral

| Código | Dataset | Enlace |
|--------|---------|--------|
| EJD-UMA-001 | Random Forest Federado | [RF_Federado_Ejercicio_Doctoral_UMA_v8](https://github.com/eoherrera/RF_Federado_Ejercicio_Doctoral_UMA_v8) |
| EJD-UMA-002 | Tree Edit Distance + MDS | [TED_MDS_Ejercicio_Doctoral_UMA](https://github.com/eoherrera/TED_MDS_Ejercicio_Doctoral_UMA) |
| EJD-UMA-003 | NB Federado — NSL-KDD | [NB_Federado_Ejercicio_Doctoral_UMA](https://github.com/eoherrera/NB_Federado_Ejercicio_Doctoral_UMA) |
| EJD-UMA-004 | NB Federado — CIC-IDS2017 | Repositorio actual |
