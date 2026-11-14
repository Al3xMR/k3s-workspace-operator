FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y ttyd curl ca-certificates sudo

# Crear usuario, agregarlo al grupo sudo y permitir sudo sin contraseña
RUN useradd -m -d /home/estudiante -s /bin/bash -G sudo estudiante && \
    echo "estudiante ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER estudiante
WORKDIR /home/estudiante/workspace

EXPOSE 7681

CMD ["ttyd", "-W", "bash"]
