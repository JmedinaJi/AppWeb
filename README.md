# CaféPVS — App de Visita Técnica v0.2

App integrada para visitas técnicas de Induban Café Creciente. Combina tres herramientas en un solo flujo de campo:

1. **Datos de la visita** — productor, finca, zona, agrónomo, GPS automático, fecha
2. **Análisis de sombra** — captura del dosel hacia arriba, mide % de cobertura con umbral Otsu
3. **Detector visual** — identifica enfermedades, plagas y deficiencias con análisis de color HSV + checklist de síntomas
4. **Recibo PDF** — compila todo lo capturado, observaciones, firma del productor → PDF profesional descargable

## Archivo principal

**`cafepvs-app.html`** — Una sola página, ~92 KB. Funciona offline después de la primera carga (excepto el GPS la primera vez y el CDN de jsPDF que se cachea).

## Cómo probar

### Opción rápida (5 minutos): GitHub Pages

1. Crea un repositorio en GitHub: `cafepvs-test`
2. Sube `cafepvs-app.html` y renómbralo a `index.html`
3. En Settings → Pages → Source: deploy from branch `main`, root
4. En 1-2 minutos tu URL es `https://[tu-usuario].github.io/cafepvs-test/`
5. Abre esa URL en el celular del agrónomo. Acepta permisos de cámara y GPS.

### Opción local (para probar sin internet)

1. Crea una carpeta `cafepvs/` con `cafepvs-app.html` dentro
2. Sirve con cualquier servidor estático local. Python lo hace gratis:
   ```
   cd cafepvs/
   python3 -m http.server 8000
   ```
3. En tu celular conectado al mismo WiFi, abre `http://[IP-de-tu-laptop]:8000/cafepvs-app.html`

**Importante**: la cámara solo funciona por HTTPS o desde `localhost`. Si la abres directamente como archivo (`file://`), el botón "Cámara" fallará pero "Subir desde galería" sí funciona.

## Flujo de uso en campo

```
1. Llegar al lote
2. Abrir la app
3. Llenar datos del productor (toma 30 segundos)
4. [GPS se captura solo]
5. Tocar "Análisis de sombra"
   → Apuntar al dosel
   → Capturar
   → Analizar
   → "Guardar en la visita"
6. Tocar "Detector visual"
   → Acercar a una hoja con síntomas
   → Capturar
   → Analizar
   → Confirmar síntomas con checklist
   → "Guardar en la visita"
7. "Generar recibo de visita"
   → Agregar observaciones libres
   → Pedir firma al productor en pantalla
   → "Descargar PDF"
8. Compartir el PDF al productor por WhatsApp
```

Tiempo total: 4-6 minutos de visita técnica documentada profesionalmente.

## Datos persistentes

**Esta versión es de sesión**: si recargas la app pierdes la visita en curso. Es intencional para esta primera ronda de pruebas — quieres testear el flujo, no debugear sincronización.

Cuando estés listo de pasar a producción, hay que decidir:
- ¿Persistencia local con IndexedDB? (visitas guardadas en el celular, sin internet)
- ¿Sync a backend (Repsly, Salesforce, propio)? (visitas viajan al servidor)
- ¿Ambos? (offline-first con sync diferido)

Eso lo decidimos cuando definas el camino del CRM.

## Lo que falta (consciente, no descuido)

Esta es la v0.2 para validar el flujo. Las decisiones que dejé fuera a propósito:

- **Persistencia entre sesiones** — fácil de agregar después con IndexedDB
- **Lista de productores precargada** — hoy se escribe a mano; viene del Excel maestro o del CRM elegido
- **Modelo ML real** — la arquitectura está lista para conectarlo (ver `MODEL_TRAINING_GUIDE.md`); por ahora es análisis de color + checklist
- **Múltiples hojas / múltiples puntos de muestreo en una visita** — hoy 1 captura por módulo; ampliar cuando el flujo base esté validado
- **Calculadora de fórmula de fertilización** — la herramienta #2 de la lista original; siguiente módulo si esto camina
- **Calendario BPA contextual** — la herramienta #4; siguiente

## Qué validar con el equipo

Sugerencias de qué pedir feedback al probar con Carlos Fonseca, Yeuris, Luciandra:

1. **Flujo**: ¿el orden tiene sentido? ¿falta algo obvio?
2. **Datos del productor**: ¿qué campos faltan o sobran?
3. **Sombra**: ¿el % de cobertura coincide con su estimación visual?
4. **Detector**: ¿qué condiciones identifica bien? ¿cuáles falla?
5. **Recibo PDF**: ¿qué falta en el documento? ¿el productor lo acepta como entregable?
6. **Velocidad**: ¿es rápido suficiente para no entorpecer la visita?
7. **Robustez**: ¿qué pasa con conexión irregular, batería baja, sol directo?

## Otros archivos en este paquete

- `cafepvs-vision.html` — Versión anterior (solo detector visual). Se puede eliminar.
- `vision-detector.js` — Módulo modular para integrar al PWA antiguo. Vigente si decides mantener el PWA original con sus módulos separados en lugar de migrar todo a esta app integrada.
- `coffee-diseases-db.js` — Base de datos de condiciones (versión expandida con más detalles). Útil si en el futuro separas la base de datos del HTML.
- `MODEL_TRAINING_GUIDE.md` — Plan para entrenar modelo ML real cuando tengas dataset.
- `INTEGRATION.md` — Vigente si vas por la ruta de integrar al PWA antiguo.

## Próximos pasos sugeridos

1. **Esta semana**: Subir a GitHub Pages, probarla tú mismo con 2-3 fotos reales. Validar que el flujo cierra.
2. **Próximas 2 semanas**: Darle a Yeuris para uso real en Jamamuncito. Pedir feedback estructurado.
3. **Mes 2**: Iterar UI con feedback, agregar persistencia local, agregar el módulo de fórmula de fertilización (es el que más tiempo ahorra a Carlos).
4. **Mes 3**: Empezar a recolectar fotos etiquetadas para el modelo ML real (objetivo: 200 fotos por categoría en 3 meses).
