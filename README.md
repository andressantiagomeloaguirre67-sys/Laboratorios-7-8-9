# Laboratorios 7, 8 y 9 — ClassroomHub (Django + MySQL/MariaDB)

Repositorio con el desarrollo completo de los **Laboratorios 7, 8 y 9** del proyecto **ClassroomHub** construido con **Python**, **Django 6.1.3** y **MariaDB / MySQL**.

## Estructura del repositorio

| Carpeta | Laboratorio | Contenido principal | Pruebas automatizadas |
|---|---|---|---|
| [`laboratorio-07/`](./laboratorio-07/) | **Laboratorio 7** — Persistencia de datos con Django y MySQL/MariaDB | Configuración segura con `.env`, modelos `Category` y `Resource` (`models.PROTECT`), migraciones (`0001_initial` y `0002_resource_resource_type`), consultas ORM (`select_related`), Django Admin y catálogo público persistente. | **10 pruebas (`OK`)** en `ClassroomHubTests` |
| [`laboratorio-08/`](./laboratorio-08/) | **Laboratorio 8** — Autenticación, sesiones, grupos y permisos | Registro de cuentas sin privilegios (`UserCreationForm`), `LoginView`, `LogoutView` por `POST` con CSRF, panel privado `/dashboard/` con contador en `request.session` y vista `/resources/review/` restringida al grupo `Catalog reviewers` (`pages.view_resource`). | **28 pruebas (`OK`)** (`ClassroomHubTests` + `AuthenticationTests` + `ResourceReviewPermissionTests`) |
| [`laboratorio-09/`](./laboratorio-09/) | **Laboratorio 9** — Formularios y CRUD con Django: Vistas, `ModelForm` y Permisos | `ResourceForm` (`ModelForm` con validación `clean_title`), vistas genéricas `ResourceDetailView`, `ResourceCreateView`, `ResourceUpdateView` y `ResourceDeleteView` protegidas con `LoginRequiredMixin` y `PermissionRequiredMixin` para el grupo `Catalog editors`. | **55+ pruebas (`OK`)** (`ClassroomHubTests` + `AuthenticationTests` + `ResourceReviewPermissionTests` + `ResourceCrudTests`) |

## Ejecución rápida (para cualquiera de los laboratorios)

1. Entrar a la carpeta del laboratorio deseado (`laboratorio-07`, `laboratorio-08` o `laboratorio-09`):
   ```bash
   cd laboratorio-09
   ```
2. Crear y activar el entorno virtual e instalar dependencias:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   python -m pip install -r requirements.txt
   ```
3. Copiar `.env.example` a `.env` y configurar las credenciales locales de MySQL o MariaDB:
   ```bash
   cp .env.example .env
   ```
4. Aplicar migraciones y ejecutar la suite de pruebas sobre la base aislada `test_classroom_hub`:
   ```bash
   python manage.py migrate
   python manage.py test pages
   ```
5. Iniciar el servidor de desarrollo:
   ```bash
   python manage.py runserver
   ```
