# Estándares de Arquitectura e Ingeniería (Ecosistema GIPC)

## 1. Visión del Ecosistema (Core y Satélites)
- **Core GIPC (Personas y Cumplimiento):** Es la "Fuente de la Verdad" (Source of Truth). Gestiona la identidad, roles y datos maestros de las personas.
- **Módulos Satélite (Inmobiliario, Automotores, etc.):** Son microservicios independientes. NUNCA deben duplicar la gestión de usuarios. Deben referenciar a las personas del Core GIPC mediante IDs únicos.
- **Comunicación Inter-Módulos:** Se realizará de forma asíncrona mediante **Apache Kafka**. El Core GIPC emitirá eventos (ej. `PersonaCreada`, `CumplimientoActualizado`) y los módulos satélite consumirán estos eventos usando contratos de datos estrictos (Esquemas JSON/Avro).

## 2. Stack Tecnológico Base
- **Backend:** Java moderno (LTS), Spring Boot, Spring Data JPA / Hibernate.
- **Seguridad (IAM):** Keycloak. El backend actúa exclusivamente como OAuth2 Resource Server.
- **Utilidades:** Lombok (para reducir boilerplate) y Jakarta Bean Validation.
- **Pruebas:** JUnit 5, Mockito, AssertJ.
- **Frontend (Transición):** Thymeleaf inicial (con HTMX si es necesario para interactividad ligera), pero con diseño estricto **API-First** para una futura migración a React/Angular.

## 3. Arquitectura Limpia (Clean Architecture)
Mantén una estricta separación por capas. Está prohibido saltarse capas o acoplar lógica de negocio en la infraestructura:
- **API/Web Layer:** Controladores (`@RestController`), gestión de enrutamiento HTTP y validación inicial de entradas.
- **DTOs & Mappers:** Uso exclusivo de DTOs para transferencia de datos. NUNCA exponer entidades JPA. Preferir el uso de `Records` o clases inmutables para los DTOs.
- **Application/Service Layer:** Contiene la lógica de negocio, casos de uso y transaccionalidad (`@Transactional`).
- **Domain/Core Layer:** Entidades JPA, lógica pura del dominio.
- **Repository/Persistence Layer:** Interfaces Spring Data JPA .

## 4. Reglas Técnicas y Base de Datos
- **Relaciones JPA:** Utilizar carga diferida (`FetchType.LAZY`) por defecto en todas las relaciones para evitar problemas de rendimiento (N+1 Selects).
- **Auditoría:** Mantener rastro automático de creación y modificación (`createdAt`, `updatedAt`) en datos principales.
- **Inyección de Dependencias:** Preferir siempre la inyección por constructor (usando `@RequiredArgsConstructor` de Lombok) sobre `@Autowired` en campos, para facilitar el testing.
- **Gestión de Errores:** Implementar un `@RestControllerAdvice` global que retorne errores estandarizados (Problem Details RFC 7807).