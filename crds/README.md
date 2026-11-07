# Contratos de Datos (CRDs) - API de Kubernetes

Este directorio contiene las Definiciones de Recursos Personalizados (CRDs) que extienden la API de K3s. Actúan como el contrato de integración entre el módulo de orquestación del Backend y el Operador de Kubernetes.

## 1. Recurso: `PlantillaCurso` (`crd-plantilla-curso.yaml`)

Se aplica cuando un docente crea o actualiza una asignatura y su paralelo. Ordena al clúster compilar una imagen de contenedor personalizada a partir de la imagen base del nivel seleccionado.

### Campos de Entrada (`spec`)
* **`codigoMateria`** (String, obligatorio): Código institucional de la asignatura (ej. `ICCD654`).
* **`paralelo`** (String, obligatorio): Identificador del grupo (ej. `GR1CC`, `GR1SW`).
* **`nivel`** (Integer, obligatorio): Nivel del entorno base (`1`: ttyd, `2`: code-server, `3`: KasmVNC).
* **`herramientas`** (Array de Strings, opcional): Paquetes del sistema requeridos para la asignatura.

### Campos de Salida (`status`)
* **`fase`**: Estado de la construcción (`Pending`, `Building`, `Ready`, `Failed`).
* **`imagenTag`**: Identificador de la imagen resultante en el registro interno del clúster.

## 2. Recurso: `Laboratorio` (`crd-laboratorio.yaml`)

Se aplica cuando un estudiante solicita iniciar su entorno virtual desde el Dashboard. Instancia un Pod utilizando la imagen precompilada del curso, enlaza el almacenamiento NFS y aplica las políticas de recursos.

### Campos de Entrada (`spec`)
* **`correoInstitucional`** (String, obligatorio): Correo del estudiante validado mediante SSO.
* **`codigoMateria`** (String, obligatorio): Código de la asignatura (ej. `ICCD654`).
* **`paralelo`** (String, obligatorio): Grupo matriculado (ej. `GR1CC`).
* **`nivel`** (Integer, obligatorio): Nivel del entorno (`1`, `2` o `3`).
* **`tiempoInactividadMinutos`** (Integer, opcional): Tiempo máximo sin tráfico de red antes de hibernar el Pod (por defecto `30`).

### Políticas Aplicadas Automáticamente por Infraestructura
* **Almacenamiento Persistente:** Se asigna y monta mediante `subPath` un PVC con cuota estricta de `500Mi` en el servidor NFS por estudiante.
* **Límites de Hardware (`LimitRanges`):** La memoria RAM (`512Mi` para nivel 1, `1Gi` para nivel 2, `2Gi` para nivel 3) y las ráfagas de CPU se asignan automáticamente en función del campo `nivel`.

### Campos de Salida (`status`)
* **`fase`**: Estado del entorno (`Pending`, `Provisioning`, `Running`, `Hibernating`, `Terminating`, `Failed`).
* **`urlAcceso`**: Ruta HTTPS/WSS expuesta por el Ingress Controller para incrustar en el iframe del Workspace.
* **`ultimaActividad`**: Marca de tiempo de la última actividad de red registrada.

## Comandos de Despliegue y Prueba

Registrar los CRDs en el clúster:
```bash
kubectl apply -f crds/crd-plantilla-curso.yaml
kubectl apply -f crds/crd-laboratorio.yaml
```

Crear el namespace aislado y aplicar los manifiestos de prueba:

```bash
kubectl create namespace lab-estudiante
kubectl apply -f crds/ejemplo-plantilla-curso.yaml
kubectl apply -f crds/ejemplo-laboratorio.yaml
```

Consultar el estado de los recursos:

```bash
kubectl get pcurso -n lab-estudiante
kubectl get lab -n lab-estudiante
```
