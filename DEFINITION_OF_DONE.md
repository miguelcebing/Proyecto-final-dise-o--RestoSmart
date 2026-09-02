# Definición de "Done" para RestoSmart

Esta definición establece los criterios que deben cumplirse para considerar que una historia de usuario está completa y lista para ser entregada.

## Criterios Generales de "Done"

### 1. Código
- [ ] El código está escrito siguiendo los estándares de codificación del proyecto
- [ ] El código ha sido revisado mediante pull request aprobado por al menos un miembro del equipo
- [ ] El código está integrado en la rama `develop` (o rama correspondiente)
- [ ] No hay comentarios de revisión pendientes
- [ ] El código cumple con las guías de estilo establecidas (linting passed)

### 2. Pruebas
- [ ] Se han escrito pruebas unitarias para la nueva funcionalidad (cobertura mínima del 80%)
- [ ] Se han escrito pruebas de integración cuando corresponda
- [ ] Todas las pruebas pasan en el entorno de CI/CD
- [ ] Se han realizado pruebas manuales de la funcionalidad
- [ ] Se han considerado y probado casos edge

### 3. Documentación
- [ ] Se ha actualizado la documentación técnica relevante (API docs, arquitectura, etc.)
- [ ] Se ha actualizado la documentación de usuario si aplica
- [ ] Se han actualizado los diagramas de arquitectura o flujo si es necesario
- [ ] Los comentarios en el código son claros y explican la lógica compleja

### 4. Integración y Despliegue
- [ ] La funcionalidad se despliega correctamente en el entorno de staging
- [ ] No introduce regresiones en funcionalidades existentes
- [ ] Se ha verificado la compatibilidad con versiones anteriores cuando aplica
- [ ] Los cambios en la base de datos (si los hay) tienen scripts de migración adecuados

### 5. Seguridad y Rendimiento
- [ ] No se han introducido vulnerabilidades de seguridad conocidas
- [ ] Se ha considerado el impacto en el rendimiento y se ha optimizado cuando necesario
- [ ] Se han validado los inputs y outputs adecuadamente
- [ ] Se siguen las mejores prácticas de seguridad para el contexto

### 6. Aceptación del Producto
- [ ] Se cumplen todos los criterios de aceptación definidos en la historia de usuario
- [ ] El Product Owner ha aceptado la historia de usuario
- [ ] Se ha demostrado la funcionalidad en la revisión de sprint
- [ ] Se ha obtenido feedback de stakeholders cuando corresponde

## Criterios Específicos por Tipo de Historia

### Para Historias de Interfaz de Usuario (UI/UX)
- [ ] La interfaz es responsive y funciona en los dispositivos objetivo
- [ ] Se siguen las guías de estilo y componentes del sistema de diseño
- [ ] Se ha realizado prueba de usabilidad básica
- [ ] Los estados de carga, error y vacío están manejados apropiadamente

### Para Historias de API/Backend
- [ ] Los endpoints están documentados (OpenAPI/Swagger o similar)
- [ ] Se manejan adecuadamente los códigos de error HTTP
- [ ] Se valida la entrada y salida de datos
- [ ] Se implementa el manejo adecuado de excepciones
- [ ] Se sigue el patrón RESTful o el estándar acordado

### Para Historias de Integración (pagos, notificaciones, etc.)
- [ ] Se han configurado credenciales de prueba para servicios externos
- [ ] Se maneja adecuadamente la desconexión o fallos de servicios externos
- [ ] Se registran adecuadamente las operaciones para auditoría
- [ ] Se implementan reintentos cuando corresponde

### Para Historias de Base de Datos
- [ ] Los cambios de esquema tienen scripts de migración documentados y testeados
- [ ] Se han considerado los impactos en el rendimiento de las consultas
- [ ] Se han implementado índices apropiados cuando necesario
- [ ] Se siguen las mejores prácticas de normalización o desnormalización

## Proceso de Verificación

1. El desarrollador completa la historia y crea un pull request
2. El pull request pasa por revisión de código (al menos un aprobador)
3. Las pruebas automatizadas se ejecutan en CI/CD y pasan
4. El desarrollador realiza una demostración al Product Owner
5. El Product Owner verifica que se cumplan los criterios de aceptación
6. Si todo está correcto, la historia se marca como "Done" y se cierra el issue

## Excepciones

Cualquier excepción a esta definición debe ser aprobada explícitamente por el Product Owner y el Scrum Master, y documentada en el issue correspondiente.