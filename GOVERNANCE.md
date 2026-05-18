# Gobernanza Maestra y Configuración Organizacional — IEEE CS USIL

**Documento canónico de gobernanza técnica del capítulo IEEE Computer Society USIL.**
Consolida e integra: la *Propuesta de Gobernanza y Esquema de Trabajo*, los *Anexos Técnicos de Rulesets*, la *Política de Code Review y Tiering de Repositorios* y el modelo de roles organizacionales de GitHub. Reemplaza a esos documentos como referencia única.

| Campo | Valor |
|---|---|
| Versión | 1.0 |
| Estado | Propuesta para ratificación (Fase I–II) |
| Ámbito | Organización GitHub `ieee-cs-usil` |
| Idioma | Español (gobernanza interna, según convención bilingüe) |
| Repositorios de resguardo | `.github` (presentación global) y `docs-internal` |

---

## 0. Tabla de contenido

1. Resumen ejecutivo y principios rectores
2. Composición del equipo y restricciones de diseño
3. Modelo de acceso y roles organizacionales en GitHub
4. Niveles de rigurosidad de repositorios (Tiering)
5. Política de Code Review
6. Configuración de Rulesets (integrada y corregida)
7. Flujo de trabajo y CI/CD
8. Plan de implementación por fases
9. Riesgos residuales
10. Decisiones abiertas (ADRs pendientes)
- Apéndice A: Plantilla `CODEOWNERS`

---

## 1. Resumen ejecutivo y principios rectores

El capítulo opera bajo alta rotación estudiantil y con equipos pequeños. Esta gobernanza convierte el conocimiento individual en activos transferibles y prepara al equipo bajo estándares reales de la industria (RBAC, CI/CD, Rulesets), **calibrando el control a la criticidad real de cada repositorio** para no frenar el aprendizaje con sobre-ingeniería.

**Principios rectores:**

1. **Privilegio mínimo.** Cada persona recibe el menor acceso que le permite trabajar. Los poderes amplios (admin org, escritura global) se conceden por excepción justificada, no por comodidad.
2. **Redundancia antes que profundidad.** Con equipos de 3 personas, todo punto único de falla (un solo *code owner*, un solo administrador) es un riesgo estructural, no hipotético.
3. **El control sigue al riesgo.** Un repo de hackathon y uno que despliega a producción no merecen las mismas reglas. El *tiering* es obligatorio.
4. **La fricción se justifica o se elimina.** Cada regla estricta se evalúa contra la pregunta: *¿protege más de lo que frena, dado que mis colaboradores rotan cada semestre?*
5. **Lo que el documento afirma, el sistema lo hace cumplir.** Una regla declarada pero no configurada técnicamente genera falsa confianza y queda prohibida.

---

## 2. Composición del equipo y restricciones de diseño

| Equipo | Integrantes | Rol funcional |
|---|---|---|
| Dirección Técnica | 2 | Gobierno, arquitectura, escalamiento y *bypass* controlado |
| DevOps Team | 2 | Infraestructura, CI/CD, secretos, Rulesets, seguridad |
| Back-End Team | 3 | Desarrollo y revisión de dominio backend |
| Front-End Team | 3 | Desarrollo y revisión de dominio frontend |
| UX/UI Team | 7 | Diseño + implementación frontend (subconjunto que codifica) |
| Analistas de Sistemas | 5 | Triage de issues y validación funcional (QA) |
| **Total** | **23** | |

**Restricciones que condicionan todo el diseño:**

- **Pool de revisión por dominio = 2 personas reales.** Backend y Frontend tienen 3 integrantes; excluido el autor, quedan 2 revisores. En exámenes o ante una baja a mitad de semestre, el pool colapsa a 1 o 0. Esta es la restricción dominante.
- **Dirección Técnica (2) es el techo de poder.** No puede ser revisor por defecto ni administrador rutinario: es escalamiento y árbitro.
- **DevOps (2) concentra infraestructura y seguridad.** Riesgo de *bus factor* alto: ambas personas deben tener acceso completo y 2FA.
- **Analistas (5) es el mayor recurso de QA y está infrautilizado** si no se le da una función formal de validación.

---

## 3. Modelo de acceso y roles organizacionales en GitHub

### 3.1 Distinción crítica: Propietario de Organización vs. Roles de Organización

GitHub separa dos cosas que no deben confundirse al documentar la configuración:

- **Organization Owner (Propietario):** control total de la organización — ajustes, miembros, facturación, *billing*, todos los repositorios. Es el poder máximo y **no es uno de los 8 roles de organización**; se asigna aparte.
- **Roles de organización (los 8 disponibles):** capacidades org-wide específicas que se asignan a equipos o personas **sin** convertirlas en propietarias. Permiten privilegio mínimo.

**Decisión:** los Propietarios de la organización serán exactamente los **2 integrantes de Dirección Técnica**, con 2FA obligatorio y claves resguardadas en gestor institucional. No habrá un tercer propietario "por si acaso": la redundancia se cubre con 2, y ampliarla diluye el control sobre la cuenta maestra `computer.ieee.usil@gmail.com`.

### 3.2 Matriz de asignación de los 8 roles de organización

| Rol de organización | Asignado a | Justificación | Frecuencia de uso |
|---|---|---|---|
| **All-repository admin** | *No se asigna* | Redundante con Propietario; concederlo a un no-propietario rompe el privilegio mínimo | — |
| **All-repository maintain** | *No se asigna org-wide* | "Maintain" sobre **todos** los repos es demasiado amplio; se concede por repo vía Teams | — |
| **All-repository write** | *No se asigna org-wide* | Conceder escritura global anularía el *tiering* y `CODEOWNERS`; la escritura se da por repo | — |
| **All-repository triage** | **Analistas de Sistemas** (5) | Necesitan gestionar y clasificar issues en todos los repos sin acceso de escritura: encaje exacto con su rol QA | Alta |
| **All-repository read** | *No se asigna* (o equipo "Observadores/Alumni") | El acceso de lectura se gestiona por Teams; uso opcional solo para observadores externos | Baja/Nula |
| **CI/CD Admin** | **DevOps Team** (2) | Administra Actions, *runners*, *runner groups*, secretos y variables org-wide: coincide literalmente con su responsabilidad | Media |
| **Security manager** | **DevOps Team** (2) | Gestiona Secret Scanning, alertas y configuraciones de seguridad — requisito explícito de la gestión de riesgos | Media |
| **Apps manager** | **DevOps Team** (2) | Administra las GitHub Apps de la organización (integraciones de CI/CD); baja frecuencia, encaja en DevOps | Baja |

### 3.3 Por qué tres roles "All-repository" quedan deliberadamente sin asignar

`All-repository write`, `All-repository maintain` y `All-repository admin` son tentadores por comodidad pero **anulan toda esta gobernanza**: conceden acceso uniforme a *todos* los repos, incluida la superficie crítica (Tier 0), saltándose el *tiering*, `CODEOWNERS` y la revisión por dominio. La escritura y el mantenimiento se conceden **por repositorio mediante Teams**, de modo que las reglas de protección sigan aplicando. Esta es una decisión de diseño, no una omisión.

### 3.4 Teams y permiso por repositorio (acceso de grano fino)

El acceso real de desarrollo se gestiona con **Teams**, no con roles org-wide:

| Team | Permiso base por repo | Repos típicos |
|---|---|---|
| `frontend` | Write | `landing-page-frontend`, apps FE |
| `backend` | Write | `landing-page-backend`, apps BE |
| `devops` | Admin (por repo) | infraestructura, `.github`, plantillas |
| `analysts` | Triage (vía rol org) | todos (issues) |
| `direccion-tecnica` | (Propietarios) | todos |
| `ux-ui` | Write en repos FE / Read en backend | repos de producto frontend |

> El subconjunto de `ux-ui` que codifica se añade además al team `frontend` para entrar formalmente al pool de revisión (ver §10, decisión abierta 1).

---

## 4. Niveles de rigurosidad de repositorios (Tiering)

Se extiende la nomenclatura existente (Tier 1 / Tier 2) añadiendo un nivel superior y uno inferior.

| Tier | Nombre | Aprob. humanas | Code owner | CI | Force push `main` | Self-merge | Firma |
|---|---|---|---|---|---|---|---|
| **0** | Crítico | 2 | Sí (≥2 *owners*) | Bloqueante | Bloqueado | No | Recomendada |
| **1** | Estándar | 1 | Sí (≥2 *owners*) | Bloqueante | Bloqueado | No | Opcional |
| **2** | Ligero | 0 (PR + CI) | No | Bloqueante | Bloqueado | Sí (autor) | No |
| **3** | Experimental | 0 | No | No obligatorio | Permitido fuera de `main` | Sí | No |

**Definiciones y mapeo:**

- **Tier 0 — Crítico.** Superficie de seguridad/producción: autenticación, secretos, infraestructura, despliegue a producción. **Se aplica a *paths/directorios* vía `CODEOWNERS`, no a repos completos** (un Tier 0 sobre todo `landing-page-backend` obligaría a casi todo el Back-End Team de 3 a aprobar cada PR).
- **Tier 1 — Estándar.** Productos principales: `landing-page-frontend`, `landing-page-backend` (paths no sensibles), aplicaciones del capítulo.
- **Tier 2 — Ligero.** Documentación y herramientas internas: `docs-internal`, `ieee-cs-usil-template`.
- **Tier 3 — Experimental.** Hackathon, *sandboxes*, prototipos de stack. Nivel deliberado de baja fricción para no matar la curva de aprendizaje.

**Matriz de decisión para un repositorio nuevo:**

| Pregunta | Sí → |
|---|---|
| ¿Maneja secretos, auth o despliega a producción? | **Tier 0** (path-scoped) |
| ¿Es un producto que verán usuarios o evaluadores? | **Tier 1** |
| ¿Es documentación o herramienta interna estable? | **Tier 2** |
| ¿Es aprendizaje, prototipo o hackathon? | **Tier 3** |
| (Ninguna) | Tier 1 por defecto |

---

## 5. Política de Code Review

### 5.1 Modelo: revisión por dominio con respaldo cruzado activado por SLA

El revisor primario pertenece al dominio del cambio vía `CODEOWNERS`. El respaldo cruzado (revisor de otro dominio) **solo se activa cuando el pool de dominio incumple el SLA**, para que la revisión cruzada aporte resiliencia sin degradar la calidad por defecto. La revisión cruzada es además el mecanismo más efectivo contra la rotación: distribuye conocimiento y reduce el *bus factor*.

### 5.2 Pools de revisión efectivos

| Tipo de cambio | Revisor primario | Respaldo cruzado (pool agotado) | Compuerta funcional |
|---|---|---|---|
| Backend | `backend` (≥2 *owners*) | Dirección Técnica · FE con contexto backend | Analistas (bloqueante Tier 0 / asesora Tier 1) |
| Frontend | `frontend` + UX/UI que codifican | UX/UI frontend · BE con contexto frontend | Analistas (asesora) |
| Infra / CI/CD / Secretos | `devops` | Dirección Técnica | — |
| Documentación / ADR | Autor + 1 revisor de cualquier equipo | — | — |

### 5.3 Regla dura de `CODEOWNERS`

> **Ningún path crítico tendrá un solo *code owner*. Mínimo 2 dueños por área, siempre.** Si un área queda con un único dueño elegible por rotación o baja, Dirección Técnica debe asignar un segundo dueño **antes** de habilitar nuevos PR sobre ese path.

### 5.4 Analistas de Sistemas como compuerta funcional

Los 5 Analistas no aprueban el *merge* (rol Triage), pero ejecutan validación funcional sobre *staging* mediante checklist: **bloqueante en Tier 0**, **asesora en Tier 1** (un hallazgo crítico escala a Dirección Técnica), no aplica en Tier 2/3.

### 5.5 SLA de revisión y escalamiento

| Tier | SLA primera revisión (días hábiles) | Escalamiento al excederse |
|---|---|---|
| 0 | 1 | Dirección Técnica asigna revisor |
| 1 | 2 | Dirección Técnica activa respaldo cruzado |
| 2 | 3 | Auto-merge habilitado (CI en verde) |
| 3 | — | Sin SLA |

El incumplimiento del SLA es la condición formal que habilita el respaldo cruzado.

---

## 6. Configuración de Rulesets (integrada y corregida)

Esta sección reemplaza los Anexos Técnicos originales. Correcciones aplicadas respecto de la versión inicial: (a) `required_status_checks` ya no va vacío; (b) `.env.example` deja de bloquearse; (c) se añade Tier 0; (d) la nomenclatura de ramas incluye `hotfix/` y `release/`; (e) se aclara el mecanismo real de identidad de commits.

> **Prerrequisito de validez:** los `required_status_checks` deben rellenarse con el **nombre exacto del *check*** que emite el workflow de GitHub Actions. Hasta entonces, las celdas "CI: Bloqueante" del §4 son aspiracionales, no efectivas. Esto bloquea el cierre de la Fase III.

### 6.1 Tier 0 — Crítico (path-scoped)

```json
{
  "name": "Tier 0 Critical Protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/heads/main", "refs/heads/develop"], "exclude": [] }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 2,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "require_last_push_approval": true,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [
          { "context": "build" },
          { "context": "lint" },
          { "context": "test" }
        ]
      }
    },
    { "type": "required_signatures" },
    { "type": "block_force_push" }
  ]
}
```

> **Nota técnica honesta:** GitHub no permite exigir nativamente "1 aprobación de Backend **+** 1 de Dirección Técnica". El requisito de §5.2 se aproxima componiendo el `CODEOWNERS` del *path* sensible con **ambos** equipos como dueños y exigiendo `required_approving_review_count: 2` + `require_code_owner_review`. `required_signatures` se incluye porque en superficie crítica la verificación de identidad sí justifica la fricción.

### 6.2 Tier 1 — Estándar

```json
{
  "name": "Tier 1 Protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/heads/main", "refs/heads/develop"], "exclude": [] }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [
          { "context": "build" },
          { "context": "lint" }
        ]
      }
    },
    { "type": "block_force_push" }
  ]
}
```

### 6.3 Tier 2 — Ligero

```json
{
  "name": "Tier 2 Protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/heads/main"], "exclude": [] }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 0,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": false,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [ { "context": "build" } ]
      }
    },
    { "type": "block_force_push" }
  ]
}
```

### 6.4 Tier 3 — Experimental

```json
{
  "name": "Tier 3 Experimental",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/heads/main"], "exclude": [] }
  },
  "rules": [
    { "type": "block_force_push" }
  ]
}
```

> Solo protege `main` contra reescritura de historial. Las ramas de trabajo quedan libres deliberadamente para permitir experimentación sin fricción.

### 6.5 Nomenclatura de ramas (corregida)

Se añaden `hotfix/` y `release/` (y `perf`, `test`, `ci`) para no bloquear flujos legítimos de urgencia o versión.

```json
{
  "name": "Branch Naming Convention",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["~ALL"], "exclude": ["refs/heads/main", "refs/heads/develop"] }
  },
  "rules": [
    {
      "type": "branch_name_pattern",
      "parameters": {
        "operator": "regex",
        "pattern": "^(feat|fix|hotfix|release|docs|chore|refactor|perf|test|ci)/.+$"
      }
    }
  ]
}
```

### 6.6 Identidad de commits (corrección de intención)

La gobernanza original pedía "bloquear commits si el correo del autor no coincide con su cuenta verificada". Técnicamente, un patrón de email **no verifica identidad**; los desarrolladores usan cuentas personales y no un dominio corporativo. El intento real se cumple mejor con:

1. **`required_signatures` en Tier 0** (ya incluido en §6.1) — garantía criptográfica de identidad.
2. **Vigilant mode** activado a nivel de usuario para mostrar commits no verificados.
3. Opcionalmente, `commit_author_email_pattern` solo para **bloquear correos `noreply`/placeholder**, no como verificación de identidad:

```json
{
  "name": "Commit Author Email Guard (opcional)",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/heads/main", "refs/heads/develop"], "exclude": [] }
  },
  "rules": [
    {
      "type": "commit_author_email_pattern",
      "parameters": {
        "operator": "regex",
        "pattern": "^(?!.*users\\.noreply\\.github\\.com$).+@.+\\..+$",
        "negate": false
      }
    }
  ]
}
```

### 6.7 Protección de etiquetas de release

```json
{
  "name": "Release Tag Protection",
  "target": "tag",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["refs/tags/v*"], "exclude": [] }
  },
  "rules": [ { "type": "creation" }, { "type": "deletion" }, { "type": "update" } ],
  "bypass_actors": [
    { "actor_id": 1, "actor_type": "OrganizationAdmin", "bypass_mode": "always" }
  ]
}
```

> **Verificar antes de aplicar:** confirmar `actor_id` / `actor_type` contra la organización real (UI o API). Un *bypass* mal configurado puede dejar al capítulo sin capacidad de gestionar tags.

### 6.8 Prevención de fuga de secretos (corrección de `.env.example`)

El patrón amplio `.env.*` bloqueaba también `.env.example`, archivo que **sí** se quiere versionar para onboarding. Se reemplaza por nombres explícitos:

```json
{
  "name": "Prevent Secret Leaks",
  "target": "push",
  "enforcement": "active",
  "rules": [
    {
      "type": "file_path_restriction",
      "parameters": {
        "restricted_file_paths": [
          ".env",
          ".env.local",
          ".env.development",
          ".env.production",
          "*.pem",
          "*.key"
        ]
      }
    }
  ]
}
```

> `.env.example` queda fuera de la lista a propósito: es la plantilla de entorno para el onboarding y debe versionarse. Complementado por Secret Scanning nativo y `.gitignore` inyectado en la plantilla.

---

## 7. Flujo de trabajo y CI/CD

**Convención bilingüe:** inglés obligatorio en código, ramas, commits, PR y documentación técnica base (`README.md`, `CONTRIBUTING.md`); español en gestión de tickets, manuales de relevo y ADRs.

**Ciclo de Pull Request:**

1. **Apertura:** vincular al issue (`Closes #NN`).
2. **CI Pipeline:** linting + build automáticos (los *checks* que alimentan §6).
3. **Revisión técnica:** según pool y SLA del §5.
4. **Validación funcional:** compuerta de Analistas según tier.
5. **Merge:** *Squash & Merge* para historial limpio.

**CI/CD:** CI en cada PR hacia `develop`/`main`. CD post-merge en `main` (Producción) o `develop` (Staging) hacia Vercel/Supabase. *Rollback* mediante Instant Rollback de Vercel + `git revert` ordenado posterior.

---

## 8. Plan de implementación por fases

**Fase I — Infraestructura base.** Cuenta institucional con 2FA; organización `ieee-cs-usil`; **asignación de los 2 Propietarios (Dirección Técnica)** y de los roles de organización del §3.2; creación de Teams del §3.4.

**Fase II — Estandarización.** Repositorio `ieee-cs-usil-template` con `/docs`, `.gitignore` estricto y `.env.example`; clasificación de cada repo en su Tier (§4); aplicación de Rulesets del §6 a nivel de organización; `CODEOWNERS` con regla de redundancia.

**Fase III — Automatización y seguridad.** Workflows de CI; **rellenar `required_status_checks` con los nombres reales de los *checks*** (prerrequisito de validez); GitHub Secrets para Staging/Producción gestionados por DevOps (CI/CD Admin); Secret Scanning activado por Security manager.

**Fase IV — Sostenibilidad.** `docs-internal` con este documento versionado; ADRs del §10 resueltos; manual de transferencia del rol DevOps (mitiga el *bus factor* de 2 personas).

---

## 9. Riesgos residuales

| Riesgo | Mitigación |
|---|---|
| Pool de revisión backend/frontend = 2 personas | Regla dura ≥2 *code owners* + respaldo cruzado por SLA |
| *Bus factor* de DevOps (2 personas) | Manual de transferencia (Fase IV) + ambos con acceso y 2FA |
| Roles "All-repository write/admin" usados por comodidad | Prohibidos org-wide; acceso solo por Teams (§3.3) |
| Tier 0 aplicado a repos completos | Tier 0 acotado a *paths* vía `CODEOWNERS` |
| CI declarado pero no exigido | Prerrequisito explícito en Fase III antes de cerrar |
| Revisión cruzada degradando calidad | Activada solo por incumplimiento de SLA |
| Sobre-ingeniería desincentivando estudiantes | Tier 3 de baja fricción |
| `bypass_actors` mal configurado en tags | Verificación obligatoria antes de aplicar (§6.7) |

---

## Apéndice A — Plantilla `CODEOWNERS`

Ubicación: `.github/CODEOWNERS` en cada repo. **Regla dura: ningún path con un solo dueño.**

```
# CODEOWNERS — IEEE CS USIL
# Regla: minimo 2 owners por path. Nunca un solo owner.

# Defecto del repo (Tier 1): equipo de dominio + fallback Direccion Tecnica
*                       @ieee-cs-usil/backend @ieee-cs-usil/direccion-tecnica

# Frontend
/src/ui/                @ieee-cs-usil/frontend @ieee-cs-usil/ux-ui
/src/components/        @ieee-cs-usil/frontend @ieee-cs-usil/ux-ui

# Backend
/src/api/               @ieee-cs-usil/backend @ieee-cs-usil/direccion-tecnica
/src/services/          @ieee-cs-usil/backend @ieee-cs-usil/direccion-tecnica

# --- Superficie Tier 0 (critica): dominio + Direccion Tecnica obligatorio ---
/src/auth/              @ieee-cs-usil/backend @ieee-cs-usil/direccion-tecnica
/src/config/secrets/    @ieee-cs-usil/devops  @ieee-cs-usil/direccion-tecnica
/.github/workflows/     @ieee-cs-usil/devops  @ieee-cs-usil/direccion-tecnica
/infra/                 @ieee-cs-usil/devops  @ieee-cs-usil/direccion-tecnica
```

---
