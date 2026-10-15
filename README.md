<div align="center">

# 🚇 MovilidadData — Analítica de Transporte Público

### Data Product para la Línea 1 del Metro de Lima

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)
![Python](https://img.shields.io/badge/Python-Pandas%20%7C%20NumPy-3776AB?logo=python&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-Clean%20Architecture%20%2F%20DDD-0FA3A0)
![Status](https://img.shields.io/badge/status-en%20desarrollo-FFB400)

</div>

---

## 📌 Descripción del proyecto

Un **Producto de Datos** (*Data Product*) avanzado diseñado para afrontar los
retos de congestión y eficiencia en el transporte público. Este proyecto
analiza [datos abiertos](https://serviciosdigitales.ositran.gob.pe:8443/PortalDatosOsitran/reporte.jsp?code=23) de pasajeros para identificar patrones temporales y
proponer visualizaciones que apoyen la toma de decisiones, con foco en la
**Línea 1 del Metro de Lima**, operada por **Tren Urbano de Lima S.A.** El
Estado peruano, mediante el **MTC**, firma la concesión y delega en
**Ositrán** la fiscalización del cumplimiento de la empresa.

El sistema está compuesto por dos capas desacopladas:

### 1. Capa de Transformación, Enriquecimiento y Agregación (*Serving Layer*)

Desarrollada en **Python (Pandas & NumPy)**, actúa como el motor de
transformación del Data Product. Su propósito es procesar los datos crudos
de tráfico de pasajeros de la Línea 1, estandarizar sus variables mediante
expresiones regulares (**Regex**) y aplicar lógica estadística basada en
**percentiles** para categorizar el volumen de usuarios. El pipeline limpia
los nombres de las estaciones y unifica los formatos de tiempo para
estructurar patrones de movilidad urbana en la Línea 1.

Mediante la **simulación de ingresos** y la **segmentación automatizada de
perfiles** (Pasajeros Regulares, Estudiantes, Exonerados), los datos se
preparan bajo estándares de **Datos Abiertos**.

Finalmente, implementa una estrategia de **Serving Layer**, distribuyendo el
dataset en **micromatrices agregadas y serializadas en formato JSON**. Estas
estructuras quedan optimizadas en tamaño y jerarquía para ser consumidas de
forma asíncrona por el frontend del Data Product, garantizando consultas de
alta velocidad y cargas eficientes en los componentes visuales interactivos.

| Archivo | Claves esperadas |
|---|---|
| `obj1_patrones_temporales.json` | `Año, mes, dia_semana, rango_horario, numero_de_pasajeros` |
| `obj2_distribucion_infraestructura.json` | `Estación, Número_de_Pasajeros, porcentaje_participación, categoria_operativa` |
| `obj3_tendencia_mensual_consolidado.json` | `mes, Volumen (=tipo de tarifa), Total_Pasajeros, Total_Ingresos, Porcentaje_Pasajeros, Porcentaje_Ingresos` |
| `obj4_correlaciones_demanda.json` | `rango_horario` + una columna numérica por cada tipo de tarifa |
| `obj5_anomalias_alertas.json` | `fecha, Estación, rango_horario, Número_de_Pasajeros` |

### 2.  Capa de Consumo y Orquestación del Frontend

Última milla del Data Product: las micromatrices optimizadas en formato JSON
son consumidas de manera asíncrona por la aplicación cliente. Este sistema
implementa una **Arquitectura Limpia (Clean Architecture)** basada en el
**Diseño Guiado por el Dominio (DDD)** en **React + TypeScript**, alimentada
por el pipeline de *feature engineering* desarrollado en **Python**.

---
## 🧱 Arquitectura Desaclopada (Decoupled Architectures)

```
├── pipeline/                      # Capa 1 — Python (Pandas / NumPy)
│   └── ...                        # limpieza, regex, percentiles, serialización a JSON
│
└── frontend/                      # Capa 2 — React + TypeScript
    ├── public/data/                obj1...obj5.json (Serving Layer consumido en runtime)
    └── src/
        ├── domain/                 entidades + puertos (sin dependencias externas)
        ├── application/            casos de uso + DTOs
        ├── infrastructure/         mappers + repositorios + cliente HTTP
        ├── presentation/           pages, components, hooks
        └── shared/                 DI container, utilidades

```

---

## 🎯 Objetivos del proyecto

1. Identificar **patrones temporales** en la demanda de transporte público
2. Evaluar la **distribución del flujo de pasajeros** por estaciones y concesiones
3. Analizar la **segmentación por tipo de tarifa y pase**
4. Detectar **correlaciones** entre variables críticas
5. Visualizar **tendencias y anomalías** para la toma de decisiones

---

## 🏗️ Arquitectura del sistema (Clean Architecture & DDD)

El frontend sigue estrictamente la regla de dependencia de Clean
Architecture: las capas internas (`domain`) no conocen nada de las externas,
y toda dependencia apunta hacia adentro.

```
src/
  domain/            → entidades + puertos (interfaces). No importa nada de React ni de fetch.
    entities/           PatronTemporal, EstacionDistribucion, TendenciaMensual,
                         CorrelacionDemanda, Anomalia (Alerta)
    repositories/        Repository.ts (contratos que la infraestructura implementa)

  application/        → casos de uso: lógica de negocio pura, testeable sin UI.
    usecases/            GetHourlyDemand, GetWeekdayComparison, GetHeatmap,
                          GetStationRanking, GetFareComposition, GetVolumeRevenueGap,
                          GetCorrelationSeries, DetectAnomalies
    dto/                  ViewModels.ts — lo que los casos de uso devuelven a la UI

  infrastructure/     → detalles técnicos: cómo se leen los datos.
    http/JsonFileClient.ts   fetch con caché en memoria
    mappers/                 JSON crudo (con tildes/mayúsculas reales) → entidades de dominio
    repositories/             implementación concreta de cada puerto del dominio

  presentation/       → todo lo que es React.
    layout/AppShell.tsx       nav lateral + <Outlet/>
    components/               charts (recharts), filtros, cards, estados de carga/error
    hooks/                    un hook por objetivo (usePatronesTemporales, etc.)
    pages/                    una página por objetivo + portada

  shared/
    di/container.ts     composition root: única línea que sabe que los datos son JSON
    utils/               orden de meses/días, estadística (media, desv. estándar, Pearson)
```

`src/shared/di/container.ts` es el **único punto del frontend** que sabe que
los datos provienen de archivos JSON. Si el pipeline de Python migra a una
API REST, solo se reescriben los 5 repositorios de `infrastructure/` — el
dominio, los casos de uso y la UI no cambian.

---



## 🛠️ Stack tecnológico

| Capa | Tecnología |
|---|---|
| **Procesamiento y feature engineering** | Python · Pandas · NumPy · Regex |
| **Serving Layer** | JSON (micromatrices agregadas) |
| **Frontend** | React 19 · TypeScript · Vite |
| **Arquitectura** | Clean Architecture · Domain-Driven Design (DDD) |
| **Visualización de datos** | Recharts |
| **Enrutamiento** | React Router |
| **Diseño / UI** | Design system propio (CSS variables, tipografía Space Grotesk / IBM Plex) |

---

Los 5 JSON del Serving Layer se leen desde `public/data/` en tiempo de
ejecución (no se importan como módulos), por lo que reemplazar el pipeline
de Python que los genera no requiere tocar el código del frontend.

---

## 📊 Vistas del dashboard

| Vista | Objetivo que responde |
|---|---|
| `/patrones` | Patrones temporales — picos horarios detectados automáticamente |
| `/estaciones` | Distribución del flujo por estación y categoría operativa |
| `/tarifas` | Segmentación por tipo de tarifa/pase vs. recaudación |
| `/correlaciones` | Correlación de Pearson entre tarifas y rango horario |
| `/anomalias` | Detección de anomalías por z-score (línea base por estación + hora) |

---

## ✍️ Autor

Proyecto desarrollado por Robert Ushiñahua como parte de un portafolio de analítica de datos
aplicada a Tráfico de pasajeros en la Linea 1 del Metro de Lima. Contribuciones y sugerencias son bienvenidas.
