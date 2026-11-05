# Informe Técnico — Laboratorio 7: Persistencia de datos con Django y MySQL/MariaDB

## 1. Enlaces del Repositorio en GitHub

- **Repositorio general (Laboratorios 7, 8 y 9):** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9)
- **Carpeta específica del Laboratorio 7 en GitHub:** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-07](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-07)
- **Ubicación local:** `/home/andres/Descargas/laboratorio-07`

---

## 2. Resumen Ejecutivo

En el **Laboratorio 7** se transformó **ClassroomHub** (que en el Laboratorio 6 mostraba una lista fija de diccionarios en memoria dentro de `views.py`) en una aplicación web respaldada por una base de datos relacional **MariaDB / MySQL**. Se diseñaron los modelos `Category` y `Resource` con relación uno a muchos protegida (`on_delete=models.PROTECT`), se gestionó la evolución del esquema mediante migraciones de Django, se habilitó el panel administrativo (**Django Admin**) y se implementó una suite de **10 pruebas automatizadas** ejecutadas sobre una base de pruebas aislada (`test_classroom_hub`).

---

## 3. Detalle Paso a Paso de lo Realizado

### Práctica 7.1 — Preparación de MariaDB/MySQL y conexión segura desde Django
1. **Creación de la base de datos de desarrollo:**
   - Se creó el esquema `classroom_hub` con codificación `utf8mb4` y colación `utf8mb4_unicode_ci` para soportar texto Unicode completo.
2. **Cuenta SQL de privilegios mínimos:**
   - Se creó el usuario `'classroomhub_user'@'127.0.0.1'` otorgándole permisos únicamente sobre `classroom_hub.*` (desarrollo) y `test_classroom_hub.*` (base descartable para pruebas), evitando usar `root` en la aplicación.
3. **Instalación de controladores y variables de entorno:**
   - Se instalaron `mysqlclient>=2.2.1,<3` y `python-dotenv>=1,<2` en el entorno virtual y se registraron en `requirements.txt`.
   - Se creó `.env.example` (versionado en Git sin secretos reales) y `.env` (ignorado en `.gitignore` con las credenciales locales).
   - Se actualizó `config/settings.py` para cargar `.env` mediante `load_dotenv(BASE_DIR / ".env")` y configurar `DATABASES["default"]` con el motor `django.db.backends.mysql`, `OPTIONS={"charset": "utf8mb4"}` y `TEST={"NAME": "test_classroom_hub"}`.
   - Se comprobó la conexión real desde `python manage.py shell` con `SELECT VERSION(), DATABASE()`.

### Práctica 7.2 — Definición de modelos y relaciones (`pages/models.py`)
1. **Modelo `Category`:**
   - Campos: `name = models.CharField(max_length=80, unique=True)` y `description = models.TextField(blank=True)`.
   - Metadatos: `ordering = ["name"]`, `verbose_name_plural = "categories"` y método `__str__`.
2. **Modelo `Resource`:**
   - Campos: `title = models.CharField(max_length=150)`, `description = models.TextField(blank=True)`, `available = models.BooleanField(default=True)`, `created_at = models.DateTimeField(auto_now_add=True)`.
   - Relación: `category = models.ForeignKey(Category, on_delete=models.PROTECT, related_name="resources")`, impidiendo que se elimine una categoría mientras tenga recursos asociados.

### Práctica 7.3 — Migraciones y evolución del esquema
1. **Migración inicial (`0001_initial.py`):**
   - Se generó con `python manage.py makemigrations pages`, se inspeccionó el SQL con `sqlmigrate pages 0001` y se aplicó con `python manage.py migrate`, verificando en MariaDB las tablas `pages_category` y `pages_resource` con motor `InnoDB`.
2. **Evolución del modelo (`0002_resource_resource_type.py`):**
   - Se añadió el campo `resource_type = models.CharField(max_length=40, blank=True)` a `Resource` y se aplicó la segunda migración sin pérdida de datos existentes.

### Práctica 7.4 — Operaciones con el ORM de Django
1. **Creación y consulta de registros:**
   - Se insertaron las tres categorías (`Programming`, `Collaboration`, `Web development`) y los tres recursos (`Python basics`, `Git and GitHub`, `Django models`).
   - Se verificaron consultas con `all()`, `filter()`, `get()`, `first()`, actualizaciones persistentes con `save(update_fields=["available"])`, borrado de registros temporales, recorrido de relaciones inversas (`category.resources.all()`), captura de `ProtectedError` al intentar borrar una categoría referenciada y optimización de consultas con `select_related("category")`.

### Práctica 7.5 — Configuración de Django Admin (`pages/admin.py`)
1. **Superusuario y registro de modelos:**
   - Se creó el superusuario local (`admin`) y se registraron `CategoryAdmin` (`list_display`, `search_fields`) y `ResourceAdmin` (`list_display`, `list_filter`, `search_fields`, `list_select_related = ("category",)`, `list_per_page = 20`).

### Práctica 7.6 — Catálogo público conectado al ORM y pruebas automatizadas
1. **Vista y plantilla públicas:**
   - En `pages/views.py`, la vista `resources(request)` se actualizó para consultar `Resource.objects.select_related("category").all()`.
   - En `pages/templates/pages/resources.html`, se actualizaron los campos a `{{ resource.description }}` y `Category: {{ resource.category.name }}`.
2. **Pruebas automatizadas (`pages/tests.py`):**
   - Se implementó `ClassroomHubTests(TestCase)` con **10 pruebas** que verifican páginas públicas, registros desde la base de datos, categorías relacionadas, enlaces de navegación, error 404, persistencia de cambios, protección `ProtectedError`, catálogo vacío y relación inversa.
   - Resultado de `python manage.py test pages`: **`Ran 10 tests — OK`** sobre `test_classroom_hub`.
