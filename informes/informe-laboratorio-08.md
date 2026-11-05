# Informe Técnico — Laboratorio 8: Autenticación, Sesiones, Grupos y Permisos en Django

## 1. Enlaces del Repositorio en GitHub

- **Repositorio general (Laboratorios 7, 8 y 9):** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9)
- **Carpeta específica del Laboratorio 8 en GitHub:** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-08](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-08)
- **Ubicación local:** `/home/andres/Descargas/laboratorio-08`

---

## 2. Resumen Ejecutivo

En el **Laboratorio 8** se amplió **ClassroomHub** manteniendo la persistencia en **MariaDB / MySQL** del Laboratorio 7 e incorporando el sistema de **autenticación, sesiones y autorización por grupos y permisos de Django**. Se construyó el flujo de registro de usuarios comunes (`UserCreationForm`), inicio y cierre de sesión seguros (`LoginView` y `LogoutView` mediante `POST` con protección CSRF), un panel privado (`/dashboard/`) con estado en `request.session` y una vista de revisión del catálogo (`/resources/review/`) restringida exclusivamente al grupo `Catalog reviewers` mediante el permiso `pages.view_resource`.

---

## 3. Detalle Paso a Paso de lo Realizado

### Práctica 8.1 — Identidad, contraseñas cifradas y sesiones
1. **Verificación de tablas y middleware:**
   - Se comprobaron las tablas del sistema en MariaDB (`auth_user`, `auth_group`, `auth_permission`, `auth_user_groups`, `auth_group_permissions` y `django_session`) y el orden de `SessionMiddleware`, `CsrfViewMiddleware` y `AuthenticationMiddleware` en `config/settings.py`.
2. **Hashing de contraseñas:**
   - Se verificó en `python manage.py shell` el funcionamiento de `make_password()` y `check_password()`, confirmando que Django nunca almacena contraseñas en texto plano.

### Práctica 8.2 — Cuentas comunes y registro público (`UserCreationForm`)
1. **Creación de usuario de demostración sin privilegios:**
   - Se creó la cuenta `reader_demo` comprobando que `is_staff` e `is_superuser` permanecen en `False`.
2. **Vista `signup` y plantilla `signup.html`:**
   - En `pages/views.py` se implementó `signup(request)` usando `UserCreationForm`, redirigiendo a `pages:dashboard` si el usuario ya está autenticado o a `pages:login` tras un registro válido.
   - Se creó `pages/templates/pages/signup.html` con `{% csrf_token %}`, `{{ form.non_field_errors }}` y `{{ form.as_p }}`, sin exponer campos de privilegios (`is_staff`, `is_superuser`, `groups`).

### Práctica 8.3 — Inicio y cierre de sesión con vistas integradas
1. **Configuración de redirecciones en `config/settings.py`:**
   - Se declararon `LOGIN_URL = "pages:login"`, `LOGIN_REDIRECT_URL = "pages:dashboard"` y `LOGOUT_REDIRECT_URL = "pages:home"`.
2. **Rutas y plantillas de autenticación:**
   - Se registraron en `pages/urls.py` las rutas `/accounts/signup/`, `/accounts/login/` (`LoginView`) y `/accounts/logout/` (`LogoutView`).
   - Se creó `pages/templates/pages/login.html` con soporte para el parámetro seguro `next`.
   - En `pages/templates/pages/base.html` se actualizó la barra de navegación para mostrar `Log in` y `Sign up` a visitantes anónimos y un formulario `POST` con `{% csrf_token %}` para `Log out` a usuarios autenticados (evitando cierres de sesión por `GET`, que devuelven `405 Method Not Allowed`).

### Práctica 8.4 — Panel privado (`/dashboard/`) y estado de sesión
1. **Vista privada y contador en `request.session`:**
   - Se implementó `@login_required def dashboard(request)` en `pages/views.py`, incrementando `request.session["dashboard_visits"]` en cada visita de la sesión activa.
   - Se creó `pages/templates/pages/dashboard.html` mostrando el usuario autenticado (`{{ user.username }}`) y el número de visitas en la sesión (`{{ visit_count }}`).

### Práctica 8.5 — Grupos, permisos y revisión autorizada (`/resources/review/`)
1. **Grupo `Catalog reviewers`:**
   - Se creó el grupo `Catalog reviewers` y se le asignó el permiso `pages.view_resource` asociado al modelo `Resource`.
   - Se asoció el usuario `reader_demo` al grupo, comprobando `has_perm("pages.view_resource") == True`, `has_perm("pages.change_resource") == False` e `is_staff == False`.
2. **Vista restringida `resource_review` y plantilla `review.html`:**
   - Se decoró `resource_review(request)` con `@login_required` y `@permission_required("pages.view_resource", raise_exception=True)` para que un visitante anónimo sea redirigido al login y un usuario autenticado sin permiso reciba **HTTP 403 Forbidden**.
   - Se creó `pages/templates/pages/review.html` y se añadió el enlace condicional `{% if perms.pages.view_resource %}` en `base.html`.

### Práctica 8.6 — Reto integrador y pruebas automatizadas (`pages/tests.py`)
1. **Ampliación de la suite de pruebas:**
   - Se conservaron las **10 pruebas** del Laboratorio 7 (`ClassroomHubTests`) y se agregaron **18 pruebas nuevas** divididas en:
     - `AuthenticationTests` (12 pruebas: renderizado de registro, creación de usuario normal, rechazo de contraseñas distintas o usuario duplicado, login fallido y exitoso, redirección con `next`, protección de `/dashboard/`, incremento del contador de sesión, rechazo `405` en logout por `GET`, cierre de sesión por `POST` y rechazo `403` sin token CSRF).
     - `ResourceReviewPermissionTests` (6 pruebas: redirección anónima en `/resources/review/`, respuesta `403` para usuario común, acceso `200` para miembro de `Catalog reviewers`, verificación de que el revisor no adquiere `is_staff`, catálogo público disponible e imposibilidad de evadir el permiso escribiendo la URL).
   - Resultado de `python manage.py test pages`: **`Ran 28 tests — OK`** sobre `test_classroom_hub`.
