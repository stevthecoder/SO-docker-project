🐳 Projeto Docker — Sistemas Operacionais:

Aplicação Node.js containerizada com MySQL, desenvolvida como projeto prático da disciplina de Sistemas Operacionais.

Aluno: Estevão Uber Góes


Sobre o Projeto:

Este projeto demonstra na prática conceitos fundamentais de Sistemas Operacionais através do Docker: isolamento de processos via namespaces, gerenciamento de recursos com cgroups, sistema de arquivos em camadas (OverlayFS) e segurança com princípio do menor privilégio.

A infraestrutura consiste em dois serviços:


so-web-app — servidor HTTP Node.js com Express, expondo informações do processo e do sistema
so-mysql-db — banco de dados MySQL 8.0 com volume persistente



Estrutura do Projeto:

projeto-docker/
├── app/
│   ├── Dockerfile
│   ├── server.js
│   └── package.json
├── docker-compose.yml
└── README.md


Pré-requisitos:


Docker 20.10+
Docker Compose v2+


Como Executar:

Subir todos os serviços:

bashdocker compose up -d --build

Verificar status dos containers:

bashdocker ps

Ver logs da aplicação:

bashdocker logs so-web-app

Monitorar uso de recursos em tempo real:

bashdocker stats

Parar e remover os containers:

bashdocker compose down

Remover também o volume do banco de dados:

bashdocker compose down -v


Endpoints da API:

Com os containers em execução, acesse:

RotaDescriçãoExemplo de respostaGET /Informações gerais do containerdisciplina, aluno, hostname, plataforma, arquiteturaGET /infoInformações do processo Node.jspid, uptime, cpus

Exemplos:

bash# Rota principal
curl http://localhost:3000/

# Informações do processo
curl http://localhost:3000/info

Resposta esperada em GET /:

json{
  "disciplina": "Sistemas Operacionais",
  "aluno": "Estevão Uber Góes",
  "hostname": "846a907f6f2e",
  "plataforma": "linux",
  "arquitetura": "x64"
}

Resposta esperada em GET /info:

json{
  "pid": 1,
  "uptime": 120,
  "cpus": 4
}


Arquitetura:

Host (Windows/Linux/macOS)
│
└── Docker Engine
    │
    ├── so-network (bridge)
    │   │
    │   ├── so-web-app (Node.js)  ←── porta 3000 exposta ao host
    │   │   └── imagem: projeto-docker-web
    │   │       base: node:18-alpine
    │   │       usuário: appuser (não-root)
    │   │       mem_limit: 128MB
    │   │
    │   └── so-mysql-db (MySQL 8.0)  ←── porta NÃO exposta ao host
    │       mem_limit: 128MB
    │
    └── so-db-volume (volume persistente → /var/lib/mysql)


Detalhes do Dockerfile:

dockerfileFROM node:18-alpine        # imagem base leve (Alpine Linux)
WORKDIR /app
COPY package.json .
RUN npm install --production
COPY server.js .
RUN addgroup -S appgroup && adduser -S appuser -G appgroup  # usuário não-root
RUN chown -R appuser:appgroup /app
USER appuser               # princípio do menor privilégio
EXPOSE 3000
CMD ["node", "server.js"]


Recursos Configurados:

ServiçoImagemLimite de RAMRedePortaso-web-appnode:18-alpine (build local)128 MBso-network3000:3000so-mysql-dbmysql:8.0128 MBso-network— (interna)


Conceitos de SO Demonstrados:

1. Processos e Isolamento (Namespaces)
O Node.js executa como PID 1 dentro do namespace de processos do container — completamente isolado dos demais processos do host.

2. Gerenciamento de Recursos (cgroups)
O mem_limit: 128m utiliza Control Groups (cgroups) do kernel Linux para limitar a memória de cada container, impedindo que um processo monopolize os recursos do sistema.

3. Sistema de Arquivos em Camadas (OverlayFS)
As imagens Docker usam OverlayFS: cada instrução do Dockerfile gera uma camada imutável e reutilizável, confirmado pelo campo "Driver": "overlayfs" no docker inspect.

4. Segurança — Princípio do Menor Privilégio
A aplicação roda como appuser (não-root). Se um atacante explorar uma vulnerabilidade, terá apenas os privilégios desse usuário restrito.

5. Isolamento de Rede
A rede so-network (modo bridge) cria um segmento isolado. O banco de dados não é acessível externamente — apenas o serviço web se comunica com ele pelo hostname db.

6. Persistência de Dados (Volume)
O volume so-db-volume garante que os dados do MySQL sobrevivam à remoção e recriação do container.


Comandos Úteis de Diagnóstico:

bash# Inspecionar configuração completa do container
docker inspect so-web-app

# Executar shell dentro do container
docker exec -it so-web-app sh 

# Ver uso de recursos
docker stats --no-stream

# Ver processos dentro do container
docker exec so-web-app ps aux