# 🤖 WhatsApp Modular Chatbot Engine

[![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)](https://php.net)
[![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://mysql.com)

## 📋 Resumen del Proyecto
Este proyecto consiste en un **Motor de Chatbot Modular** desarrollado para una empresa del sector automotriz. El sistema fue diseñado para permitir la creación de flujos conversacionales dinámicos (encuestas NPS, seguimiento de ventas, soporte) de forma 100% declarativa a través de base de datos, eliminando la necesidad de modificar código fuente para implementar nuevos flujos.

> [!NOTE]
> **Aviso de Confidencialidad:** El código fuente original es propiedad privada. Este repositorio sirve como **Caso de Estudio Técnico**, detallando la arquitectura, patrones de diseño y lógica implementada.

---

## 🏗️ Arquitectura del Sistema

El motor utiliza una arquitectura desacoplada donde cada componente tiene una responsabilidad única, facilitando el mantenimiento y la escalabilidad.

```mermaid
flowchart TB
    subgraph ClientLayer["📱 Capa de Cliente"]
        WA[WhatsApp Business API]
        User((Usuario Final))
    end

    subgraph LogicLayer["⚙️ Core Engine (Laravel)"]
        CFS[ChatbotFlowService<br>Orquestador Principal]
        
        subgraph InternalServices["Servicios Especializados"]
            SP[StepProcessor<br>Gestión de Plantillas]
            RV[ResponseValidator<br>Motores de Validación]
            CE[ConditionEvaluator<br>Navegación Lógica]
            AE[ActionExecutor<br>Webhooks y Side-Effects]
        end
    end

    subgraph DataLayer["🗄️ Persistencia"]
        DB[(Base de Datos MySQL)]
        Cache[(Cache de Sesión)]
    end

    User <-->|Interaction| WA
    WA <-->|Webhooks/API| LogicLayer
    CFS --> InternalServices
    LogicLayer <--> DataLayer
```

---

## ✨ Características Principales

### 1. Motor de Flujos Declarativo
Los flujos no están "hard-coded". Se definen mediante estructuras JSON en la base de datos que especifican:
- **Pasos:** Mensajes, tipos de input y lógica de navegación.
- **Validaciones:** Reglas personalizadas (DNI, Email, Rangos numéricos).
- **Acciones:** Disparadores automáticos al responder (Ej: enviar datos a CRM externo).

### 2. Navegación Inteligente (Branching)
Soporta lógica condicional compleja basada en respuestas previas o datos del cliente.
- **Skip Logic:** Salta preguntas innecesarias si el dato ya existe en el perfil del cliente.
- **Conditional Branching:** Cambia el rumbo de la conversación según la calificación del usuario.

### 3. UX de WhatsApp Business
- **Mensajes Interactivos:** Uso de botones de respuesta rápida y listas para minimizar errores de entrada.
- **Soporte de Plantillas (Templates):** Capacidad de iniciar conversaciones proactivas cumpliendo con las políticas de WhatsApp.

### 4. Robustez y Seguridad
- **Rate Limiting (Debounce):** Protección integrada contra ráfagas de mensajes rápidos para evitar inconsistencias en el estado de la sesión.
- **Session Management:** Gestión de tiempos de expiración y persistencia de datos parciales.

---

## 📊 Modelo de Datos (Esquema ER)

El diseño de la base de datos permite la coexistencia de múltiples flujos activos simultáneamente.

```mermaid
erDiagram
    FLOW ||--o{ STEP : "defines"
    SESSION ||--o{ SESSION_DATA : "persists"
    FLOW ||--o{ SESSION : "instantiates"

    FLOW {
        string flow_code PK
        string name
        json global_config
        boolean is_active
    }

    STEP {
        string step_code PK
        text message_content
        string input_type
        json navigation_rules
        json validation_rules
    }

    SESSION {
        int id PK
        string wa_id "WhatsApp ID"
        string status "ACTIVE/COMPLETED"
        json context_data
        datetime expires_at
    }

    SESSION_DATA {
        int id PK
        string data_key
        text value
        datetime captured_at
    }
```

---

## 🛠️ Tecnologías Utilizadas

- **Backend:** Laravel 8.x / PHP 7.4+
- **Database:** MySql 8.0
- **Integraciones:** WhatsApp Business Cloud API
- **Documentación Técnica:** Mermaid.js para diagramas de secuencia e infraestructura.

---

## 📈 Lógica de Navegación (Ejemplo de Caso de Uso NPS)

A continuación se muestra cómo el motor procesa un flujo de satisfacción (NPS) típico:

```mermaid
sequenceDiagram
    participant U as Usuario
    participant E as Chatbot Engine
    participant C as CRM Externo

    E->>U: Envío de Plantilla de Bienvenida
    U->>E: Click en "Empezar"
    E->>E: Valida Perfil (¿Tiene Email?)
    alt No tiene Email
        E->>U: Pregunta por Email
        U->>E: Proporciona Email
    end
    E->>U: Pregunta NPS (1-10)
    U->>E: Responde "3"
    Note right of E: Lógica detecta Detractor
    E->>U: Pregunta ¿Qué podemos mejorar?
    U->>E: "La entrega se demoró"
    E->>C: Push de alerta de Detractor en tiempo real
    E->>U: Despedida Personalizada
```

---

## 💡 Patrones de Diseño Aplicados

- **Strategy Pattern:** Para los diferentes validadores de respuesta.
- **Template Method:** Para el procesamiento estandarizado de cada paso conversacional.
- **State Pattern:** Para gestionar el ciclo de vida de las sesiones de usuario.

---

## 📄 Conclusión
Este proyecto demuestra habilidades avanzadas en arquitectura de software, gestión de APIs de mensajería a escala y diseño de sistemas reactivos orientados a la experiencia del usuario (UX) conversacional.

---
*Este documento fue elaborado como evidencia de capacidad técnica y diseño arquitectónico.*
