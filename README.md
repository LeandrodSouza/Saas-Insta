Aqui está o plano de desenvolvimento completo, estruturado em Pull Requests, abrangendo todo o escopo do projeto que definimos.
Plano de Desenvolvimento Detalhado: Micro-Entregas (Pull Requests)

 - Docker-first
 - Arquitetura limpa
 - Clean Code
 - Don't Repeat Yourself

Épico 1: Fundação do Projeto e Ambiente
PR #1: feat(project): Configuração inicial do ambiente Docker e monorepo
Objetivo: Estabelecer a estrutura de pastas e a orquestração de containers que servirão de base para todo o projeto.
Tarefas:
Criar a estrutura de diretórios raiz: backend/, frontend/.
Implementar o arquivo docker-compose.yml na raiz, definindo os três serviços: backend, frontend e db (PostgreSQL).
Criar um Dockerfile genérico para o serviço backend (baseado em Node.js).
Criar um Dockerfile genérico para o serviço frontend (baseado em Node.js).
Criar um Dockerfile genérico para o serviço bd.
Adicionar um arquivo .env.example na raiz, listando todas as variáveis de ambiente necessárias para os três serviços, conforme a documentação.
Critérios de Aceitação:
O comando docker-compose up deve iniciar os três containers sem erros.
As portas definidas (ex: 3000, 5173, 5432) devem ser mapeadas corretamente para o host.
A estrutura de pastas do projeto está criada.
PR #2: feat(backend): Integração com Prisma e definição do schema do banco de dados
Objetivo: Definir a estrutura completa do banco de dados e configurar o Prisma para gerenciá-la.
Tarefas:
Adicionar o Prisma como dependência no projeto backend.
Inicializar o Prisma, criando a pasta prisma/.
Preencher o arquivo prisma/schema.prisma com todas as tabelas, campos, tipos e relacionamentos definidos na documentação mestre (distribuidoras, usuarios, contas_instagram, conversas, mensagens).
Gerar o cliente Prisma pela primeira vez.
Critérios de Aceitação:
O comando npx prisma validate deve passar sem erros.
O comando npx prisma migrate dev --name init deve criar a primeira migração e aplicar o schema com sucesso no banco de dados que está rodando no container Docker.
Épico 2: Autenticação e Onboarding do Cliente
PR #3: feat(backend): Implementação da autenticação via Google e geração de JWT
Objetivo: Permitir que um novo usuário se autentique na plataforma usando sua conta do Google.
Tarefas:
Criar um módulo de autenticação (auth) no NestJS.
Implementar os endpoints GET /auth/google e GET /auth/google/callback.
Configurar a estratégia Passport.js para o Google OAuth2.
Na lógica do callback, verificar se o usuário do Google já existe na tabela usuarios. Se não, criar um novo registro.
Gerar um token JWT contendo o id do usuário e o distribuidora_id (se já existir) e retorná-lo ao frontend.
Critérios de Aceitação:
Um usuário consegue completar o ciclo de login com Google e a aplicação backend o identifica corretamente, criando um novo registro de usuário se necessário.
PR #4: feat(frontend): Criação das telas de Login, Onboarding e Modo de Espera
Objetivo: Construir a interface inicial que o usuário verá, desde o login até a aprovação da conta.
Tarefas:
Criar uma página de Login com um único botão "Entrar com Google" que redireciona para o endpoint GET /auth/google do backend.
Criar uma página de Onboarding com o formulário para (CNPJ, Razão Social, Endereço).
Implementar a lógica de redirecionamento: após o login, se o usuário não tiver uma distribuidora associada, ele é levado para a página de Onboarding.
Criar a tela do Dashboard em "Modo de Espera", que é exibida após o envio do formulário de onboarding.
Critérios de Aceitação:
O usuário consegue se logar. Um novo usuário é forçado a preencher o formulário. Após o envio, ele vê a tela de "Aguardando Aprovação".
PR #5: feat(backend): Implementação do endpoint de Onboarding e validação de CNPJ
Objetivo: Processar e validar os dados da empresa enviados pelo novo cliente.
Tarefas:
Criar o endpoint POST /onboarding/complete.
A lógica deve ser protegida, exigindo um JWT válido.
Implementar a validação para garantir que o CNPJ enviado não exista no banco de dados. Se existir, retornar um erro 409 (Conflict) com a mensagem de erro segura.
Se o CNPJ for único, criar uma nova distribuidora, associá-la ao usuario logado e definir seu status como AGUARDANDO_APROVACAO.
Critérios de Aceitação:
O formulário do frontend consegue enviar os dados para este endpoint. A validação de CNPJ funciona. A distribuidora é criada corretamente no banco.
Épico 3: Gestão do Administrador e Conexão com Instagram
PR #6: feat(backend): Implementação das rotas de gestão para o Admin
Objetivo: Dar ao administrador as ferramentas mínimas para gerenciar o ciclo de vida dos clientes.
Tarefas:
Criar um módulo de administração (admin) no NestJS.
Implementar uma rota GET /admin/pending-approvals que lista todas as distribuidoras com status AGUARDANDO_APROVACAO.
Implementar uma rota POST /admin/approve/:distribuidoraId que altera o status para ATIVO, define a data_expiracao e gera o api_token.
Proteger essas rotas com um Guard que verifica se o usuário logado tem um papel de administrador global.
Critérios de Aceitação:
O administrador pode, através de uma ferramenta de API, listar e aprovar novos clientes com sucesso.
PR #7: feat(backend): Implementação do fluxo OAuth 2.0 do Instagram
Objetivo: Permitir que um cliente aprovado conecte sua conta do Instagram à plataforma.
Tarefas:
Criar um módulo (instagram) no NestJS.
Implementar os endpoints GET /instagram/auth e GET /instagram/auth/callback.
A lógica do callback deve realizar a troca de códigos por tokens de longa duração.
Criar um serviço de criptografia (AES-256) e usá-lo para criptografar o access_token antes de salvá-lo na tabela contas_instagram.
Critérios de Aceitação:
Um cliente com status ATIVO consegue autorizar o app da Meta, e o sistema armazena o token criptografado e o instagram_page_id corretamente.
PR #8: feat(frontend): Implementação da lógica de conexão pós-aprovação
Objetivo: Atualizar a interface do cliente para permitir a conexão com o Instagram após a aprovação da sua conta.
Tarefas:
No dashboard do cliente, fazer uma chamada para um endpoint que retorna o status da sua distribuidora.
Se o status for ATIVO e a conta do Instagram ainda não estiver conectada, exibir o card e o botão "Conectar com Instagram".
O botão deve redirecionar para o endpoint GET /instagram/auth do backend.
Critérios de Aceitação:
A interface do cliente se adapta dinamicamente ao status da conta, mostrando o botão de conexão apenas no momento certo.
Épico 4: Coleta e Visualização de Dados
PR #9: feat(backend): Implementação do Webhook para recebimento de mensagens
Objetivo: Capturar em tempo real as DMs enviadas para as contas conectadas e roteá-las para o tenant correto.
Tarefas:
Criar um módulo de webhook (webhook) no NestJS.
Implementar o endpoint GET /webhook para responder ao desafio de verificação da Meta.
Implementar o endpoint POST /webhook para processar os eventos de mensagem.
A lógica do POST deve extrair o instagram_page_id, identificar a distribuidora_id correspondente, e salvar os dados nas tabelas conversas e mensagens.
Critérios de Aceitação:
O webhook passa na verificação da Meta. Mensagens enviadas para uma conta conectada são salvas corretamente no banco de dados, associadas à distribuidora correta.
PR #10: feat(backend): Implementação dos endpoints de analytics (Cliente e Admin)
Objetivo: Criar as fontes de dados para os dashboards.
Tarefas:
Criar o endpoint GET /dashboard/analytics que retorna métricas (ex: contagem de conversas) filtradas estritamente pelo distribuidora_id do cliente logado.
Criar os endpoints GET /admin/analytics/summary e GET /admin/analytics/:distribuidoraId para o dashboard "God Mode" do admin.
Critérios de Aceitação:
O cliente só consegue acessar seus próprios dados. O admin consegue acessar tanto os dados agregados quanto os dados de um cliente específico.
PR #11: feat(frontend): Implementação do Dashboard Analítico do Cliente
Objetivo: Apresentar os dados de analytics de forma visual para o cliente.
Tarefas:
Criar os componentes de UI para os KPIs, gráficos e tabelas.
Fazer a chamada ao endpoint GET /dashboard/analytics e popular os componentes com os dados recebidos.
Adicionar um filtro de período (ex: últimos 7 dias, último mês).
Critérios de Aceitação:
O cliente logado consegue ver seu dashboard com gráficos e métricas da sua própria conta.
PR #12: feat(frontend): Implementação da Caixa de Entrada Global do Admin
Objetivo: Fornecer ao administrador a ferramenta para operar as conversas.
Tarefas:
Criar a interface de chat de dois painéis no portal do admin.
Adicionar um dropdown para filtrar as conversas por distribuidora.
Implementar a lógica para buscar e exibir as conversas e mensagens.
Implementar a funcionalidade de envio de mensagens, que fará uma chamada ao endpoint POST /admin/conversations/:id/messages.
Critérios de Aceitação:
O administrador consegue visualizar e responder a mensagens de diferentes clientes a partir de uma única interface.
