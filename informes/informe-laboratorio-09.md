# Informe Técnico — Laboratorio 9: Formularios y CRUD con Django (Vistas, ModelForm y Permisos)

## 1. Enlaces del Repositorio en GitHub

- **Repositorio general (Laboratorios 7, 8 y 9):** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9)
- **Carpeta específica del Laboratorio 9 en GitHub:** [https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-09](https://github.com/andressantiagomeloaguirre67-sys/Laboratorios-7-8-9/tree/main/laboratorio-09)
- **Ubicación local:** `/home/andres/Descargas/laboratorio-09`

---

## 2. Resumen Ejecutivo

En el **Laboratorio 9** se completó la evolución de **ClassroomHub** incorporando **operaciones CRUD (creación, consulta de detalle, actualización y eliminación) de recursos** desde la propia interfaz web de la aplicación. Se construyó un formulario reutilizable `ResourceForm` (`ModelForm`) con validación personalizada, se implementaron vistas genéricas basadas en clases (`DetailView`, `CreateView`, `UpdateView`, `DeleteView`) protegidas en el servidor con `LoginRequiredMixin` y `PermissionRequiredMixin`, y se separaron los roles entre lectura (`Catalog reviewers`) y edición (`Catalog editors`).

---

## 3. Detalle Paso a Paso de lo Realizado

### Práctica 9.1 — Formularios, métodos GET/POST, CSRF y patrón Post/Redirect/Get (PRG)
1. **Validación y normalización en el servidor:**
   - Se verificó el ciclo de formularios bound/unbound en `python manage.py shell` (`is_valid()`, `cleaned_data` y `errors`), comprobando el uso obligatorio de `{% csrf_token %}` en solicitudes `POST` y la redirección posterior al guardado (`success_url`) para evitar reenvíos accidentales.

### Práctica 9.2 — Construcción de `ResourceForm` (`pages/forms.py`)
1. **Definición explícita de campos y widgets:**
   - Se creó `pages/forms.py` definiendo `ResourceForm(forms.ModelForm)` asociado al modelo `Resource` con la lista explícita `fields = ["title", "description", "resource_type", "category", "available"]` (excluyendo `id` y `created_at`) y `Textarea(attrs={"rows": 4})` para `description`.
2. **Validación personalizada `clean_title()`:**
   - Se implementó `clean_title(self)` aplicando `.strip()` y exigiendo al menos 3 caracteres (`ValidationError("Title must contain at least 3 characters.")`).

### Práctica 9.3 — Consulta de detalle público y creación protegida de recursos
1. **Vista de detalle público (`ResourceDetailView`):**
   - Se implementó `ResourceDetailView(DetailView)` en `pages/views.py` sobre la ruta `resources/<int:pk>/` (`name="resource-detail"`) y se creó `pages/templates/pages/resource_detail.html`, enlazando cada título en `resources.html` hacia su página de detalle.
2. **Vista de creación (`ResourceCreateView`) y grupo `Catalog editors`:**
   - Se implementó `ResourceCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView)` en `resources/new/` (`name="resource-create"`) exigiendo el permiso `pages.add_resource` y reutilizando `pages/templates/pages/resource_form.html`.
   - Se creó el grupo `Catalog editors` con los permisos `view_resource`, `add_resource`, `change_resource` y `delete_resource`, y se asoció la cuenta de prueba `editor_demo` (sin otorgar `is_staff`).

### Práctica 9.4 — Actualización de recursos (`ResourceUpdateView`)
1. **Edición reutilizando formulario y plantilla:**
   - Se implementó `ResourceUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView)` sobre la ruta `resources/<int:pk>/edit/` (`name="resource-update"`), exigiendo el permiso `pages.change_resource`, reutilizando `ResourceForm` y `resource_form.html`, y conservando la clave primaria (`pk`) del registro modificado.

### Práctica 9.5 — Eliminación controlada mediante confirmación POST (`ResourceDeleteView`)
1. **Confirmación en `GET` y borrado en `POST`:**
   - Se implementó `ResourceDeleteView(LoginRequiredMixin, PermissionRequiredMixin, DeleteView)` en `resources/<int:pk>/delete/` (`name="resource-delete"`) exigiendo `pages.delete_resource`.
   - Se creó `pages/templates/pages/resource_confirm_delete.html` para que una solicitud `GET` únicamente muestre la pantalla de confirmación sin borrar el registro, y solo la solicitud `POST` con `{% csrf_token %}` ejecute la eliminación.

### Práctica 9.6 — Reto integrador y suite completa de pruebas (`pages/tests.py`)
1. **Actualización de navegación y plantillas:**
   - Se integraron los enlaces condicionales en `base.html`, `resources.html` (`{% if perms.pages.add_resource %}`) y `resource_detail.html` (`{% if perms.pages.change_resource %}`, `{% if perms.pages.delete_resource %}`).
2. **Pruebas automatizadas de CRUD y permisos (`ResourceCrudTests`):**
   - Se conservaron las 28 pruebas de los Laboratorios 7 y 8 y se añadió la clase `ResourceCrudTests` cubriendo detalle público, 404 en recursos inexistentes, redirección a login para anónimos en crear/editar/borrar, rechazo `403` para lectores y revisores (`Catalog reviewers`), creación/edición/eliminación por editores (`Catalog editors`), rechazo de títulos cortos (`"ab"`) y categorías inexistentes, preservación de `pk` al actualizar, confirmación `GET` sin borrado y rechazo `403` ante `POST` sin CSRF.
   - Resultado de `python manage.py test pages`: **`Ran 58 tests — OK`** sobre la base aislada `test_classroom_hub`.
