### 1. Configuração (src/config)

Responsabilidade: Contém a configuração do ambiente e a configuração do banco de dados.
Arquivos:
env.ts: Gerencia as variáveis de ambiente usando dotenv.
database.ts: Configura e gerencia a conexão com o banco de dados.

### 2. Modelos (src/models)
Responsabilidade: Define a estrutura dos dados e interage diretamente com o banco de dados.
Arquivos:
user.model.ts: Define o esquema do usuário e os métodos de interação com o banco de dados (CRUD).

### 3. Serviços (src/services)
Responsabilidade: Contém a lógica de negócio da aplicação. É responsável por implementar funcionalidades específicas, como registro e login.
Arquivos:
auth.service.ts: Lida com a autenticação do usuário, incluindo registro, login e geração de tokens JWT.
user.service.ts: Pode incluir funcionalidades relacionadas aos usuários, como busca, atualização e exclusão de perfis.

### 4. Controladores (src/controllers)
Responsabilidade: Recebem as requisições HTTP, processam dados, chamam os serviços necessários e retornam a resposta adequada.
Arquivos:
auth.controller.ts: Recebe e processa as requisições de registro e login, chamando os métodos apropriados do auth.service.
user.controller.ts: Gerencia as operações relacionadas ao usuário, como obter detalhes do usuário, atualizar informações, etc.

### 5. Rotas (src/routes)
Responsabilidade: Define os endpoints da API e mapeia esses endpoints para os controladores correspondentes.
Arquivos:
auth.routes.ts: Define as rotas para autenticação (/register, /login) e mapeia para auth.controller.
user.routes.ts: Define as rotas relacionadas ao usuário e mapeia para user.controller.

### 6. Middlewares (src/middlewares)
Responsabilidade: Contém funções intermediárias que processam requisições antes de chegarem aos controladores ou após saírem dos controladores.
Arquivos:
auth.middleware.ts: Pode conter lógica de verificação de tokens JWT para proteger rotas.
error.middleware.ts: Gerencia erros e formata a resposta de erro para o cliente.

### 7. Utilitários (src/utils)
Responsabilidade: Contém funções auxiliares e utilitárias usadas em diferentes partes da aplicação.
Arquivos:
logger.ts: Implementa um logger usando winston para registrar logs da aplicação.

### 8. Inicialização da Aplicação (src/app.ts e src/server.ts)
Responsabilidade: Configura e inicializa a aplicação Express.
Arquivos:
app.ts: Configura o Express, middlewares globais, rotas e a conexão com o banco de dados.
server.ts: Inicia o servidor HTTP na porta definida.

### 9. Testes (tests)
Responsabilidade: Contém testes para verificar a funcionalidade e a integridade da aplicação.
Arquivos:
auth.test.ts: Testa os endpoints de autenticação (registro e login).
user.test.ts: Testa os endpoints relacionados ao usuário.
Resumo das Responsabilidades por Camada
Configuração: Gerenciamento de variáveis de ambiente e configuração do banco de dados.
Modelos: Estrutura dos dados e interação com o banco de dados.
Serviços: Lógica de negócio.
Controladores: Recepção e processamento de requisições HTTP.
Rotas: Mapeamento de endpoints para controladores.
Middlewares: Processamento intermediário de requisições e respostas.
Utilitários: Funções auxiliares e logs.
Inicialização da Aplicação: Configuração e inicialização do servidor.
Testes: Verificação da funcionalidade da aplicação.
Essa separação em camadas garante uma arquitetura modular e escalável, facilitando a manutenção, testes e evolução da aplicação.