# Vocifera

Este workspace contiene el código fuente de [kokoclone](kokoclone), pero la instalación de desarrollo que se documenta aquí usa los archivos de requisitos que están en la raíz del proyecto.

## Desarrollo

### Estructura relevante

- El código principal está en [kokoclone](kokoclone).
- La instalación base del entorno está en [requirements.txt](requirements.txt).
- La variante CPU de PyTorch está en [requirements-cpu.txt](requirements-cpu.txt).
- La variante GPU de PyTorch está en [requirements-gpu.txt](requirements-gpu.txt).
- Hay un archivo auxiliar para Python 3.11 en [requirements-311.txt](requirements-311.txt).
- El paquete local declara `requires-python = ">=3.12"` en [kokoclone/pyproject.toml](kokoclone/pyproject.toml#L6).

### Resumen rápido

- Python 3.12 es la ruta limpia y alineada con los metadatos actuales del proyecto y de `kanade-tokenizer`.
- Python 3.11 puede funcionar para desarrollo, pero no es una instalación oficial soportada por metadatos: `kanade-tokenizer` declara `Requires-Python >= 3.12` y hay que instalarlo aparte ignorando esa validación.
- CPU y GPU cambian sobre todo en cómo instalas `torch`, `torchaudio` y en si mantienes `kokoro-onnx[gpu]` en la parte base.

### Requisitos previos

- Git
- Python 3.12 o 3.11
- Un entorno virtual por separado
- En GPU NVIDIA: drivers y CUDA compatibles con los wheels de PyTorch que vayas a usar

### 1. Clonar el código de kokoclone

```powershell
git clone https://github.com/Ashish-Patnaik/kokoclone.git .\kokoclone
```

Si ya tienes este workspace preparado, no hace falta repetir este paso.

### 2. Trabajar desde la raíz del proyecto

```powershell
cd .
```

Todos los comandos de instalación de esta guía asumen que estás en la raíz del workspace, porque los `requirements` usados están en la raíz. El código fuente seguirá estando en [kokoclone](kokoclone).

### 3. Crear y activar entorno virtual

#### Python 3.12

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

#### Python 3.11

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

## Matriz de instalación

### Python 3.12 + CPU

Esta es la opción más estable si no necesitas CUDA.

```powershell
pip install -r requirements-cpu.txt
pip install -r requirements.txt
```

Notas:

- Usa [requirements-cpu.txt](requirements-cpu.txt) para PyTorch CPU y [requirements.txt](requirements.txt) para el resto.
- `kanade-tokenizer` se instalará normalmente porque aquí sí se cumple `Requires-Python >= 3.12`.
- El archivo [requirements.txt](requirements.txt) sigue incluyendo `kokoro-onnx[gpu]`. Si quieres una ruta CPU estricta, esa línea debería pasar a `kokoro-onnx` o a un fichero base separado sin el extra `gpu`.
- El archivo [requirements.txt](requirements.txt) también incluye una instalación de `kokoclone` desde Git. Para desarrollo local, eso convive mal con el checkout local y conviene sustituirlo por una instalación editable local si vas a tocar el código.

### Python 3.12 + GPU

Esta es la ruta recomendada si vas a desarrollar con NVIDIA.

```powershell
pip install -r requirements-gpu.txt
pip install -r requirements.txt
```

Notas:

- [requirements-gpu.txt](requirements-gpu.txt) fija el índice CUDA de PyTorch.
- [requirements.txt](requirements.txt) instala el resto de dependencias y ya viene alineado con `kokoro-onnx[gpu]`.
- Igual que en CPU, [requirements.txt](requirements.txt) incluye `kokoclone` desde Git; si vas a desarrollar sobre [kokoclone](kokoclone), es preferible usar el checkout local.

### Python 3.11 + CPU

Esto es un workaround de desarrollo. `pip` falla si intenta resolver `kanade-tokenizer` de forma normal porque ese paquete declara `Requires-Python >= 3.12`.

Instala primero la base sin Kanade, y después Kanade ignorando solo esa validación en un comando separado:

```powershell
pip install -r requirements-cpu.txt
pip install -r requirements-311.txt
pip install --ignore-requires-python git+https://github.com/frothywater/kanade-tokenizer
```

Notas:

- [requirements-311.txt](requirements-311.txt) existe para evitar que el resolvedor se bloquee durante la instalación base.
- No se puede poner `--ignore-requires-python` solo para una línea dentro de un `requirements.txt`; `pip` lo trata como una opción global de ejecución, no por paquete.
- Este camino sirve para desarrollar y probar desde el checkout local, pero no cambia el hecho de que los metadatos actuales del proyecto siguen marcando Python 3.12 como mínimo.
- Igual que en Python 3.12, la ruta CPU sigue arrastrando la salvedad de `kokoro-onnx[gpu]` si usas [requirements.txt](requirements.txt) tal cual.

### Python 3.11 + GPU

Mismo enfoque que en CPU, pero instalando PyTorch CUDA antes:

```powershell
pip install -r requirements-gpu.txt
pip install -r requirements-311.txt
pip install --ignore-requires-python git+https://github.com/frothywater/kanade-tokenizer
```

Notas:

- La parte delicada no es CUDA, sino `kanade-tokenizer`.
- Si algo falla aquí, normalmente el primer sospechoso no será Python 3.11 sino la combinación exacta de `torch`, `torchaudio`, drivers y CUDA.

## Sobre Python 3.11 y `kanade-tokenizer`

Lo importante de esta conversación es esto:

- El bloqueo viene del metadato `Requires-Python`, no necesariamente de una incompatibilidad real del código.
- El upstream de `kanade-tokenizer` declara `>=3.12`.
- El proyecto local también declara `>=3.12` en [kokoclone/pyproject.toml](kokoclone/pyproject.toml#L6).
- En un `requirements.txt`, `pip` no soporta aplicar `--ignore-requires-python` a una única dependencia.

Si quieres una solución más limpia para 3.11, las dos alternativas razonables son:

1. Mantener el flujo en dos pasos: instalar base y luego `kanade-tokenizer` con `--ignore-requires-python`.
2. Hacer un fork de `kanade-tokenizer`, bajar su `requires-python` a `>=3.11` y depender de ese fork.

Con los archivos actuales en [kokoclone](kokoclone), no hace falta instalar `kokoclone` desde Git para desarrollo local: basta con trabajar desde el checkout y ejecutar los scripts del repositorio.

## Instalación editable del proyecto

Si además quieres trabajar como paquete editable, ahora mismo hay que tener en cuenta que [kokoclone/pyproject.toml](kokoclone/pyproject.toml#L6) sigue fijando `>=3.12`.

### Editable en Python 3.12

```powershell
pip install -e .
```

### Editable en Python 3.11

No es una ruta limpia mientras el `pyproject.toml` local siga declarando `>=3.12`.

Opciones:

1. Desarrollar sin `pip install -e .`, ejecutando directamente `app.py`, `cli.py` o imports locales desde la carpeta del repo.
2. Cambiar localmente [kokoclone/pyproject.toml](kokoclone/pyproject.toml#L6) a `>=3.11` si quieres que el empaquetado editable también acepte 3.11.

En este workspace, si quieres desarrollar sobre el código clonado en [kokoclone](kokoclone), la variante más coherente es usar una instalación editable local en lugar de depender de la línea `git+https://github.com/Ashish-Patnaik/kokoclone.git` que hoy aparece en [requirements.txt](requirements.txt).

## Comprobación rápida

Tras instalar, puedes validar el entorno así:

```powershell
python -c "import torch, torchaudio, soundfile, gradio; print('base ok')"
python -c "import kanade_tokenizer; print('kanade ok')"
```

Y para levantar la app:

```powershell
python .\kokoclone\app.py
```

## Flash Attention en Windows

En Windows, para instalar Flash Attention, usa una wheel precompilada desde las releases de:

- https://github.com/mjun0812/flash-attention-prebuild-wheels/releases

La instalación se hace con `pip install` apuntando al enlace directo del `.whl` publicado en esa release.

Ejemplo:

```powershell
pip install https://github.com/mjun0812/flash-attention-prebuild-wheels/releases/download/v0.9.39/flash_attn_3-3.0.0+cu126torch2.9gite2743ab-cp39-abi3-manylinux_2_34_aarch64.whl
```

Importante:

- Elige la wheel que corresponda exactamente a tu plataforma (Windows), versión de Python, versión de PyTorch y versión de CUDA.
- El ejemplo anterior es solo de referencia del formato del comando.

## Recomendación práctica

- Si quieres el camino con menos fricción: usa Python 3.12.
- Si necesitas quedarte en Python 3.11 para desarrollo: usa [requirements-311.txt](requirements-311.txt) y luego instala `kanade-tokenizer` aparte con `--ignore-requires-python`.
