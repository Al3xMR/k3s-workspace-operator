# Imágenes Base de Contenedores (/docker)

Este directorio contiene las definiciones (`Dockerfiles`) de los entornos base que se utilizarán para instanciar los laboratorios de los estudiantes. 

La arquitectura divide los recursos en diferentes **Niveles de Servicio**, asignando el entorno adecuado según la complejidad de la asignatura.

## Estructura de Directorios

```text
docker/
├── nivel1-ttyd/
│   └── Dockerfile      # Entorno CLI ligero (Terminal web)
└── nivel2-codeserver/
    └── Dockerfile      # Entorno de desarrollo IDE (VS Code web)
```

## Niveles de Servicio Implementados

* **Nivel 1 (`ttyd`)**: Entorno de terminal pura accesible desde el navegador. Ideal para materias de fundamentos de programación, bases de datos por CLI o redes básicas. Expone el puerto `7681`.
* **Nivel 2 (`code-server`)**: Entorno visual basado en VS Code. Ideal para materias de desarrollo web, programación orientada a objetos o arquitectura de software. Expone el puerto `8080`.

## Decisiones de Diseño y Seguridad

1. **Aislamiento Rootless:** K3s restringe los contenedores a nivel de nodo físico. Los contenedores inician su proceso principal bajo el usuario `estudiante` (UID > 1000), garantizando que un escape del contenedor no comprometa el servidor anfitrión.
2. **Privilegios Internos (Sudo):** Dentro de su entorno efímero, el estudiante pertenece al grupo `sudo` con la directiva `NOPASSWD`. Esto le otorga total libertad para instalar paquetes, compilar código y modificar el sistema operativo huésped sin requerir la intervención del administrador.
3. **Caché de Repositorios (APT):** A diferencia de las imágenes tradicionales de producción, estos Dockerfiles omiten el borrado del caché de `apt`. Esto incrementa marginalmente el peso de la imagen base (~40MB), pero elimina la necesidad de que el estudiante ejecute `sudo apt-get update` al iniciar su laboratorio, mejorando significativamente la experiencia de usuario (UX).
4. **Punto de Montaje Central:** Todos los entornos definen `/home/estudiante/workspace` como su `WORKDIR`. En el clúster real, Kubernetes montará el volumen persistente (NFS de 500 MB) exactamente en esta ruta.

## Guía Rápida de Compilación Local

Durante la etapa de desarrollo local, utilizamos `podman` para construir las imágenes sin depender de un demonio de Docker.

### 1. Construir las imágenes

```bash
podman build -t localhost/registro-local/base-nivel1-ttyd:latest ./nivel1-ttyd/
podman build -t localhost/registro-local/base-nivel2-codeserver:latest ./nivel2-codeserver/
```

### 2. Probar localmente (Opcional)

```bash
# Nivel 1
podman run --rm -d -p 7681:7681 --name test-ttyd localhost/registro-local/base-nivel1-ttyd:latest

# Nivel 2
podman run --rm -d -p 8080:8080 --name test-codeserver localhost/registro-local/base-nivel2-codeserver:latest
```

### 3. Transferir al clúster K3s

Para que Kubernetes pueda utilizar estas imágenes sin necesidad de subirlas a Docker Hub, se exportan desde Podman y se inyectan directamente en `containerd`:

```bash
podman save localhost/registro-local/base-nivel1-ttyd:latest | sudo k3s ctr images import -
podman save localhost/registro-local/base-nivel2-codeserver:latest | sudo k3s ctr images import -
```

Aquí tienes la sección que debes añadir al final del `README.md` para documentar cómo levantar los Pods en K3s y verificar que todo funcione dentro de Kubernetes.

## 4. Prueba de Ejecución en K3s (Validación Final)

Una vez que las imágenes están importadas en `containerd`, puedes crear un Pod manual en Kubernetes para confirmar que el entorno de desarrollo levanta correctamente dentro del clúster.

**Manifiesto de prueba (`test-pod.yaml`):**
Crea este archivo para obligar a Kubernetes a usar tu imagen local (`imagePullPolicy: Never`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-codeserver
  namespace: lab-estudiante
spec:
  containers:
  - name: workspace
    image: localhost/registro-local/base-nivel2-codeserver:latest
    imagePullPolicy: Never
    ports:
    - containerPort: 8080
```

**Ejecución y Verificación:**

1. **Aplica el manifiesto:**
```bash
kubectl apply -f test-pod.yaml
```


2. **Verifica que el Pod esté en estado Running:**
```bash
kubectl get pods -n lab-estudiante
```


3. **Expón el puerto a tu máquina física (Linux Mint):**
```bash
kubectl port-forward pod/test-codeserver 8080:8080 -n lab-estudiante
```

4. **Prueba el acceso:** Abre `http://localhost:8080` en tu navegador. Deberías ver VS Code Web.
5. **Limpieza:** Presiona `Ctrl+C` en la terminal para detener el port-forwarding y elimina el Pod con `kubectl delete -f test-pod.yaml`.
