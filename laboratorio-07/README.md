# ClassroomHub — Laboratorio 7: Persistencia de datos con Django y MySQL/MariaDB

**ClassroomHub** evoluciona el catálogo del Laboratorio 6 hacia una aplicación con persistencia real en **MySQL o MariaDB** mediante el ORM de Django, migraciones versionadas (`Category` y `Resource`) y gestión administrativa en Django Admin.

## Requisitos y versiones

- **Python:** 3.12+ (probado con Python 3.14)
- **Django:** 6.1.3
- **Motor SQL:** MariaDB 10.11+ (13.0.2-MariaDB) o MySQL 8.4+
- **Controlador y configuración:** `mysqlclient>=2.2.1,<3`, `python-dotenv>=1,<2`

## Estructura del proyecto

```text
classroom-hub/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── pages/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations/
    │   ├── __init__.py
    │   ├── 0001_initial.py
    │   └── 0002_resource_resource_type.py
    ├── models.py
    ├── tests.py
    ├── urls.py
    ├── views.py
    └── templates/
        └── pages/
            ├── base.html
            ├── home.html
            ├── resources.html
            └── about.html
```

## Preparación de la base de datos (MySQL / MariaDB)

Ejecuta con una cuenta administrativa SQL:

```sql
CREATE DATABASE IF NOT EXISTS classroom_hub
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'classroomhub_user'@'127.0.0.1'
    IDENTIFIED BY 'replace_with_a_private_password';

GRANT ALL PRIVILEGES ON classroom_hub.*
    TO 'classroomhub_user'@'127.0.0.1';

GRANT ALL PRIVILEGES ON test_classroom_hub.*
    TO 'classroomhub_user'@'127.0.0.1';
```

## Instalación y configuración

1. **Crear y activar el entorno virtual e instalar dependencias:**

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   python -m pip install -r requirements.txt
   ```

2. **Configurar variables de entorno (`.env`):**

   Copia `.env.example` a `.env` y define las credenciales locales (no publiques `.env` en Git):

   ```bash
   cp .env.example .env
   ```

3. **Aplicar migraciones y verificar el proyecto:**

   ```bash
   python manage.py check
   python manage.py migrate
   ```

4. **Crear un superusuario para Django Admin:**

   ```bash
   python manage.py createsuperuser
   ```

5. **Arrancar el servidor de desarrollo:**

   ```bash
   python manage.py runserver
   ```

   - Catálogo público: `http://127.0.0.1:8000/resources/`
   - Panel de administración: `http://127.0.0.1:8000/admin/`

## Pruebas automatizadas

Las pruebas utilizan la base aislada y descartable `test_classroom_hub`:

```bash
python manage.py test pages
```

Resultado esperado: **10 pruebas (`OK`)** que comprueban renderizado de vistas, consulta ORM con relaciones (`Category` -> `Resource`), persistencia tras `save()`, protección `ProtectedError` al intentar eliminar categorías referenciadas, estado vacío y relación inversa `category.resources`.
