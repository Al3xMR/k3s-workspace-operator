FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y curl ca-certificates sudo

RUN curl -fsSL https://code-server.dev/install.sh | sh

# Crear usuario, agregarlo al grupo sudo y permitir sudo sin contraseña
RUN useradd -m -d /home/estudiante -s /bin/bash -G sudo estudiante && \
    echo "estudiante ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER estudiante
WORKDIR /home/estudiante/workspace

EXPOSE 8080

CMD ["code-server", "--bind-addr", "0.0.0.0:8080", "--auth", "none", "--disable-telemetry"]
