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

test d'un prompt simple:
```shell
toto@toto:~/SAE_chatbot$ sudo docker exec -ti ollama ollama run llama3.1:8b --verbose "create a password with 12 characters, 2 special characters, 1 number and at least one uppercase letter"
Here is a password that meets the requirements:

`G4tL#eK$rM@8d`

Here's a breakdown of the password:

* 12 characters: `G4tL#eK$rM@8d` is 12 characters long
* 2 special characters: `#` and `@` are special characters
* 1 number: `4` and `8` are numbers
* At least one uppercase letter: `G`, `K`, and `M` are uppercase letters

Note: I've used a mix of uppercase and lowercase letters to make the password easier to read, but you can modify it to your liking.

total duration:       27.652345015s
load duration:        3.177447048s
prompt eval count:    32 token(s)
prompt eval duration: 4.215865s
prompt eval rate:     7.59 tokens/s
eval count:           139 token(s)
eval duration:        20.238527s
eval rate:            6.87 tokens/s
```
- `--verbose` permet d'afficher un rapport plus detailler de la reponse

## Installation de Opencode

installation avec curl :
```bash
curl -fsSL https://opencode.ai/install | bash
```
lancement de opencode via terminal:
```bash
opencode
```

### Creation d'un model

On creer un model avec un fichier `ModelFile`.

exemple utiliser:
```
FROM llama3.1:8b

# --- OPTIMISATIONS LOCALES ET SAMPLING ---
PARAMETER temperature 0.1
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1

# Fenêtre de contexte étendue à 8192 tokens (2048 par défaut)
PARAMETER num_ctx 8192

# --- SYSTEM PROMPT ---
SYSTEM """
Tu es DevCode-8B, un ingénieur logiciel senior spécialisé dans l'écriture de code propre, performant et immédiatement exécutable.

Règles de réponse :
1. Sois direct. Pas de politesses superflues ("Bien sûr", "Voici le code"). Donne directement la solution ou le bloc de code.
2. Écris du code production-ready : gestion des erreurs, typage explicite (Python type hints, TypeScript, etc.) et respect des conventions du langage.
3. Utilise les meilleures pratiques de développement (Clean Code, DRY, SOLID).
4. Pose des questions de clarification si la demande est ambiguë.
"""
```

Creation du modele a partir du ModelFile
```bash
ollama create coder -f ModelFile
```

Pour verifier les parametres qui ont ete mis dans le `ModelFile`
```bash
sudo docker exec -it ollama ollama run coder
>>> /show parameters
```
### Configurer LLMs local sur opencode

```bash
vim ~/.config/opencode/opencode.json
```
exemple avec llama3.1:8b :
```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama",
      "options": {
        "baseURL": "http://127.0.0.1:11434/v1"
      },
      "models": {
	    "llama3.1:8b": { "name": "llama3.1:8b" },
	    "mistral:7b-instruct": { "name": "mistral:7b-instruct" },
	    "phi3:mini": { "name": "phi3:mini"},
        "qwen2.5:14b": { "name": "qwen2.5:14b" }
      }
    }
  },
  "model": "ollama/qwen2.5:14b"
}
```
## Installation de WebUI

docker compose:
```yaml
services:
	open-webui:
		image: ghcr.io/open-webui/open-webui:main
		container_name: open-webui
		ports:
			- "3000:8080"
		volumes:
			- open-webui:/app/backend/data
		environment:
			- OLLAMA_BASE_URL=http://host.docker.internal
		extra-hosts:
			- "host.docker.internal:host-gateway"
		restart: always

volumes:
	open-webui:
```
- `host.docker.internal` permet de communiquer directement sur le pc hotes et donc recuperer les models de ollama dans l'autre conteneur
- `host-gateway` permet d'identifier l'adresse IP du pc Hote

```bash
docker compose up -d
```

### Configuration de WebUI

Deja il faut creer un compte et se connecter.

Une fois connecter  dans `settings > AI > Connections` changer l'ip dans `Manage Ollama API Connections` :
`http://<ip_pc_hote>:11434`

Une fois ca fait vous pouvez utiliser les models de Ollama en local.


## Utilisation de Bandit

Bandit va permettre d'analyser le code que le model IA nous a creer avec les commandes suivantes:
```shell
pip bandit # installer bandit
bandit -r main.py
```

resultat:
```shell
[main]    INFO    profile include tests: None
[main]    INFO    profile exclude tests: None
[main]    INFO    cli include tests: None
[main]    INFO    cli exclude tests: None
[main]    INFO    running on Python 3.13.5
Run started:2026-10-13 09:25:11.174319+00:00

Test results:
    No issues identified.

Code scanned:
    Total lines of code: 33
    Total lines skipped (#nosec): 0

Run metrics:
    Total issues (by severity):
        Undefined: 0
        Low: 0
        Medium: 0
        High: 0
    Total issues (by confidence):
        Undefined: 0
        Low: 0
        Medium: 0
        High: 0
Files skipped (0):
```

