# Guía: Del análisis heurístico al modelo ML real

Este documento es la hoja de ruta para evolucionar `CaféPVS Visión` desde su versión actual (análisis de color + checklist de síntomas) hacia un modelo de machine learning entrenado específicamente para hojas de café dominicano.

## Arquitectura — qué cambia y qué no

La aplicación actual ya está diseñada para conectar un modelo. El flujo es:

```
Imagen → analyzeImage() → { profile, mlPredictions } → rankConditions() → diagnóstico
                                ↑
                                └── aquí entra el modelo ML cuando exista
```

El motor heurístico (histograma HSV) NO se elimina cuando llegue el modelo. Se mantiene como segundo motor que valida y complementa al ML. La precisión final es la combinación: red neuronal + análisis de color + síntomas confirmados por el agrónomo.

## Tres caminos para conseguir un modelo, ordenados de menor a mayor inversión

### Camino 1 — Teachable Machine (1-2 semanas, sin código)

Google Teachable Machine entrena modelos de clasificación directamente desde el navegador y exporta a TensorFlow.js. Cero programación.

**Procedimiento:**

1. Cada agrónomo (Luciandra, Yeuris, Jose Anibal, Oliver) toma fotos durante visitas regulares, etiquetando cada una manualmente:
   - "Roya leve", "Roya severa"
   - "Antracnosis"
   - "Mancha de hierro"
   - "Deficiencia N visible", "Deficiencia K bordes", "Deficiencia B"
   - "Hoja sana"
2. Mínimo 50 fotos por categoría. Idealmente 100+. Variedad de luz, ángulo, distancia.
3. Subir a teachablemachine.withgoogle.com → Image Project → Standard image model.
4. Entrenar (5-10 minutos en el navegador).
5. Exportar como TensorFlow.js → bajar carpeta `model/` con `model.json` y archivos `.bin`.
6. Servir esos archivos junto al PWA (en `/models/coffee-detector/`).
7. Cargar en `vision-detector.js` con `tf.loadLayersModel('/models/coffee-detector/model.json')`.

**Ventaja:** Funciona sin saber ML. Resultados decentes con dataset pequeño.
**Limitación:** Modelo simple, menos preciso que transfer learning. Categorías limitadas a las que hayas etiquetado.

### Camino 2 — Transfer learning con MobileNet en Google Colab (2-4 semanas)

Más preciso. Requiere conocimiento básico de Python y notebooks. Carlos Fonseca puede aprender esto, o se puede contratar un ingeniero de ML por proyecto.

**Procedimiento:**

1. Curar dataset propio (idealmente 500-1000 imágenes por categoría):
   - Fotos de campo etiquetadas por los agrónomos
   - Augmentar con datasets públicos relacionados:
     - **PlantVillage** (Penn State, Open Access) — incluye hojas de café con roya, ~1700 imágenes
     - **BRACOL** (Brazilian Coffee Leaf Dataset) — Universidad Federal de Espírito Santo, varias enfermedades
     - **JMuBEN** (Kenya) — ~58k imágenes de café arábica con anotaciones
     - **RoCoLe** (Robusta Coffee Leaf Dataset) — Ecuador, varias condiciones
2. Subir dataset a Google Drive o Kaggle.
3. En Google Colab (gratis), usar transfer learning con MobileNetV2 o EfficientNet-Lite0 como backbone.
4. Entrenar el clasificador final con tu dataset (30-60 min de GPU gratis en Colab).
5. Validar con conjunto de prueba (separado, fotos que NUNCA vio el modelo).
6. Convertir a TensorFlow.js: `tensorflowjs_converter --input_format=keras model.h5 ./tfjs_model/`
7. Integrar en el PWA igual que en Camino 1.

**Ventaja:** Precisión sustancialmente mayor. Modelo se entrena rápido en Colab gratis.
**Limitación:** Requiere alguien con manejo básico de Python/Colab.

### Camino 3 — Modelo profesional con servicio externo (variable)

Para cuando el programa tenga presupuesto y busque máxima precisión.

- Contratar a una consultoría de visión computacional (1-3 meses)
- O usar servicios cloud como Google Cloud AutoML Vision, Azure Custom Vision, AWS Rekognition Custom Labels
- Estos servicios entregan APIs (no modelos descargables) — para uso offline conviene insistir en TensorFlow.js export

**Costos típicos:** $5,000 - $30,000 USD según alcance.

## Recomendación pragmática para Induban

**Trimestre 1 (Mayo-Julio 2026):** Camino 1 — Teachable Machine. Cada agrónomo dedica 5 minutos por visita a etiquetar 1-2 fotos. Llegan a 300-500 fotos en 3 meses. Primer modelo funcional.

**Trimestre 2-3 (Agosto 2026 en adelante):** Mientras el dataset crece, considerar Camino 2 si Carlos Fonseca o un asesor externo puede tomar el rol de fine-tuning. El dataset de 500+ fotos ya permite transfer learning serio.

**Trimestre 4+ (2027):** Si los resultados justifican la inversión, profesionalizar con servicio externo o ingeniero dedicado.

## Datasets públicos — links y notas

| Dataset | Origen | Tamaño | Categorías | Acceso |
|---------|--------|--------|------------|--------|
| PlantVillage | Penn State | ~54k (todas plantas) | 38 (incluye café-roya) | github.com/spMohanty/PlantVillage-Dataset |
| BRACOL | UFES Brasil | ~1700 | Roya, mancha de hierro, cercospora, miner | data.mendeley.com (buscar "BRACOL") |
| JMuBEN | Kenia | ~58k | Roya, miner, cercospora, anthracnose, sano | data.mendeley.com (buscar "JMuBEN") |
| RoCoLe | Ecuador | ~1500 | Robusta — roya, ácaros, sano | data.mendeley.com (buscar "RoCoLe") |

**Importante:** Estos datasets son útiles para arrancar, pero la luz, suelo, altitud y variedades dominicanas (Catucaí, Acauã, Arara, Tupí, RA-15) no están representadas. Un modelo entrenado solo con datasets brasileños o kenianos cometerá más errores en Monte Bonito que uno que incluya 200 fotos locales.

## Cómo capturar fotos para entrenamiento

Esto es lo que cada agrónomo debería hacer durante visitas regulares:

1. **Una hoja, fondo neutro:** apoyar la hoja sobre papel blanco o un fondo de tela. Evita fotos con muchas hojas y planta entera (al menos por ahora).
2. **Luz natural difusa:** sombra abierta, no sol directo. Idealmente 9-11 AM o 3-5 PM.
3. **Distancia 20-30 cm:** la hoja debe ocupar al menos 60% del cuadro.
4. **Foco bien:** tocar el celular sobre la hoja antes de disparar.
5. **Un síntoma claro por foto:** si la hoja tiene roya Y manchas de cercospora, mejor dos fotos separadas o etiquetar con la condición dominante.
6. **Etiquetar al instante:** antes de que pase otro caso. Una hoja en Excel o un campo libre en Repsly/Salesforce.

Estructura sugerida del nombre de archivo:

```
zona_productor_condicion_severidad_fecha.jpg
azua_jose-melo_roya_alta_2026-05-15.jpg
ocoa_mahoma_alex-medina_sano_normal_2026-05-15.jpg
```

## Integración del modelo entrenado

Una vez tienes el modelo TF.js exportado:

```javascript
// En vision-detector.js, descomentar e implementar initModel():
initModel: async function() {
  try {
    this.state.mlModel = await tf.loadLayersModel('/models/coffee-detector/model.json');
    console.log('[VisionDetector] Modelo ML cargado');
  } catch (e) {
    console.warn('[VisionDetector] Sin modelo ML — usando solo análisis heurístico', e);
  }
},

// Y completar runMLInference():
runMLInference: async function(imageElement) {
  const tensor = tf.browser.fromPixels(imageElement)
    .resizeBilinear([224, 224])
    .expandDims(0)
    .div(255.0);
  const predictions = await this.state.mlModel.predict(tensor).data();
  tensor.dispose();
  // Map a las claves de COFFEE_CONDITIONS
  return predictions; // array con probabilidades por clase
}
```

Y agregar TF.js al index.html:

```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.10.0/dist/tf.min.js"></script>
```

(O servir tfjs localmente para uso 100% offline.)

## Métricas a vigilar

- **Accuracy global:** % de aciertos en conjunto de validación. Empezar con >70% es realista.
- **Confusion matrix:** ¿qué confunde con qué? Si confunde roya con mancha de hierro, OK; si confunde antracnosis con hoja sana, problema serio.
- **Precision por clase:** especialmente para enfermedades de alto impacto (roya, antracnosis, broca). Falsos positivos son menos malos que falsos negativos.
- **Recall por clase:** porcentaje de casos reales que detecta. Para deficiencias, recall importa más que precision.
- **Tiempo de inferencia en celular:** debe ser <2 segundos. MobileNet logra esto, EfficientNet-B3+ no.

## Una nota honesta sobre expectativas

Un modelo bien entrenado no reemplaza al agrónomo. Lo asiste. Yeuris en La Peonia siempre tendrá mejor juicio que la app sobre si una mancha es roya o ojo de gallo, porque puede tocar la hoja, ver el envés, oler, evaluar el contexto del lote. La app sirve para:

1. Acelerar el diagnóstico de casos típicos
2. Ayudar al agrónomo nuevo o en formación
3. Estandarizar criterios entre los 4 agrónomos del equipo
4. Documentar el caso con foto + diagnóstico en un solo paso
5. Generar recomendación + recibo automáticamente

No para sustituir su criterio. Esto importa comunicarlo al equipo: la app es una herramienta más, no el reemplazo del experto.
