# Datanova Forecast

**Decisiones de inventario más inteligentes para negocios en crecimiento.**

Convierte un historial de ventas en una respuesta a una sola pregunta: *¿qué hago
hoy con mi inventario?* Cargas un CSV o Excel, defines cuatro parámetros, y
obtienes pronóstico de demanda por producto, stock de seguridad, punto de
reorden, cantidad sugerida de pedido y alertas de quiebre y sobreinventario —
cada una con el razonamiento detrás.

**Demo en vivo:** _(pega aquí tu enlace de GitHub Pages cuando lo publiques)_

---

## Publicarlo en GitHub Pages

Son dos archivos y cuatro pasos.

### 1. Crea el repositorio

Entra a <https://github.com/new>:

- **Repository name:** `datanova-forecast`
- **Public** — Pages gratis requiere que el repositorio sea público
- No marques nada más (ni README, ni .gitignore, ni licencia)

**Create repository**.

### 2. Sube los archivos

En el repositorio vacío: **uploading an existing file**.

Arrastra `index.html` y `README.md`. Abajo, **Commit changes**.

### 3. Activa Pages

**Settings** → en el menú lateral izquierdo, **Pages**.

En *Build and deployment*:

- **Source:** Deploy from a branch
- **Branch:** `main` y carpeta `/ (root)`
- **Save**

### 4. Espera un minuto y abre tu enlace

```
https://TU-USUARIO.github.io/datanova-forecast/
```

Recarga la página de Settings → Pages; cuando esté listo aparece el enlace
arriba en un recuadro. La primera publicación tarda uno o dos minutos.

Listo. Ese enlace funciona desde cualquier dispositivo y lo puedes compartir.

---

## Cómo probarlo

En la portada, **Probar con datos de ejemplo**. Genera 50 productos con 24 meses
de historial diario y distintos patrones de demanda — estables, con tendencia,
estacionales, intermitentes, erráticos — más un inventario con quiebres y
sobreinventarios reales. Defines los parámetros, y en unos segundos tienes el
tablero completo.

Con tus propios datos, **Cargar mi archivo**. El archivo necesita tres columnas,
en cualquier orden y con casi cualquier nombre:

| Necesario | Lo reconocemos como |
|---|---|
| Fecha | Date, Fecha, Day, Order Date, Invoice Date, Periodo |
| Código de producto | SKU, Product ID, Código, Item, Material, Article, UPC |
| Cantidad | Units Sold, Qty, Cantidad, Unidades, Ventas, Demand |

Nombre del producto, categoría y precio unitario son opcionales pero mejoran el
resultado — sin precio no hay cifra de ingresos ni análisis ABC por valor.

Entiende `yyyy-mm-dd`, `dd/mm/yyyy`, `mm/dd/yyyy` (lo distingue revisando toda la
columna en busca de un valor mayor a 12), fechas seriales de Excel, decimales con
coma o con punto, y separadores `,` `;` `tab` `|`. Si un encabezado no se
reconoce, revisa los valores: una columna de fechas ISO es una columna de fechas
se llame como se llame.

El archivo de inventario necesita código y stock actual. Lead time, costo,
cantidad en pedido, MOQ y unidades por caja son opcionales.

---

## Tus datos no salen de tu computadora

Todo corre en el navegador: el archivo se lee, se pronostica y se planifica en tu
máquina. No hay servidor, no hay base de datos y no se sube nada a ningún lado.
Para una PYME que desconfía de subir su información de ventas a una nube ajena,
esto no es una limitación — es el argumento de venta.

La consecuencia es que no hay cuentas ni sesiones: al cerrar la pestaña se pierde
el análisis. Para un MVP que muestras a un cliente, eso está bien.

---

## Metodología de pronóstico

### El principio

**No se aplica un solo modelo a todo el catálogo.** Cada producto recibe el
método que demuestra mejor desempeño sobre su propio historial reservado. Un
producto que vende 60 unidades cada día hábil y un repuesto que vende seis veces
al año no son el mismo problema de pronóstico, y tratarlos igual es la falla más
común en herramientas de este rango de precio.

### Paso 1 — Demanda sobre una malla diaria

Las transacciones se expanden sobre un calendario diario continuo. **Un día sin
transacción es demanda = 0, no un dato faltante.** Esto importa más de lo que
parece: si tratas los días en cero como huecos, la media se infla, la varianza se
colapsa y la intermitencia se vuelve invisible — y el motor le aplicaría
Holt-Winters a un producto de baja rotación.

### Paso 2 — Clasificación del patrón de demanda

Clasificación de Syntetos–Boylan–Croston, con los cortes publicados:

- **ADI** — número promedio de periodos entre demandas
- **CV²** — coeficiente de variación al cuadrado de los tamaños de demanda **no cero**

| | CV² < 0.49 | CV² ≥ 0.49 |
|---|---|---|
| **ADI < 1.32** | Estable | Errática |
| **ADI ≥ 1.32** | Intermitente | Irregular |

**La clasificación restringe qué modelos compiten.** La maquinaria de tendencia y
estacionalidad nunca se le ofrece a un producto intermitente o irregular, y los
métodos de la familia Croston nunca se le ofrecen a uno estable. El método
ingenuo queda excluido por completo de la demanda esporádica: "pide lo que se
vendió ayer" en un producto lento alterna entre cero y un lote completo.

### Paso 3 — Los trece métodos

**Para demanda estable y errática:** ingenuo, media histórica, deriva
amortiguada, media móvil, media móvil ponderada, suavizamiento exponencial
simple, Holt con tendencia amortiguada, ingenuo estacional semanal, Holt-Winters
aditivo semanal, y descomposición estacional con índice mensual multiplicativo
(requiere 18+ meses).

**Para demanda intermitente e irregular:** Croston, Croston-SBA con el factor de
corrección de sesgo (1 − α/2), y TSB (Teunter–Syntetos–Babai), que actualiza la
probabilidad de demanda en cada periodo y por eso decae hacia cero en un producto
que entra en obsolescencia — algo que Croston estructuralmente no puede hacer.

Se usa estacionalidad aditiva y no multiplicativa, porque los modelos
multiplicativos se rompen cuando la demanda puede ser cero.

**Por qué no ARIMA ni Prophet:** ambos necesitan más historial del que tiene una
PYME típica y más supervisión de la que puede dar un proceso desatendido. Sobre
12–24 meses de datos diarios con promociones y quiebres adentro, un suavizamiento
exponencial bien calibrado es competitivo y mucho más estable.

### Paso 4 — Backtesting de origen móvil

Tres orígenes walk-forward, no una sola ventana de validación. Cada pliegue
entrena con todo lo anterior al corte y pronostica una ventana del largo del
horizonte. Una sola ventana permitiría que un modelo gane por suerte en un tramo
afortunado del calendario; tres orígenes lo hacen mucho más difícil de fingir.

### Paso 5 — Métricas, y cuál decide

| Métrica | Se reporta | Decide la selección |
|---|---|---|
| **WAPE** (Σ\|e\| / Σ real) | Siempre | Sí — peso 30% |
| **WAPE sobre lead time** | Siempre | **Sí — peso 55%** |
| **MASE** | Siempre | No |
| **RMSE**, **MAE**, **sesgo** | Siempre | El sesgo — penalización hasta 50% |
| **MAPE** | **Solo si ningún valor real es cero** | Nunca |

**MAPE se suprime, no se aproxima.** Con un cero en el denominador es indefinido;
con un valor cercano a cero explota a miles por ciento. Las herramientas que
"manejan" esto sumando un epsilon o descartando los periodos en cero están
reportando un número que no significa nada justo en los productos donde la
precisión más importa. Este motor muestra `—` y explica por qué.

**El WAPE sobre lead time pesa más porque es sobre lo que se decide el
inventario.** No importa si el pronóstico del martes decía 12 o 18 si la semana
cuadró — importa la demanda sobre la ventana de reposición.

**El sesgo se penaliza aparte** porque un modelo sistemáticamente sesgado
sub-abastece o sobre-abastece incluso cuando su error absoluto se ve aceptable.

### Paso 6 — Desempate

Los modelos cuyo score queda dentro del 1.5% del mejor se tratan como **empate
estadístico** y se resuelven por metodología, no por el tercer decimal del WAPE:

- **Demanda esporádica** → se prefiere SBA, luego TSB, luego Croston.
- **Demanda continua** → se prefiere el modelo más simple. Un modelo estacional
  apenas mejor que un suavizamiento exponencial compró un uno por ciento de
  precisión con una docena de parámetros extra, y se degradará más rápido cuando
  el próximo trimestre deje de parecerse al anterior.

Sin esta regla, la elección entre candidatos casi idénticos la decide el ruido y
cambia cada vez que se refrescan los datos.

Vale saberlo porque parece un error y no lo es: **en demanda intermitente
estacionaria, una media histórica plana le gana legítimamente a Croston.** Para
un proceso estacionario i.i.d. la media muestral es el estimador de mínima
varianza; Croston descarta observaciones viejas sin motivo. Croston se gana su
lugar cuando la *tasa de demanda deriva* — obsolescencia o arranque.

### Paso 7 — Intervalos de predicción

La banda diaria es **plana**, en z₈₀ × σ_e, con una tolerancia pequeña solo para
modelos con tendencia. Una banda que se abre con √h sería incorrecta: ese
crecimiento describe el error sobre la demanda **acumulada** a lo largo de h
periodos — que es exactamente donde se aplica, en el stock de seguridad sobre el
lead time. Aplicarlo también a los puntos diarios cuenta dos veces la misma
incertidumbre.

En el gráfico del tablero, donde se agrega entre productos, los intervalos se
combinan **en cuadratura** (√Σ semi-ancho²). Sumarlos de punta a punta asumiría
que todos los productos fallan en la misma dirección el mismo día.

---

## Fórmulas de planificación de inventario

### Stock de seguridad

La fórmula de libro es `SS = z × σ_demanda × √L`. Este motor no la usa como
método principal, porque protege contra lo equivocado.

σ_demanda mide cuánto *varía* la demanda. Lo que realmente causa un quiebre es el
**error del pronóstico** sobre el lead time de reposición. Un producto fuertemente
estacional tiene σ_demanda grande y puede no necesitar casi colchón, porque su
variación es predecible. La distinción es la diferencia entre un colchón
dimensionado para la ignorancia y uno dimensionado para la incertidumbre real.

Tres métodos, elegidos por producto:

**1. Error de pronóstico (predeterminado, demanda continua)**

```
SS = z × σ_e × √L
```

donde σ_e es la raíz del error cuadrático medio de los **residuales
out-of-sample del backtesting**, no del ajuste en muestra.

**2. Error de pronóstico con lead time variable** (cuando σ_L > 0) — fórmula de King:

```
SS = z × √( L × σ_e²  +  d̄² × σ_L² )
```

**3. Cuantil empírico por bootstrap (demanda intermitente e irregular)**

```
SS = Cuantil_NS( demanda de lead time remuestreada ) − E[demanda de lead time]
```

2,000 muestras de bloques de L días consecutivos tomadas del propio historial del
producto. Para demanda esporádica la aproximación normal subestima la cola, así
que un colchón basado en z protege de menos justo en los productos con más
probabilidad de quebrar. La interfaz dice cuál método se usó y por qué.

### Punto de reorden

```
ROP = demanda esperada durante el lead time + stock de seguridad
```

La demanda esperada durante el lead time sale **del pronóstico**, sumando los
primeros L puntos diarios — no de un promedio histórico. En un producto con
tendencia o estacionalidad son números muy distintos.

### Nivel objetivo y pedido sugerido

```
S              = demanda sobre (L + R) + stock de seguridad      R = frecuencia de pedido
Pedido sugerido = max(0, S − posición de inventario)
                  luego se redondea al múltiplo de caja y se eleva al MOQ
```

**Posición de inventario, no stock en mano:**

```
Posición de inventario = en mano + en pedido − pendientes
```

Ordenar contra el stock en mano vuelve a pedir todo lo que ya viene en tránsito.
Es uno de los errores más caros en la práctica, y el motor mantiene las dos
cantidades separadas en todo el flujo.

### Días de cobertura y fecha de quiebre

```
Días de cobertura = stock en mano / demanda diaria PRONOSTICADA promedio
```

Hacia adelante, no una razón histórica. La fecha de quiebre proyectada sale de
una **simulación de agotamiento día por día** contra el pronóstico diario, que es
lo que produce "se quiebra en 4 días" en lugar de un número estático de cobertura.

### Clasificación de riesgo

| Nivel | Condición |
|---|---|
| **Crítico** | Se proyecta quiebre dentro del lead time — ya es tarde para que una orden nueva llegue |
| **Reordenar** | Posición de inventario bajo el punto de reorden, o se proyecta romper el stock de seguridad dentro de L + R |
| **Sobreinventariado** | Días de cobertura por encima del umbral (90 por defecto) |
| **Saludable** | Ninguna de las anteriores |

Nota la definición de crítico. Un quiebre no empieza cuando llegas a cero;
empieza el día en que cruzas el punto de reorden sin ordenar.

### Análisis ABC

Productos ordenados por contribución a los ingresos pronosticados (o al volumen),
cortes acumulados 80% / 95%. La clasificación corre sobre el valor de la demanda
pronosticada y no sobre las ventas pasadas, así que un producto en ascenso se
clasifica por donde va.

### Índice de salud del inventario

0–100, mezcla ponderada de cuatro componentes, cada uno mostrado con los números
que lo sustentan:

| Componente | Peso | Mide |
|---|---|---|
| Riesgo de disponibilidad | 40% | Porción de los **ingresos** pronosticados en productos que requieren reorden — ponderado por ingreso, porque un quiebre en un artículo A no es el mismo evento de negocio que en uno C |
| Inventario excedente | 25% | Porción del valor de inventario por encima del umbral de sobreinventario |
| Confiabilidad del pronóstico | 20% | WAPE sobre lead time ponderado por demanda |
| Cobertura de stock de seguridad | 15% | Porción de productos que mantienen al menos su colchón recomendado |

Nada de números arbitrarios. Cada componente publica su puntaje, su peso y una
frase que explica qué lo movió.

---

## Correcciones de criterio aplicadas

Siete decisiones donde la práctica común está equivocada:

1. **`SS = z × σ × √L` como método principal** → reemplazado por stock de
   seguridad sobre error de pronóstico. El original protege contra variabilidad
   de demanda; los quiebres los causa el error del pronóstico.
2. **MAPE como métrica principal** → suprimido cuando algún valor real es cero,
   que es la mayoría de los productos de un catálogo real.
3. **Stock en mano como base para ordenar** → reemplazado por posición de
   inventario.
4. **Una sola ventana de validación** → backtesting de origen móvil con tres
   pliegues.
5. **La misma métrica decidiendo todos los productos** → score compuesto
   ponderado hacia el error de lead time, con penalización de sesgo explícita, y
   desempatado por metodología para que la elección sea estable.
6. **Días de cobertura desde demanda histórica** → calculados desde demanda
   pronosticada, con simulación de agotamiento diaria.
7. **Modelos estacionales para todo producto con historial suficiente** → el
   patrón de demanda restringe qué modelos compiten.

Y una más: **la precisión del pronóstico y la disponibilidad de inventario se
mantienen separadas**. Un pronóstico perfecto sin stock sigue siendo un quiebre, y
un WAPE de 40% en un artículo C bien amortiguado no es un problema.

---

## Qué está implementado

- Carga de CSV, XLSX, XLS y TSV con detección tolerante de columnas en español e
  inglés, y respaldo por inspección de valores
- Validación con reporte de registros, productos, rango de fechas, meses de
  historial, días sin transacciones, fechas ilegibles, códigos faltantes,
  cantidades negativas, duplicados, historial insuficiente y demanda esporádica
- Opciones de limpieza controladas por el usuario, revalidadas en vivo
- Trece modelos restringidos por patrón, backtesting de tres pliegues, selección
  por producto con desempate metodológico
- Tres métodos de stock de seguridad según el patrón de demanda
- Punto de reorden, nivel objetivo, pedido sugerido con MOQ y múltiplo de caja
- Simulación de agotamiento diaria y fecha de quiebre proyectada
- Cuatro niveles de riesgo, alertas de quiebre y sobreinventario
- Análisis ABC y índice de salud explicado por componentes
- Tablero con triage ordenado por dinero en juego, gráfico y tabla con búsqueda,
  filtros, ordenamiento y exportación a CSV
- Página de detalle por producto con qué / por qué / impacto, metodología,
  métricas y los métodos que compitieron
- Interfaz completa en español e inglés, incluidas las recomendaciones generadas
- Responsive hasta 390px, foco visible por teclado, respeta `prefers-reduced-motion`

## Qué no está incluido

- **Cuentas y persistencia.** No hay login ni base de datos; al cerrar la pestaña
  se pierde el análisis. Es la consecuencia de que todo corra en el navegador.
- **Cobro.** No hay planes ni pagos.
- **Integraciones** con ERP, Shopify, QuickBooks o SAP.
- **Órdenes de compra y proveedores.** Las cantidades se calculan pero no hay a
  dónde enviarlas.
- **Inventario multi-ubicación.** Una posición de stock por producto.
- **Pronóstico causal y promocional.** No hay calendario de promociones ni
  elasticidad de precio. En un catálogo muy promocionado, toma la línea base como
  línea base.
- **Alertas automáticas** por correo o WhatsApp, y corridas programadas.
- **Ajuste manual de pronósticos** por parte del planificador.
- **ML avanzado** — ARIMA, Prophet, LightGBM, reconciliación jerárquica.

---

## Para convertirlo en un SaaS comercial

Lo que separa esto de facturar, en orden:

1. **Cuentas y persistencia.** Es el primer límite real. Requiere un backend:
   Supabase o similar para autenticación y almacenamiento.
2. **Cobro con Stripe.** Checkout, webhook, y límites de productos por plan.
3. **Una app de Shopify.** Historial de ventas e inventario actual, ambos
   automáticos, sin CSV. Elimina el primer paso del onboarding y es donde esta
   categoría consigue distribución.
4. **Corridas programadas con resumen por correo.** "Seis productos requieren
   pedido esta semana" en la bandeja cada lunes es el hábito que vuelve pegajoso
   el producto.
5. **Generación de órdenes de compra.** Las cantidades ya se calculan. Agruparlas
   por proveedor y emitir la orden es el paso de *información* a *hecho*.
6. **Ajuste manual con rastro de auditoría.** Lo primero que hará un planificador
   con experiencia es discrepar de un número. Déjalo ajustarlo, registra quién y
   por qué, y mide la precisión del ajuste contra el modelo.
7. **Pronóstico promocional y causal.** En un catálogo FMCG promocionado es la
   mayor ganancia de precisión disponible.
8. **Seguimiento de desempeño de proveedores.** Medir el lead time real desde la
   orden hasta la recepción y alimentar la media y σ observadas de vuelta al stock
   de seguridad. La fórmula de King ya está implementada esperando un σ_L real.

**El diferenciador no es el pronóstico** — todos dicen tener uno. Es la
**explicabilidad**, y debería ser el argumento comercial. "Ordena 240 unidades
del SKU001, porque la demanda de los próximos 14 días es 180 unidades y caes bajo
el stock de seguridad en 6 días" es una frase que un dueño puede ejecutar y
defender ante su contador. "Nuestra IA predice 240" no lo es.

---

Construido para el ecosistema de consultoría **Datanova**.
