# 📊 REPORTE TÉCNICO DE AUDITORÍA DE BASE DE DATOS (PRISMA) - KAN-14

**ID de la Tarea:** KAN-14 (Subtarea de KAN-11)
**Responsable:** OpenCode Specialist (Bionic Sentinel)
**Estado:** Finalizado
**Metodología:** Validación directa del esquema de Prisma en desarrollo y contraste con la base de datos MariaDB física local.

---

## 1. Mapeo y Diagnóstico del Esquema Actual

Se realizó una auditoría física en tiempo real sobre la base de datos local `mecatrack` y los modelos definidos en `schema.prisma`. Hallazgos críticos:

### A. Tabla `users` (Definición Física)
*   **Columna Crítica:** `role` es un campo de tipo `enum('ADMIN','MANAGER','RECEPTIONIST','TECH')` directamente hardcodeado en la tabla `users` con valor por defecto `'TECH'`.
*   **Problema de Rigidez (Arquitectura):** Cualquier cambio en la definición de roles o permisos requerirá una migración física en el esquema de base de datos (`ALTER TABLE`), lo cual no es dinámico y limita la escalabilidad en producción.
*   **Multi-Tenancy:** La columna `organization_id` está correctamente definida y apunta como FK a `organizations(id)` para el aislamiento principal.

### B. Relación de Multi-Tenancy y Taller (`workshops`)
*   La tabla `organizations` actúa como el inquilino (Tenant) principal de la infraestructura.
*   La tabla `workshops` representa las sucursales de talleres físicos que pertenecen a una organización mediante la relación FK `organization_id`.
*   Los modelos operativos principales, como `repair_orders`, se aíslan mediante `workshop_id` (Taller), garantizando que los datos operativos no se mezclen entre diferentes sucursales pertenecientes a la misma organización.

---

## 2. Análisis de Brechas de Seguridad (Gap Analysis)

1.  **Inexistencia de Permisos Granulares:** No existe una entidad o tabla de permisos (por ejemplo, `read:repair_orders`, `create:expenses`). Las validaciones de acceso se ejecutan de forma inflexible en el backend comparando strings directamente contra el enum del rol del usuario.
2.  **Soporte de Roles Rígido:** Un usuario solo puede tener asignado un rol de forma exclusiva en su organización, impidiendo jerarquías dinámicas o herencia de permisos.
3.  **Seguridad por Capas:** Aunque Prisma aísla los datos mediante relaciones, no hay un middleware centralizado que bloquee fugas de tenant a nivel del query-engine de forma automática.

---

## 3. Plan de Acción Recomendado (Sello Gold)

Para el diseño del nuevo **RBAC-Core** en la siguiente subtarea (`KAN-15`), se propone normalizar la base de datos mediante la creación de 3 tablas dinámicas en Prisma:
*   `roles` (id, name, description)
*   `permissions` (id, key, description)
*   `role_permissions` (role_id, permission_id)
*   Una tabla pivot `user_roles` o la asociación directa de roles dinámicos al usuario.
