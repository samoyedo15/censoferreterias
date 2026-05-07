# Censo Digital de Ferreterías — Cementos Argos
### Beca Ser ANDI · EAFIT | Mayo 2026

Sistema automatizado de inteligencia comercial para el canal ferretero de Cementos Argos. Recolecta ferreterías desde Google Maps API, las clasifica con IA generativa (Groq + Llama 3.3), y las visualiza en un dashboard interactivo con mapa, gráficas y chatbot.

---

## Resultados en Antioquia

| Indicador | Valor |
|---|---|
| Municipios buscados | 51 |
| Ferreterías únicas recolectadas | 879 |
| Clasificadas por Groq IA | ~180 (20%) |
| Clasificadas por reglas Python | ~699 (80%) |
| Clientes Argos identificados | 49 de 104 (47.1%) |
| **Prospectos nuevos para Argos** | **830** |
| Tiempo total del pipeline | ~12 minutos |
| Costo de infraestructura | $0 |

---

## Arquitectura del pipeline

```
piloto_antioquia_argos.py   →   piloto_antioquia_argos_raw.csv
        ↓ Google Maps API
limpieza_antioquia_argos.py →   inteligencia_antioquia_argos.csv
        ↓ Groq API (Llama 3.3)
cruce_datasets.py           →   cruce_censo_vs_argos.xlsx
        ↓ Algoritmo 5 criterios
generar_reporte.py          →   reporte_mercado_antioquia.txt
        ↓ Groq API (Llama 3.3)
dashboard.py                →   http://localhost:8501
        ↓ Streamlit + Folium + Plotly
```

---

## Instalación

```bash
# Clonar el repositorio
git clone https://github.com/usuario/censo-ferreterias-argos.git
cd censo-ferreterias-argos

# Instalar dependencias
pip install streamlit folium streamlit-folium plotly pandas openpyxl requests python-dotenv

# Configurar API keys
cp .env.example .env
# Editar .env con tus claves
```

### Archivo .env
```
GOOGLE_MAPS_KEY=AIzaSy...
GROQ_API_KEY=gsk_...
```

---

## Uso

### Opción 1 — Desde el dashboard (recomendado)
```bash
streamlit run dashboard.py
```
Selecciona la ciudad en el sidebar y presiona **▶ Ejecutar pipeline**.

### Opción 2 — Desde la terminal paso a paso
```bash
python piloto_antioquia_argos.py    # ~4.5 minutos
python limpieza_antioquia_argos.py  # ~8 minutos
python cruce_datasets.py            # ~2 minutos
python generar_reporte.py           # ~10 segundos
streamlit run dashboard.py
```

### Opción 3 — Para demo rápida (Pereira, ~2 minutos)
```bash
python piloto_pereira.py
python limpieza_pereira.py
streamlit run dashboard.py
```

---

## Estructura del proyecto

```
censo-ferreterias-argos/
├── piloto_antioquia_argos.py    # Recolección Google Maps — Antioquia
├── piloto_pereira.py            # Recolección Google Maps — Pereira (demo)
├── piloto_barranquilla.py       # Recolección Google Maps — Barranquilla
├── piloto_cartagena.py          # Recolección Google Maps — Cartagena
├── piloto_popayan.py            # Recolección Google Maps — Popayán
├── limpieza_antioquia_argos.py  # Clasificación IA + cruce Argos
├── limpieza_pereira.py          # Clasificación IA — Pereira
├── limpieza_barranquilla.py     # Clasificación IA — Barranquilla
├── cruce_datasets.py            # Cruce censo vs base Argos
├── generar_reporte.py           # Reporte ejecutivo con Groq
├── dashboard.py                 # Dashboard Streamlit (8 pestañas)
├── .env.example                 # Plantilla de variables de entorno
├── requirements.txt             # Dependencias Python
└── docs/                        # Documentación técnica
    ├── documentacion_piloto_definitivo.docx
    ├── documentacion_limpieza_definitiva.docx
    ├── documentacion_cruce_datasets.docx
    ├── documentacion_dashboard_final.docx
    └── preguntas_respuestas.docx
```

---

## Tecnologías utilizadas

| Tecnología | Versión | Uso |
|---|---|---|
| Python | 3.11+ | Lenguaje principal |
| Google Maps Places API (New) | v1 | Recolección de datos |
| Groq + Llama 3.3 70B | — | Clasificación IA y reportes |
| Streamlit | 1.32+ | Dashboard web |
| Folium | 0.16+ | Mapas interactivos |
| Plotly | 5.18+ | Gráficas interactivas |
| Pandas | 2.0+ | Procesamiento de datos |

---

## Dashboard — 8 pestañas

| Pestaña | Descripción |
|---|---|
| 🗺️ Mapa | Ferreterías geolocalizadas con zoom, popup y color dinámico |
| 📊 Gráficas | Barras, donut, radar chart, histograma, heatmap |
| 🏆 Prospectos | Score ≥ 60, descarga CSV para el asesor |
| ⚠️ Competencia | Clientes Cemex/Holcim/Construrama detectados |
| 📋 Datos | Tabla completa con buscador en tiempo real |
| 🤖 Asistente IA | Chatbot con Groq que responde sobre el dataset |
| 🔗 Cruce vs Argos | Clientes conocidos vs prospectos nuevos con mapa |
| 🎯 Prioridades | Cards ejecutivos por ciudad con prioridades A/B/C/D |

---

## Algoritmo de clasificación

El sistema usa dos capas para clasificar si una ferretería vende cemento:

**Capa 1 — Reglas Python (80% de los casos, 0 tokens):**
- Lista `ALTA`: depósito de materiales, distribuidora, construrama...
- Lista `MEDIA`: ferretería, almacén...
- Lista `BAJA`: cerrajería, maderas, encofrado...

**Capa 2 — Groq IA (20% de casos ambiguos):**
- Modelo: `llama-3.3-70b-versatile`
- Batches de 5 registros por llamada
- Contexto mínimo: nombre + tipo Google + rating
- Respuesta JSON: vende_cemento, razon, volumen_sacos, observacion

---

## Score de potencial comercial

```
Score 0-100 = Cemento(40) + Madurez(25) + Rating(15) + Telefono(10)
            - Penalizacion_competencia(20)

Prioridad A: score >= 60  →  Atacar ya
Prioridad B: score 45-59  →  Prospectar
Prioridad C: score 25-44  →  Seguimiento
Prioridad D: score < 25   →  Base fría
```

---

## Algoritmo de cruce con base Argos

5 criterios en orden de confianza:
1. **Teléfono** — últimos 8 dígitos exactos (confianza 100%)
2. **Nombre principal** — similitud Jaccard de tokens normalizados
3. **Nombre legal** — razón social sin formas legales
4. **Apellido del dueño** — resuelve personas naturales
5. **Nombre comercial** — columna "Nombre 2" del archivo Argos

Deduplicación: cada negocio de Argos coincide máximo con un registro del censo.

---

## Costo del sistema

| Componente | Costo |
|---|---|
| Google Maps Places API | $0 (dentro de $200 crédito mensual) |
| Groq + Llama 3.3 | $0 (plan gratuito, 30 req/min) |
| Streamlit | $0 (open source) |
| Todas las demás librerías | $0 (open source) |
| **Total** | **$0** |

---

## Licencia

Proyecto académico desarrollado para la Beca Ser ANDI · EAFIT en alianza con Cementos Argos.
