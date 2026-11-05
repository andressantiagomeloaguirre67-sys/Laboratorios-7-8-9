# ClassroomHub — Laboratorio 8: Autenticación, sesiones, grupos y permisos en Django

**ClassroomHub** amplía el catálogo persistente en MySQL/MariaDB del Laboratorio 7 incorporando **registro público de cuentas sin privilegios (`UserCreationForm`), inicio y cierre de sesión seguros (`LoginView` y `LogoutView` por POST con CSRF), un panel privado con contador de sesión (`/dashboard/`) y una vista de revisión autorizada por permisos (`/resources/review/` para el grupo `Catalog reviewers`)**.

## Requisitos y versiones

- **Python:** 3.12+ (probado con Python 3.14)
- **Django:** 6.1.3
- **Motor SQL:** MariaDB 10.11+ (13.0.2-MariaDB) o MySQL 8.4+
- **Paquetes:** `mysqlclient>=2.2.1,<3`, `python-dotenv>=1,<2`

## Rutas de la aplicación

| Ruta | Método | Nombre | Control de acceso | Descripción |
|---|---|---|---|---|
| `/` | `GET` | `pages:home` | Público | Página de inicio |
| `/resources/` | `GET` | `pages:resources` | Público | Catálogo público de recursos |
| `/about/` | `GET` | `pages:about` | Público | Información del proyecto |
| `/accounts/signup/` | `GET`, `POST` | `pages:signup` | Público (redirige si ya hay sesión) | Registro de cuentas comunes sin privilegios |
| `/accounts/login/` | `GET`, `POST` | `pages:login` | Público | Inicio de sesión (`LoginView`) |
| `/accounts/logout/` | `POST` | `pages:logout` | Autenticado / CSRF | Cierre de sesión mediante `POST` (`GET` devuelve `405`) |
| `/dashboard/` | `GET` | `pages:dashboard` | `@login_required` | Panel privado con contador de visitas en `request.session` |
| `/resources/review/` | `GET` | `pages:resource-review` | `@login_required` + `pages.view_resource` | Vista de revisión reservada al grupo `Catalog reviewers` (`403` sin permiso) |

## Configuración de cuentas y grupo revisor (`Catalog reviewers`)

Las credenciales de base de datos se mantienen únicamente en `.env` (basado en `.env.example`, nunca versionado en Git).

Para preparar el grupo `Catalog reviewers` desde `python manage.py shell` o desde Django Admin (`/admin/`):

```python
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType
from pages.models import Resource

content_type = ContentType.objects.get_for_model(Resource)
view_permission = Permission.objects.get(
    content_type=content_type,
    codename="view_resource",
)
reviewers, _ = Group.objects.get_or_create(name="Catalog reviewers")
reviewers.permissions.add(view_permission)
```

## Ejecución de migraciones y pruebas

Las pruebas se ejecutan de forma aislada sobre la base descartable `test_classroom_hub`:

```bash
python manage.py check
python manage.py makemigrations --check --dry-run
python manage.py test pages
```

Resultado esperado: **28 pruebas exitosas (`OK`)** (10 del Laboratorio 7 + 18 nuevas pruebas de autenticación, sesiones, CSRF y permisos del Laboratorio 8).
