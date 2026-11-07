# Infraestructura de Plataforma de Laboratorios Virtuales sobre Kubernetes

Este repositorio aloja la Infraestructura como Código (IaC) para el componente de backend e infraestructura del proyecto de titulación: "Diseño e implementación de una plataforma web de autoservicio para el aprovisionamiento de entornos de desarrollo basados en contenedores sobre Kubernetes". El proyecto se desarrolla para la Facultad de Ingeniería de Sistemas de la Escuela Politécnica Nacional (EPN).

## Arquitectura del Clúster

La topología de infraestructura está diseñada mediante el uso de tecnologías optimizadas:

*   **Orquestador:** K3s, seleccionado por su bajo consumo de recursos y soporte para contenedores rootless, lo que incrementa la seguridad del entorno al ejecutarse sin privilegios de administrador real.
*   **Red (CNI):** Cilium, implementado para reemplazar el CNI por defecto (Flannel) con el objetivo de gestionar el enrutamiento avanzado y aplicar reglas de firewalling y aislamiento de seguridad.
*   **Almacenamiento Persistente:** Servidor NFS acoplado con Kubernetes mediante un provisionador dinámico (`nfs-subdir-external-provisioner`) y PVCs. Esto garantiza una cuota estricta (aprox. 500MB por estudiante) que persiste independientemente de la volatilidad de los contenedores efímeros.
*   **Exposición de Servicios:** Ingress Controller configurado para enrutar el tráfico mediante HTTPS y Secure WebSockets (WSS). Este enfoque asegura que la comunicación entre el navegador y el contenedor no sea bloqueada por los firewalls institucionales.

## Entornos Soportados

Los laboratorios provisionados soportan tres niveles de interacción, montados dentro del portal web a través de iframes:

1.  **Nivel 1:** Consola interactiva a través de `ttyd`.
2.  **Nivel 2:** Entorno de desarrollo (IDE) vía `code-server`.
3.  **Nivel 3:** Escritorio remoto ligero provisto por `KasmVNC`.

## Estructura del Repositorio

*   `/crds`: Contratos de datos y Definiciones de Recursos Personalizados (Custom Resource Definitions) que modelan la estructura de un laboratorio virtual dentro de la API de Kubernetes.
*   `/docker`: Archivos Dockerfile e instrucciones para compilar las imágenes optimizadas de los tres niveles de entornos.
*   `/operator`: Código fuente del Operador de Kubernetes. Este servicio vigila la creación de los CRDs y automatiza el ciclo de vida completo de los laboratorios (Pods, Services, PVCs).
*   `/cluster-config`: Manifiestos YAML estáticos para aplicar controles de recursos (LimitRanges), el provisionador de almacenamiento NFS y configuraciones globales.

<!--
## Despliegue Inicial

*(Nota: Esta sección se actualizará progresivamente con los comandos detallados para permitir la reinstalación del clúster desde cero por parte de cualquier docente)*

Para aplicar los contratos base (CRDs) iniciales de la arquitectura en un clúster local:

```bash
kubectl apply -f crds/

```
-->
---

**Autor:** Kevin Alexis Martínez Rueda | [@Al3xMR](https://github.com/Al3xMR)


