# ClassroomHub — Laboratorio 9: Formularios y CRUD con Django (Vistas, ModelForm y Permisos)

**ClassroomHub** integra en este laboratorio un flujo completo de **CRUD (creación, consulta, actualización y eliminación) de recursos** desde la propia aplicación web, utilizando `ResourceForm` (`ModelForm`), vistas genéricas basadas en clases (`DetailView`, `CreateView`, `UpdateView`, `DeleteView`), `LoginRequiredMixin` y `PermissionRequiredMixin` para proteger cada operación según su permiso específico sobre MySQL/MariaDB.

## Requisitos y versiones

- **Python:** 3.12+ (probado con Python 3.14)
- **Django:** 6.1.3
- **Motor SQL:** MariaDB 10.11+ (13.0.2-MariaDB) o MySQL 8.4+
- **Paquetes:** `mysqlclient>=2.2.1,<3`, `python-dotenv>=1,<2`

## Rutas de la aplicación y permisos

```text
GET  /resources/
GET  /resources/<pk>/
GET  /resources/new/
POST /resources/new/
GET  /resources/<pk>/edit/
POST /resources/<pk>/edit/
GET  /resources/<pk>/delete/
POST /resources/<pk>/delete/
```

| Operación / Ruta | Anónimo | Cuenta común | `Catalog reviewers` | `Catalog editors` |
|---|---|---|---|---|
| `GET /resources/` y `GET /resources/<pk>/` | Sí | Sí | Sí | Sí |
| `GET /dashboard/` | Redirige a login | Sí | Sí | Sí |
| `GET /resources/review/` (`pages.view_resource`) | Redirige a login | `403` | Sí | Sí |
| `GET/POST /resources/new/` (`pages.add_resource`) | Redirige a login | `403` | `403` | Sí |
| `GET/POST /resources/<pk>/edit/` (`pages.change_resource`) | Redirige a login | `403` | `403` | Sí |
| `GET/POST /resources/<pk>/delete/` (`pages.delete_resource`) | Redirige a login | `403` | `403` | Sí |

## Configuración de categorías y del grupo `Catalog editors`

- Las **categorías (`Category`)** se administran desde Django Admin (`/admin/`).
- El formulario de registro público (`/accounts/signup/`) **nunca asigna privilegios ni grupos**.
- Para crear el grupo `Catalog editors` con los cuatro permisos sobre `Resource` (`view_resource`, `add_resource`, `change_resource`, `delete_resource`) desde `python manage.py shell` o desde Django Admin:

```python
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType
from pages.models import Resource

content_type = ContentType.objects.get_for_model(Resource)
required_codes = [
    "view_resource",
    "add_resource",
    "change_resource",
    "delete_resource",
]
permissions = list(
    Permission.objects.filter(
        content_type=content_type,
        codename__in=required_codes,
    )
)
editors, _ = Group.objects.get_or_create(name="Catalog editors")
editors.permissions.add(*permissions)
```

## Ejecución de pruebas automatizadas

Las pruebas se ejecutan de forma aislada sobre la base descartable `test_classroom_hub`:

```bash
python manage.py check
python manage.py makemigrations --check --dry-run
python manage.py test pages
```

Resultado esperado: **55 pruebas exitosas (`OK`)** (10 del Laboratorio 7 + 18 del Laboratorio 8 + 27 pruebas de CRUD, validaciones de `ResourceForm`, CSRF y permisos del Laboratorio 9).
