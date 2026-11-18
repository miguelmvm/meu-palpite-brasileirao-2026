# 🏆 Meu Palpite — Bolão do Brasileirão 2026

Plataforma web interativa e responsiva para gerenciamento e disputa do **Bolão do Campeonato Brasileiro Série A**. O sistema permite que amigos e participantes registrem seus palpites rodada a rodada, acompanhem estatísticas e classificação geral em tempo real, além de contar com um painel administrativo completo para controle das 38 rodadas, partidas e resultados oficiais.

---

## 📋 Índice

- [Visão Geral e Escopo](#-visão-geral-e-escopo)
- [Funcionalidades para Usuários (Participantes)](#-funcionalidades-para-usuários-participantes)
- [Funcionalidades para Administradores](#-funcionalidades-para-administradores)
- [Regras de Pontuação](#-regras-de-pontuação)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Modelo de Dados (Cloud Firestore)](#-modelo-de-dados-cloud-firestore)
- [Configuração e Execução Local](#-configuração-e-execução-local)
- [Como Definir um Administrador](#-como-definir-um-administrador)
- [Deploy](#-deploy)

---

## 🎯 Visão Geral e Escopo

O **Meu Palpite — Brasileirão 2026** foi projetado para oferecer uma experiência fluida tanto no desktop quanto em dispositivos móveis (com navegação inferior dedicada no mobile). O sistema opera sob controle de acesso baseado em perfis (**RBAC** via Firebase Auth + Cloud Firestore), dividindo a experiência em dois módulos principais:

1. **Módulo do Participante (`user`)**: Focado na usabilidade rápida para envio de palpites antes do fechamento da rodada, acompanhamento de progresso e visualização de desempenho individual e coletivo no ranking.
2. **Módulo Administrativo (`admin`)**: Focado na operação do campeonato — abertura e fechamento de rodadas, montagem da tabela de jogos com escudos oficiais dos clubes e apuração de resultados.

---

## 👤 Funcionalidades para Usuários (Participantes)

### 🔐 Autenticação e Conta (`/auth`, `/forgot-password`, `/reset-password`)
- **Cadastro Validado**: Criação de conta com validação via **Zod** exigindo nome completo, `@nickname` exclusivo (letras, números e `_`), e-mail válido e senha forte (mínimo de 8 caracteres contendo letras e números).
- **Login Seguro**: Autenticação com e-mail e senha integrada ao Firebase Authentication.
- **Recuperação de Senha**: Fluxo completo de envio de link de redefinição por e-mail e validação de código (`oobCode`) para cadastro de nova senha.

### 🏠 Dashboard Interativo (`/`)
- **Resumo Rápido (Quick Stats)**:
  - **Sua Posição**: Colocação atual do participante no ranking geral e total de pontos acumulados.
  - **Próxima Rodada**: Identificação da rodada ativa e seu status atual (`aguardando`, `em andamento` ou `finalizada`).
  - **Progresso de Palpites**: Contador e barra de progresso visual indicando quantos jogos da rodada já foram palpitados (`feitos / total`).
- **Top 5 Ranking**: Prévia dos 5 primeiros colocados do bolão com medalhas (`🥇`, `🥈`, `🥉`) e destaque visual automático para o card do próprio usuário.
- **Próximos Jogos**: Lista dos próximos confrontos agendados com data e horário formatados (`dd/MM HH:mm`).
- **Resumo das Regras**: Card informativo sempre visível com a tabela de pontuação.

### 🎯 Central de Palpites (`/palpites`)
- **Listagem de Confrontos da Rodada**: Exibição dos jogos com escudos dos clubes (Mandante vs Visitante), data/hora e status de preenchimento (`Completo`).
- **Dupla Seleção Intuitiva**:
  - Botões rápidos para cravar o **Resultado** (Vitória do Mandante, Empate ou Vitória do Visitante).
  - Campos numéricos para informar o **Placar Exato** da partida.
- **Controle de Prazo Automático**: Exibição do horário limite de fechamento da rodada (`data_fechamento`) e bloqueio automático de novos palpites ou edições após o encerramento do prazo.
- **Barra de Progresso e Salvamento Fixo**: Acompanhamento em tempo real dos palpites preenchidos e botão flutuante para salvar todos os palpites da rodada.

### 📊 Ranking Geral e Estatísticas (`/ranking`)
- **Indicadores de Desempenho Pessoal**:
  - **Posição Atual** na classificação geral.
  - **Total de Pontos** acumulados e quantidade total de palpites enviados.
  - **Taxa de Acerto (%)** calculada automaticamente com discriminação entre acertos de resultado e acertos de placar exato.
- **Tabela de Classificação Completa**:
  - Destaque especial com ícones e gradientes de Ouro, Prata e Bronze para o **Top 3**.
  - Colunas detalhando Posição, Participante (`@nickname`), Pontos Totais, Acertos de Resultado e Acertos de Placar Exato.
  - Layout adaptativo para telas móveis e desktops.

---

## 🛠️ Funcionalidades para Administradores

O **Painel Administrativo (`/admin`)** é protegido pelo componente `ProtectedRoute` com a flag `requireAdmin` e monitora permissões em tempo real via `onSnapshot` no Firestore. Usuários sem privilégio de administrador são redirecionados automaticamente.

### 📅 1. Gestão de Rodadas e Jogos (Aba `Rodadas & Jogos`)
- **Gerenciamento das 38 Rodadas**:
  - Criação de novas rodadas (numeradas de `1` a `38`) e edição de rodadas existentes.
  - Controle de **Status da Rodada**:
    - `aguardando`: Rodada criada, aguardando início.
    - `em_andamento`: Rodada ativa no campeonato.
    - `finalizada`: Rodada encerrada e apurada.
  - Configuração da **Janela de Palpites**: Definição de `Data início` (abertura dos palpites) e `Data fechamento` (encerramento automático dos palpites).
- **Cadastro e Edição de Partidas**:
  - Adição de jogos vinculados à rodada selecionada utilizando a lista oficial de clubes da Série A com escudos pré-configurados.
  - Validação automática que impede selecionar o mesmo clube como mandante e visitante.
  - Edição rápida de confrontos já cadastrados (alteração de mandante, visitante e status individual do jogo: `agendado`, `ao_vivo`, `finalizado` ou `cancelado`).
  - Exclusão de partidas cadastradas incorretamente.

### ⚽ 2. Lançamento de Resultados Oficiais (Aba `Inserir Resultados`)
- **Apuração por Rodada**:
  - Seletor rápido de rodada para carregar todos os jogos vinculados.
  - Inserção direta dos gols do mandante e do visitante para cada confronto.
  - Ao salvar o placar oficial de uma partida, o status do jogo é atualizado automaticamente para **`finalizado`**, servindo de base para a contabilização do ranking.

---

## 📐 Regras de Pontuação

| Critério | Descrição | Pontuação |
| :--- | :--- | :---: |
| 🎯 **Resultado + Placar Exato** | Acertou o vencedor (ou empate) **e** o número exato de gols de ambos os times | **5 pontos** |
| ✅ **Apenas Resultado** | Acertou quem venceu a partida (ou que houve empate), mas errou o placar exato | **3 pontos** |
| ❌ **Erro de Resultado** | Não acertou o vencedor nem o empate da partida | **0 pontos** |

> **Nota sobre prazos:** Os palpites ficam abertos apenas até a data/hora de fechamento configurada para a rodada ou início da partida.

---

## 🚀 Tecnologias Utilizadas

- **Front-end & Build**:
  - [React 18](https://react.dev/) + [TypeScript](https://www.typescriptlang.org/)
  - [Vite](https://vitejs.dev/) (com plugin React SWC)
  - [React Router DOM v6](https://reactrouter.com/) (`HashRouter` para compatibilidade em qualquer hospedagem estática)
- **Estilização & Componentes de UI**:
  - [Tailwind CSS](https://tailwindcss.com/) + `tailwindcss-animate`
  - [shadcn/ui](https://ui.shadcn.com/) (baseado em primitivos acessíveis do [Radix UI](https://www.radix-ui.com/))
  - [Lucide React](https://lucide.dev/) (ícones)
  - [Sonner](https://sonner.emilkowal.ski/) (notificações toast)
- **Estado, Cache e Validação**:
  - [TanStack React Query v5](https://tanstack.com/query/latest) (busca, cache, invalidação automática e mutações assíncronas)
  - [Zod](https://zod.dev/) + [React Hook Form](https://react-hook-form.com/) (validação de esquemas e formulários)
  - [date-fns](https://date-fns.org/) (manipulação e formatação de datas no padrão `pt-BR`)
- **Back-end as a Service (BaaS)**:
  - [Firebase Authentication](https://firebase.google.com/docs/auth) (gestão de sessão e recuperação de senha)
  - [Cloud Firestore](https://firebase.google.com/docs/firestore) (banco de dados NoSQL em tempo real)

---

## 📂 Estrutura do Projeto

```text
├── public/                     # Arquivos públicos e estáticos
├── src/
│   ├── components/
│   │   ├── ui/                 # Componentes base do shadcn/ui (Button, Card, Select, Tabs, etc.)
│   │   ├── Layout.tsx          # Layout principal com Header (Desktop) e Bottom Nav (Mobile)
│   │   └── ProtectedRoute.tsx  # Guarda de rotas autenticadas e verificação de perfil Admin
│   ├── constants/
│   │   ├── times.ts            # Lista de clubes do Brasileirão e slugs dos escudos
│   │   └── urls.ts             # CDN base para carregamento dos escudos dos clubes
│   ├── contexts/
│   │   └── AuthContextFirebase.tsx # Contexto de autenticação, perfil e listener de role em tempo real
│   ├── hooks/
│   │   ├── useJogosFirebase.ts     # Queries e Mutations de rodadas e jogos
│   │   ├── usePalpitesFirebase.ts  # Queries e Mutations de palpites dos usuários
│   │   └── useRankingFirebase.ts   # Cálculo e agregação do ranking geral e Top N
│   ├── lib/
│   │   ├── firebase.ts         # Inicialização do SDK do Firebase (Auth e Firestore)
│   │   └── utils.ts            # Utilitários de classes CSS (clsx + tailwind-merge)
│   ├── pages/
│   │   ├── Admin.tsx           # Painel Administrativo (Rodadas, Jogos e Resultados)
│   │   ├── Auth.tsx            # Tela unificada de Login e Cadastro
│   │   ├── Dashboard.tsx       # Página inicial do participante
│   │   ├── ForgotPassword.tsx  # Solicitação de recuperação de senha
│   │   ├── ResetPassword.tsx   # Redefinição de senha via código do Firebase
│   │   ├── Palpites.tsx        # Tela de envio de palpites da rodada
│   │   ├── Ranking.tsx         # Classificação completa e estatísticas
│   │   └── NotFound.tsx        # Página 404
│   ├── types/
│   │   └── firebase.ts         # Interfaces TypeScript (Profile, UserRole, Rodada, Jogo, Palpite, RankingEntry)
│   ├── App.tsx                 # Configuração de provedores e rotas da aplicação
│   └── main.tsx                # Ponto de entrada da aplicação React
├── FIREBASE_SETUP.md           # Guia detalhado de configuração do Firebase e Security Rules
├── package.json                # Dependências e scripts npm
├── tailwind.config.ts          # Configuração de tema, cores e gradientes do Tailwind
└── vite.config.ts              # Configuração do bundler Vite
```

---

## 🗄️ Modelo de Dados (Cloud Firestore)

O banco de dados no Cloud Firestore utiliza 5 coleções principais:

| Coleção | Descrição | Campos Principais |
| :--- | :--- | :--- |
| **`profiles`** | Dados cadastrais públicos do participante | `id`, `nome`, `nickname`, `email`, `created_at`, `updated_at` |
| **`user_roles`** | Controle de permissões de acesso | `id`, `user_id`, `role` (`"user"` \| `"admin"`), `created_at` |
| **`rodadas`** | Rodadas do campeonato (1 a 38) | `id`, `numero`, `status`, `data_inicio`, `data_fechamento` |
| **`jogos`** | Confrontos vinculados a cada rodada | `id`, `rodada_id`, `time_casa`, `time_visitante`, `logo_casa`, `logo_visitante`, `data_jogo`, `placar_casa`, `placar_visitante`, `status` |
| **`palpites`** | Palpites registrados pelos usuários | `id`, `usuario_id`, `jogo_id`, `palpite_casa`, `palpite_visitante`, `pontos_obtidos` |

> Consulte o arquivo [`FIREBASE_SETUP.md`](./FIREBASE_SETUP.md) para visualizar as **Security Rules** recomendadas para proteção de leitura e escrita de cada coleção no Firestore.

---

## ⚙️ Configuração e Execução Local

### Pré-requisitos
- **Node.js** (versão 18 ou superior recomendada)
- **npm** ou **bun**
- Um projeto configurado no [Firebase Console](https://console.firebase.google.com/) com **Authentication (Email/Password)** e **Cloud Firestore** ativos.

### 1. Clonar o repositório e instalar dependências

```powershell
git clone https://github.com/miguelmvm/meu-palpite-brasileirao-2026.git
cd meu-palpite-brasileirao-2026
npm install
```

### 2. Configurar variáveis de ambiente

Crie um arquivo `.env` na raiz do projeto com as credenciais Web do seu projeto Firebase:

```env
VITE_FIREBASE_API_KEY=sua_api_key
VITE_FIREBASE_AUTH_DOMAIN=seu-projeto.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=seu-projeto
VITE_FIREBASE_STORAGE_BUCKET=seu-projeto.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789012
VITE_FIREBASE_APP_ID=1:123456789012:web:abcdef1234567890
```

### 3. Executar em modo de desenvolvimento

```powershell
npm run dev
```

A aplicação ficará disponível em `http://localhost:5173` (ou na porta indicada pelo Vite no terminal).

### Scripts Disponíveis

| Script | Descrição |
| :--- | :--- |
| `npm run dev` | Inicia o servidor de desenvolvimento com Hot Module Replacement (HMR) |
| `npm run build` | Gera o bundle otimizado para produção na pasta `dist/` |
| `npm run build:dev` | Gera o build em modo de desenvolvimento |
| `npm run preview` | Inicia um servidor local para testar o build de produção |
| `npm run lint` | Executa a verificação estática de código com ESLint |

---

## 👑 Como Definir um Administrador

Por padrão, todo novo usuário cadastrado recebe a função `"user"` na coleção `user_roles`. Para conceder acesso ao **Painel Administrativo (`/admin`)**:

1. Cadastre o usuário normalmente pela tela `/auth`.
2. Acesse o **Firebase Console** → **Firestore Database**.
3. Navegue até a coleção `user_roles` e localize o documento correspondente ao `uid` do usuário.
4. Altere o campo `role` de `"user"` para `"admin"`.
5. O sistema detectará a mudança em tempo real (via `onSnapshot`) e liberará automaticamente o botão **Admin** na barra de navegação e o acesso à rota `/admin`.

---

## 🌐 Deploy

O projeto está pronto para hospedagem em plataformas como **Vercel**, **Netlify** ou **Firebase Hosting**:

1. Configure as mesmas variáveis de ambiente `VITE_FIREBASE_*` no painel do seu provedor de hospedagem.
2. Utilize `npm run build` como comando de build e `dist` como diretório de saída.
3. Como o roteamento utiliza `HashRouter`, todas as rotas (`/#/palpites`, `/#/ranking`, `/#/admin`, `/#/reset-password`) funcionam nativamente sem necessidade de regras extras de rewrite no servidor.
