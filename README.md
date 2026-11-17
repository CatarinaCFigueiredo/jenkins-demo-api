# Jenkins Demo API — Pipeline CI/CD com FastAPI, Pytest e Docker

API REST desenvolvida em Python com **FastAPI**, acompanhada de testes automatizados (**Pytest** com exportação de relatórios **JUnit XML**), contentorização com boas práticas de segurança (**Docker**) e uma **Pipeline Declarativa Jenkins** completa.

## Arquitetura e Tecnologias

* **Linguagem & Framework:** Python 3.12, FastAPI, Uvicorn
* **Testes & Qualidade:** Pytest, HTTPX, Relatórios JUnit XML
* **CI/CD:** Jenkins (Controller + Agente Linux dedicado via WebSocket)
* **Contentorização:** Docker (`python:3.12-slim` com execução em utilizador não-root)

## Estrutura do Projeto

    jenkins-demo-api/
    ├── app/
    │   ├── __init__.py
    │   └── main.py              # Endpoints GET / e GET /health
    ├── tests/
    │   └── test_main.py         # Testes unitários da API
    ├── requirements.txt         # Dependências de produção
    ├── requirements-dev.txt     # Dependências de desenvolvimento e testes
    ├── Dockerfile               # Imagem da aplicação (USER 10001)
    ├── .dockerignore
    ├── .gitignore
    └── Jenkinsfile              # Pipeline declarativa de CI e Docker Build

## Endpoints Disponíveis

| Método | Rota | Descrição | Resposta Esperada |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | Identificação do serviço | `{"service": "jenkins-demo-api"}` |
| `GET` | `/health` | Verificação de estado (*healthcheck*) | `{"status": "ok"}` |

## Etapas da Pipeline Jenkins (`Jenkinsfile`)

A pipeline executa num agente Linux isolado (`ci-python-docker`) com controlo de concorrência (`disableConcurrentBuilds`), *timestamps* e *timeout* de segurança:

1. **Checkout:** Limpa o workspace e obtém o código-fonte da branch `main`.
2. **Setup:** Cria um ambiente virtual Python (`.venv`) e instala as dependências.
3. **Test:** Executa a suíte `pytest` e publica os resultados no Jenkins via plugin **JUnit** (`reports/junit.xml`).
4. **Docker Preflight:** Valida a disponibilidade do Docker CLI, a ligação ao daemon e a existência do `Dockerfile`.
5. **Docker Build:** Constrói a imagem da API etiquetada com o número do build (`jenkins-demo-api:$BUILD_NUMBER`).
6. **Verify Image:** Inspeciona a imagem gerada para confirmar a sua integridade e identificador SHA-256.

## Execução Local

Correr testes e iniciar a API:

    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements-dev.txt
    pytest -v
    uvicorn app.main:app --host 0.0.0.0 --port 8000

Construir e executar via Docker:

    docker build -t jenkins-demo-api:local .
    docker run -d -p 8000:8000 --name demo-api jenkins-demo-api:local
