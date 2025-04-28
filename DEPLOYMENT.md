# Guia de Implantação - Aplicativo Hamburgueria Miraculos (Next.js)

Este guia fornece instruções sobre como compilar e implantar o aplicativo de autoatendimento da Hamburgueria Miraculos, desenvolvido com Next.js e configurado para Cloudflare Workers/Pages.

## Pré-requisitos

*   Node.js (versão 20.x ou superior)
*   pnpm (gerenciador de pacotes)
*   Conta em um serviço de hospedagem (recomendado: Cloudflare Pages, Vercel, Netlify)
*   Wrangler CLI (se implantando no Cloudflare)

## Estrutura do Projeto

O código-fonte está organizado da seguinte forma:

```
hamburgueria-miraculos/
├── migrations/            # Migrações do banco de dados D1
│   └── 0001_initial.sql
├── public/              # Arquivos estáticos
├── src/
│   ├── app/             # Páginas e APIs do Next.js
│   │   ├── api/         # Endpoints da API
│   │   ├── carrinho/
│   │   ├── produto/[id]/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── providers.tsx
│   ├── components/      # Componentes React reutilizáveis
│   │   └── Navigation.tsx
│   ├── context/         # Contexto React (Carrinho)
│   │   └── CartContext.tsx
│   └── ...
├── .env.example         # Exemplo de variáveis de ambiente
├── .gitignore
├── next.config.js       # Configuração do Next.js
├── package.json
├── pnpm-lock.yaml
├── README.md            # Este guia
├── tsconfig.json
└── wrangler.toml        # Configuração do Cloudflare
```

## Configuração

1.  **Variáveis de Ambiente:** Renomeie `.env.example` para `.env` e configure as variáveis necessárias (se houver, no futuro, como chaves de API para pagamento, etc.). Para a implantação, configure essas variáveis no painel do seu provedor de hospedagem.

2.  **Banco de Dados (Cloudflare D1):**
    *   Se estiver usando Cloudflare Pages, crie um banco de dados D1 no painel do Cloudflare.
    *   Atualize o `wrangler.toml` com o `database_id` real do seu banco de dados D1 de produção (substitua o `database_id = "local"`).
    *   Execute as migrações no banco de dados de produção usando Wrangler:
        ```bash
        wrangler d1 migrations apply DB --remote
        ```
    *   Se estiver usando outra hospedagem (Vercel/Netlify), você precisará configurar um banco de dados diferente (ex: PostgreSQL no Neon, Supabase, etc.) e adaptar o código do backend (APIs em `src/app/api/`) para usar o driver e as credenciais apropriadas para esse banco de dados. O D1 é específico do Cloudflare.

## Compilação (Build)

Antes de implantar, você precisa compilar o aplicativo para produção:

1.  **Instale as Dependências:**
    ```bash
    pnpm install
    ```
2.  **Execute o Build:**
    ```bash
    pnpm run build
    ```
    *   **Observação:** Encontramos erros durante a compilação neste ambiente. Pode ser necessário investigar e resolver esses erros em um ambiente de desenvolvimento local ou diretamente na plataforma de hospedagem, que geralmente possui ambientes de compilação mais robustos. Os erros podem estar relacionados a dependências, configurações do Webpack/SWC ou limitações do ambiente atual.

## Implantação

### Opção 1: Cloudflare Pages (Recomendado)

1.  **Conecte seu Repositório:** Faça o push do código para um repositório Git (GitHub, GitLab).
2.  **Crie um Projeto no Cloudflare Pages:**
    *   Vá para o painel do Cloudflare > Workers & Pages > Create application > Pages > Connect to Git.
    *   Selecione seu repositório.
    *   Configure as definições de build:
        *   **Framework preset:** Next.js
        *   **Build command:** `pnpm run build`
        *   **Build output directory:** `.open-next` (geralmente detectado automaticamente)
    *   **Variáveis de Ambiente:** Adicione as variáveis necessárias.
    *   **Vinculação D1:** Vincule seu banco de dados D1 criado anteriormente à variável `DB`.
3.  **Implante:** Salve e implante.

### Opção 2: Vercel

1.  **Conecte seu Repositório:** Faça o push do código para um repositório Git.
2.  **Importe o Projeto na Vercel:**
    *   Vá para o painel da Vercel > Add New... > Project.
    *   Selecione seu repositório Git.
    *   A Vercel geralmente detecta que é um projeto Next.js e configura o build automaticamente.
    *   **Framework Preset:** Next.js
    *   **Build Command:** `pnpm run build`
    *   **Output Directory:** `.next` (padrão da Vercel para Next.js, pode precisar ajustar se o build para Cloudflare gerou `.open-next`)
    *   **Install Command:** `pnpm install`
3.  **Banco de Dados:** Configure um banco de dados externo (ex: Vercel Postgres, Neon, Supabase) e adicione as credenciais como variáveis de ambiente. Adapte o código da API para usar este banco de dados.
4.  **Implante.**

### Opção 3: Netlify

1.  **Conecte seu Repositório:** Faça o push do código para um repositório Git.
2.  **Importe o Projeto na Netlify:**
    *   Vá para o painel da Netlify > Add new site > Import an existing project.
    *   Selecione seu provedor Git e o repositório.
    *   Configure as definições de build:
        *   **Build command:** `pnpm run build`
        *   **Publish directory:** `.next` (ou `.open-next` dependendo da saída do build)
3.  **Banco de Dados:** Configure um banco de dados externo e adicione as credenciais como variáveis de ambiente. Adapte o código da API.
4.  **Implante.**

## Pós-Implantação

1.  **Teste:** Acesse a URL fornecida pelo serviço de hospedagem e teste todas as funcionalidades.
2.  **Domínio Personalizado:** Configure um domínio personalizado (opcional) através do painel do seu provedor de hospedagem.

Lembre-se que a compilação (`pnpm run build`) precisa ser bem-sucedida antes que a implantação funcione corretamente.

