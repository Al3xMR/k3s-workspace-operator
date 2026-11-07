# Contratos de Datos (CRDs) - API de Laboratorios

Este directorio contiene las Definiciones de Recursos Personalizados (CRDs) que extienden la API de K3s. Definen la estructura exacta que el módulo de orquestación del Backend debe enviar para aprovisionar un entorno de desarrollo.

## Estructura del Recurso: `Laboratorio`

Para crear un nuevo entorno, el Backend debe aplicar un manifiesto respetando los siguientes campos obligatorios dentro del bloque `spec`:

* **`correoInstitucional`** (String): Correo del alumno devuelto por la autenticación SSO (Entra ID).
* **`codigoMateria`** (String): Código institucional de la materia (ej. `ICCD654`).
* **`paralelo`** (String): Identificador del grupo (ej. `GR1CC` o `GR2SW`). 
* **`nivel`** (Integer): Define el software a inyectar en el contenedor:
  * `1`: Consola interactiva (ttyd).
  * `2`: IDE Web (code-server).
  * `3`: Escritorio Remoto (KasmVNC).

*(Nota: La cuota de almacenamiento persistente se aplica automáticamente por el Operador de Kubernetes, asignando 500MB en el servidor NFS por cada estudiante.*

## Valores de Retorno (Bloque `status`)

Una vez que el Operador de Kubernetes inicie el aprovisionamiento, actualizará el recurso con estos campos de solo lectura para que el Backend los consuma:

* **`fase`**: Estado actual (ej. `Pending`, `Running`, `Hibernating`, `Terminating`).
* **`urlAcceso`**: URL segura (HTTPS/WSS) generada por el Ingress para incrustar el entorno en el iframe del Workspace.

## Aplicación Manual

```bash
kubectl apply -f crd-laboratorio.yaml
```