## Installation de Ollama avec docker

### Installation de docker

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

verifiaction de l'installation de docker :
```bash
sudo systemctl status docker
```

### mise en place du conteneur Ollama

https://quelllm.fr/guide/ollama-docker-installation-guide

```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```
- `-d` pour detacher le conteneur
- `-v ollama:/root/.ollama` creer un volume nommer ollama qui est situer dans le conteneur `/root/.ollama`
- `-p 11434:11434` publie le port du conteneur `11434` vers le port hote `11434` pour pouvoir utiliser ollama en dehors du conteneur

Pour etre sur que d'utiliser le CPU et no le GPU dans `~/.bashrc`ajouter:
```
export OLLAMA_NO_GPU=1
```

test de ollama:
```bash
docker exec -it ollama ollama -v
# >> ollama version is 0.40.1
```

### Telechargement d'un modele

Nous allons tester plusieurs LLMs :
- llama3.1:8b
- mistral:7b-instruct
- phi3:mini

#### test llama3.1:8b
```bash
docker exec -it ollama ollama pull llama3.1:8b
```
ou
```bash
docker exec -it ollama ollama run llama3.1:8b # lance le modele et l'installe si besoin
```

## Installation de WebUI

docker compose:
```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://127.0.0.1:11434
    volumes:
      - open-webui:/app/backend/data

volumes:
  open-webui:
```

```bash
docker compose up -d
```
