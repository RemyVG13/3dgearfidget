# Generador de esferas de engranajes con recorte por STL

Aplicacion web 100% navegador para generar mecanismos fidget de engranajes
interconectados con forma esferica y recortarlos con un volumen STL.
Toda la geometria (incluidas las operaciones booleanas) se calcula en el cliente
con **Three.js** (visualizacion) y **Manifold3D** (motor CSG, en un Web Worker).

El diseno tecnico completo esta en [docs/DISENO_TECNICO.md](docs/DISENO_TECNICO.md).

## Puesta en marcha

```sh
npm install
npm run dev      # servidor de desarrollo (Vite)
npm run build    # build de produccion (estatico, desplegable en cualquier hosting)
npm run preview  # servir el build
```

Validacion headless del pipeline geometrico + booleanas (sin UI):

```sh
npx tsx scripts/validate.mts
```

## Guia rapida de uso

La pantalla tiene tres zonas: **panel izquierdo** (parametros del mecanismo),
**visor 3D central** y **panel derecho** (recorte por STL y exportacion).

### 1. Genera la esfera de engranajes (panel izquierdo)

1. Pulsa un preset ("Octaedro 8 x 12T/15T/18T") para empezar con una
   configuracion valida, o ajusta los parametros a mano.
2. **Diametro de la esfera** es el tamano real del conjunto: los engranajes se
   calculan para llenar esa esfera y el nucleo queda dentro. El modulo resultante
   se muestra en "Validacion mecanica" (no se edita aparte).
3. Los parametros no reconstruyen el solido solos. Cuando cambies algo, pulsa
   **Refrescar** (arriba a la izquierda del visor). Veras 8 engranajes conicos
   con la base apoyada en el nucleo. Cada eje une el nucleo con la mitad interior
   del engranaje y no asoma por fuera. **Rayos X** deja solidos el nucleo y los ejes.
4. Revisa "Validacion mecanica": el badge **Mecanismo valido / No funcional**
   analiza el grafo de engrane. En el octaedro el numero de **dientes debe ser
   multiplo de 3** (si no, el badge avisa de que no cerrara).
5. "Simulacion de movimiento": pulsa **Reproducir** para ver girar el conjunto.
   Los dientes de un color entran en los huecos del otro.

### 2. Recorta con una forma o un STL (panel derecho)

1. En "Forma de recorte" elige **Esfera**, **Cubo** o **STL** (importa tu
   archivo). Aparecera un volumen morado semitransparente **junto** a los
   engranajes (el mecanismo no desaparece).
   - Si importas un STL, se valida automaticamente: veras un badge (apto /
     reparado / no apto), el numero de triangulos y si la malla esta cerrada.
2. Coloca el volumen sobre la zona que quieres conservar: usa el **gizmo**
   (Mover / Rotar / Escalar) directamente en el visor, o los **campos
   numericos** (traslacion, rotacion, escala por eje o proporcional).
3. Pulsa **Previsualizar recorte**. Se calcula `MECANISMO ∩ volumen STL` pieza a
   pieza (el original se conserva, atenuado detras). Veras:
   - un badge **Comprobaciones superadas** o **Recorte NO funcional**, y
   - una lista de **advertencias mecanicas** (engranajes eliminados, piezas
     partidas en fragmentos, contactos de engrane perdidos, etc.).
   > No se declara "funcional" si no pasa todas las comprobaciones.
4. Con "Visibilidad de capas" puedes mostrar/ocultar por separado el mecanismo
   original, el volumen STL y el resultado del recorte para compararlos.
5. Cuando te convenza, pulsa **Confirmar recorte** (o **Descartar** para volver
   al original).

### 3. Exporta (panel derecho, abajo)

La fuente de exportacion es el recorte si hay uno activo; si no, el mecanismo
original. Todas las piezas se mantienen independientes:

- **Descargar ZIP**: un STL por pieza.
- **Descargar ensamblaje**: un unico STL con todo el conjunto.
- **Descargar 3MF**: multi-objeto, preserva piezas separadas.
- O descarga cada pieza por separado desde "Piezas individuales".

> Rendimiento: mientras arrastras el gizmo no se recalcula la booleana (solo se
> mueve la silueta); el recorte real se recalcula al soltar o al pulsar los
> botones. Pensado para STL de cientos de miles de caras.

## Limitaciones conocidas

- Engranajes biselados por aproximacion de Tredgold (no involuta esferica exacta).
- Motor de malla triangular (Manifold): sin B-rep ni exportacion STEP en v1.
- El STL de recorte debe ser un solido cerrado (manifold); se intenta una
  reparacion best-effort al cargarlo y se avisa si no es valido.
- El numero de engranajes se elige entre familias validadas, no es un entero
  arbitrario (ver documento tecnico, seccion 5).
