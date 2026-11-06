# Tangicortex

**Una interfaz tangible para recorrer un cerebro humano real.** Apoyas una sonda sobre un
cráneo de plástico y la pantalla muestra el **corte del cerebro que hay justo debajo de la
punta**: tejido real escaneado con HiP-CT, a escala de sus estructuras, con 95 regiones
anatómicas con nombre en español.

El cráneo y la sonda son objetos físicos que se pueden tomar y mover: la mano sabe dónde está
porque **toca** el cráneo, y la imagen sigue ese gesto. El sistema no le devuelve al usuario
fuerza ni vibración; el tacto lo pone el cráneo de plástico (háptica pasiva). Es un
instrumento para enseñar neuroanatomía con un volumen histológico real en vez de un modelo
dibujado.

---

## Antecedente

La idea de manejar un volumen cerebral con objetos físicos viene de
**Hinckley, Pausch, Goble y Kassell (1994)**, *Passive real-world interface props for
neurosurgical visualization*, CHI '94, pp. 452–458
([DOI 10.1145/191666.191821](https://doi.org/10.1145/191666.191821),
[PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/Props-for-Neurosurgical-Visualization-CHI-1994.pdf)).
Para planificar cirugías usaron una cabeza de muñeca y una placa con sensores de posición y
orientación, que llamaron *props*: girar la cabeza giraba la resonancia, y acercar la placa
definía el plano de corte. Los neurocirujanos los usaban sin entrenamiento, porque el gesto
era el mismo que harían con un objeto real.

Tangicortex retoma esa idea con hardware de bajo costo y la lleva a la enseñanza:

| | Hinckley et al. 1994 | Tangicortex |
|---|---|---|
| sensor | rastreador electromagnético de 6 grados de libertad | IMU de bajo costo (MPU9250) en una ESP32-C3 |
| posición | medida por el rastreador | inferida: la punta siempre está sobre el cráneo |
| corrección de deriva | no hace falta | imanes de neodimio en puntos craneométricos |
| volumen | resonancia del paciente | cerebro completo HiP-CT (Human Organ Atlas), 95 regiones |
| usuario | neurocirujano | estudiantes y público |

## Qué hace

- **Corte bajo la punta.** La orientación de la sonda define un corte oblicuo del volumen,
  centrado en el punto del cráneo donde está apoyada.
- **Profundidad con gestos.** Girar la sonda como un destornillador hunde o sube el corte
  (1 vuelta = 3 cm); un golpecito baja 5 mm, un doble golpecito sube 5 mm.
- **Imanes como faros.** Imanes de neodimio en frente, Cz, temporales y occipital corrigen la
  deriva del giroscopio cada vez que la sonda pasa quieta por uno; el de la frente devuelve la
  vista a la superficie. El magnetómetro solo detecta imanes y nunca entra en el cálculo de la
  orientación.
- **Regiones con nombre.** 95 estructuras (segmentación FastSurfer) con colores y nombre bajo
  la mira o bajo el mouse, p. ej. *«Giro precentral: movimiento (izquierdo)»*.
- **Vistas 3D** del cerebro por lóbulos, con el plano de corte y la sonda, en pantallas anchas.
- **Calibración guiada** por voz y dibujos. No supone nada sobre cómo va montado el sensor en
  la sonda: aprende los ejes, la polaridad de cada imán y la dirección de los golpes.
- **Funciona sin sonda**: con el mouse se recorre la cabeza y con la rueda se cambia la
  profundidad.

## Hardware

| pieza | detalle |
|---|---|
| microcontrolador | ESP32-C3 Super Mini (USB CDC) |
| IMU | MPU9250 (acelerómetro, giroscopio, magnetómetro) en la punta de la sonda |
| cráneo | cráneo anatómico de plástico |
| imanes | neodimio en frente, Cz (polaridad invertida), temporal izq., temporal der., occipital |
| PC | Linux x86-64 con escritorio (X11 o Wayland) |

El firmware está en [`firmware/hoa_imu/`](firmware/hoa_imu/) (Arduino). `firmware/flashear.sh`
compila y sube con arduino-cli; `firmware/flashear_lapiz.sh` graba un binario ya compilado con
esptool.

## Instalación

Con [uv](https://docs.astral.sh/uv/) (Python 3.11–3.13):

```bash
git clone https://github.com/Debaq/Tangicortex.git
cd Tangicortex
uv run tangicortex.py --paquetes   # la primera vez: elegir y descargar el cerebro
```

Con pip, `pip install .` deja el comando `tangicortex`.

**Puerto serie:** el usuario debe estar en el grupo `dialout` (Debian/Ubuntu) o `uucp` (Arch).
Sin él el programa funciona igual, pero sin sonda.

### Datos

Los cerebros **no están en el repositorio**. La primera vez se abre la pantalla de paquetes,
que los descarga desde la fuente oficial (CC BY 4.0):

| paquete | contenido | descarga | en disco |
|---|---|---|---|
| `ladaf17-regiones` (recomendado) | imagen 247 µm + 95 regiones | ~330 MB | 275 MB |
| `ladaf17-regiones-liviano` | imagen 494 µm + 95 regiones | ~45 MB | 83 MB |

En desarrollo se guardan en `datos/`; el binario usa `~/.local/share/tangicortex/`. Si existe
la carpeta del nombre anterior (`hoa-imu-demo`), la reutiliza con su calibración y sus paquetes.

## Uso

```bash
uv run tangicortex.py                         # sonda en /dev/ttyACM*, se reconecta sola
uv run tangicortex.py --completa --monitor 1  # pantalla completa en otro monitor
uv run tangicortex.py --sin-imu               # solo mouse y teclado
uv run tangicortex.py --instalar ladaf17-regiones   # preparar un equipo sin ventana
```

| tecla | acción |
|---|---|
| `p` | calibrar la sonda (guía en pantalla) |
| `m` | configurar el montaje del sensor dentro de la sonda |
| `espacio` | armar la vuelta a la superficie (se completa en el imán de la frente) |
| `e` | mostrar u ocultar regiones |
| `b` | ventana de imanes (distancia y polaridad) |
| `F2` | pantalla de paquetes |
| `Esc` / `q` | salir |

Binario autónomo para Linux: `./construir_binario.sh` → `dist/tangicortex-linux.tar.gz`.

## Documentación

- [GUIA-VISITANTES.md](GUIA-VISITANTES.md): texto para el público (qué se ve, desafíos, datos curiosos).
- [LEEME-TECNICO.md](LEEME-TECNICO.md): montaje, calibración, gestos, imanes y mantenimiento.

## Estructura

| archivo | qué hace |
|---|---|
| `tangicortex.py` | programa principal: cortes, regiones, sonda, gestos, pantalla |
| `paquetes.py` | catálogo de cerebros, descarga, conversión, nombres de regiones en español |
| `instalador.py` | pantalla de paquetes |
| `vista3d.py` | mini vista 3D (lóbulos, plano de corte y sonda) |
| `hoa_bajar.py` | descarga directa de volúmenes del Human Organ Atlas |
| `grabar_guiado.py`, `analizar_*.py` | grabaciones guiadas de la sonda y su análisis |
| `firmware/` | firmware Arduino de la sonda y scripts para grabarlo |

## Créditos y datos

**El cerebro y sus regiones no son nuestros.** Son datos abiertos con licencia CC BY 4.0:

- **Imágenes:** Human Organ Atlas, cerebro completo del donante LADAF-2021-17, escaneado en el
  ESRF (línea BM18) con HiP-CT. DOI [10.15151/ESRF-DC-2268032801](https://doi.org/10.15151/ESRF-DC-2268032801).
- **Regiones:** segmentación FastSurfer de Chourrout et al., *The Extremely Brilliant Brain: An
  Isotropic Microscale Human Brain Dataset*
  ([artículo](https://pmc.ncbi.nlm.nih.gov/articles/PMC12874060/),
  [repositorio](https://github.com/UCL-MXI-Bio/2025-chourrout-extremely-brilliant-brain)).

Agradecemos a la persona donante y a su familia, y a los equipos del ESRF, del University
College London y del Laboratorio de Anatomía de los Alpes Franceses (LADAF).

**Autores:** Nicolás Baier-Quezada, Vanessa Uribe-Hernández, Fernanda López-Moncada y
Benjamín Catipillán-Ulloa — TecMedHub · Universidad Austral de Chile, Sede Puerto Montt.

## Licencia

Código: por definir. Datos: CC BY 4.0 (Human Organ Atlas); se descargan aparte y no se
redistribuyen aquí.
