# Imagem base oficial do Node.js (versão LTS leve)
FROM node:18-alpine

# Criar diretório de trabalho dentro do container
WORKDIR /app

# Copiar arquivo de dependências
COPY package.json .

# Instalar dependências
RUN npm install --production

# Copiar código-fonte
COPY server.js .

# Criar grupo e usuário não-root para segurança
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Definir dono dos arquivos para o novo usuário
RUN chown -R appuser:appgroup /app

# Trocar para o usuário não-root
USER appuser

# Expor a porta da aplicação
EXPOSE 3000

# Comando que inicia a aplicação
CMD ["node", "server.js"]