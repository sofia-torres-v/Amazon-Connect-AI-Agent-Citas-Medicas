
# Citas Médicas - AI Agent

Asistente agéntico de citas médicas con Amazon Connect y Amazon Q in Connect. Utiliza prompts y tools para razonar, decidir acciones y ejecutar operaciones mediante AWS Lambda y DynamoDB.

> Proyecto presentado en **ComuniAWS Day 2026** como parte de la charla «De intents a prompts: construyendo agentes con Amazon Connect».

**Idea central:** la transición de bots basados en intents y slots hacia AI Agents y tools para la gestión de citas médicas.

## Arquitectura del sistema

El proyecto implementa un asistente conversacional con servicios serverless de AWS:

- **Amazon Connect** para la experiencia conversacional.
- **Amazon Lex V2** como puente hacia el AI Agent.
- **Amazon Q in Connect** para el razonamiento del agente.
- **Flow Module** como herramienta (Tool) del agente.
- **AWS Lambda** para ejecutar las operaciones.
- **DynamoDB** como base de datos.

> El AI Agent decide qué necesita hacer y cuándo utilizar la herramienta. El backend ejecuta las reglas de negocio y devuelve el resultado al agente.

<div align="left">
<img src="docs/assets/arquitectura.png" alt="Arquitectura Citas Médicas AI Agent" width="100%" />
</div>

### Diagrama

```text
Usuario
   │
   ▼
Chat (Amazon Connect)
   │
   ▼
Bot Lex "Puente"
(sin intents, solo reenvía)
   │
   ▼
AI Agent (Amazon Q in Connect)
   │  Razonamiento + decisión
   ▼
Tool (Flow Module)
   │
   ▼
AWS Lambda
   │
   ▼
Repository Pattern
   │
   ▼
DynamoDB
```

> El AI Agent decide qué necesita hacer y cuándo utilizar la herramienta. El backend ejecuta las reglas de negocio y devuelve el resultado al agente.

## Empezar desde cero

Si acabas de clonar el repositorio y quieres reproducir la solución completa, sigue este orden:

### Paso 1. Preparar y desplegar el backend

Configura Python, ejecuta las pruebas y despliega:

→ **[Guía de Despliegue del Backend](docs/deploy-backend.md)**

Cubre: preparación del entorno Python, pruebas unitarias, validación y construcción con AWS SAM, despliegue de Lambda, DynamoDB e IAM, carga de especialidades y verificación del Lambda desplegado.

### Paso 2. Configurar Amazon Connect y el AI Agent

Una vez desplegado el backend:

→ **[Guía de Configuración de Amazon Connect](docs/amazon-connect-setup.md)**

Cubre: dominio de Amazon Q in Connect, Flow Module (Tool), permisos del Security Profile, AI Agent de tipo Orchestration, prompt y reglas del agente, publicación, Bot puente de Amazon Lex y Contact Flow principal.

> ⚠️ El backend debe estar desplegado antes de configurar la Tool, porque necesitas el ARN de la función Lambda.

### Paso 3. Configurar el comportamiento del agente

→ **[Prompt y reglas del AI Agent](docs/prompt-agente.md)**

### Paso 4. Si encuentras algún problema

→ **[Troubleshooting y limitaciones conocidas](docs/troubleshooting.md)**

## Quickstart del backend

Si ya conoces el proyecto y solo necesitas levantar el backend:

```bash
# Clonar el repositorio
git clone <URL_DEL_REPOSITORIO>
cd citas-medicas-ai-agent

# Crear el entorno virtual
python -m venv .venv
.venv\Scripts\Activate.ps1

# Instalar dependencias
pip install -r requirements.txt

# Ejecutar pruebas
pytest tests/ -v

# Validar y construir
sam validate
sam build

# Primer despliegue
sam deploy --guided

# Siguientes despliegues
sam deploy

# Cargar datos iniciales
python scripts/seed_especialidades.py
```

> Para el proceso completo de despliegue, consulta **[docs/deploy-backend.md](docs/deploy-backend.md)**.

## Estructura del proyecto

El código está separado en capas para mantener aisladas la entrada desde Amazon Connect, la lógica de negocio y el acceso a datos.

```text
📦 citas-medicas-ai-agent
├── template.yaml                 # Infraestructura como código (SAM)
├── 📁 src
│   ├── lambda_function.py        # Router de entrada
│   ├── channel_adapter.py        # Adaptador del formato de Connect
│   ├── repository.py             # Operaciones DynamoDB
│   ├── utils.py                  # Utilidades y normalización
│   └── 📁 handlers               # Reglas de negocio
├── 📁 tests                      # Pruebas unitarias
├── 📁 events                     # Eventos para pruebas
├── 📁 scripts
│   └── seed_especialidades.py    # Carga inicial de especialidades
└── 📁 docs
    ├── deploy-backend.md         # Despliegue del backend
    ├── amazon-connect-setup.md   # Configuración de Connect
    ├── prompt-agente.md          # Prompt y reglas del agente
    └── troubleshooting.md        # Problemas y limitaciones
```

## Resultado de las pruebas

Las pruebas se ejecutan 100% en memoria, sin consumir servicios reales de AWS.

| Módulo | Descripción | Pruebas | Resultado |
| :--- | :--- | :---: | :---: |
| `test_handlers.py` | Lógica de agendar, consultar, cancelar y reagendar | 13 | ✅ PASSED |
| `test_utils.py` | Normalización de datos y extracción de códigos | 15 | ✅ PASSED |
| **Total** | **Cobertura de las reglas de negocio** | **28** | **100% PASSED** |

```bash
pytest tests/ -v
```

## Datos de prueba

> ⚠️ Este proyecto utiliza datos ficticios para demostración y pruebas. **No utilizar información real de pacientes ni datos personales.** Las credenciales de AWS tampoco deben almacenarse dentro del repositorio.

---

**Desarrollado por [Sofía Torres](https://github.com/sofia-torres-v) y [Yahaira Pérez](https://github.com/YahairaPerez1994)** ✨
