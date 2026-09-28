# Portal de Agendamento de Coletas — Wheaton Brasil Vidros

Portal em HTML/JS puro (sem build) para transportadoras agendarem e
cancelarem coletas, com painel operacional para o administrador. Os
dados são persistidos no [Supabase](https://supabase.com).

## Estrutura do repositório

```
.
├── index.html              # A aplicação inteira (front-end)
├── config.example.js       # Template de configuração do Supabase
├── config.js                # Suas credenciais reais (NÃO versionar — está no .gitignore)
├── supabase_schema.sql     # Script de criação/ajuste da tabela no Supabase
└── .github/workflows/
    └── deploy.yml          # Publica automaticamente no GitHub Pages
```

## 1. Configurar o Supabase

1. Crie um projeto em [app.supabase.com](https://app.supabase.com) (ou use um existente).
2. Abra o **SQL Editor** do projeto.
3. Cole o conteúdo de [`supabase_schema.sql`](./supabase_schema.sql) e execute.
   O script é seguro para rodar mais de uma vez (usa `IF NOT EXISTS`/`IF EXISTS`)
   e funciona tanto para criar a tabela do zero quanto para ajustar uma
   tabela `agendamentos` já existente.
4. Em **Project Settings → API**, copie a **Project URL** e a chave
   **anon public**.

## 2. Rodar localmente

1. Copie `config.example.js` para `config.js`.
2. Preencha `SUPABASE_URL` e `SUPABASE_ANON_KEY` com os valores do passo 1.
3. Abra `index.html` diretamente no navegador, ou sirva a pasta com
   qualquer servidor estático (ex.: `npx serve .`).

Sem um `config.js` preenchido, o portal funciona apenas localmente
nesta sessão (nada é salvo no banco compartilhado) — útil para testar
a interface sem afetar dados reais.

## 3. Publicar no GitHub Pages (deploy automático)

Este repositório já vem com um workflow (`.github/workflows/deploy.yml`)
que publica o site no GitHub Pages a cada `push` na branch `main`.

Passos:

1. Em **Settings → Pages**, em "Build and deployment", selecione
   **Source: GitHub Actions**.
2. Em **Settings → Secrets and variables → Actions → New repository secret**,
   crie dois secrets:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
3. Dê um `push` na branch `main`. O workflow vai:
   - gerar um `config.js` a partir dos secrets (o arquivo real nunca
     fica salvo no repositório, só existe durante o deploy);
   - publicar todo o conteúdo do repositório no GitHub Pages.
4. A URL final fica disponível em **Settings → Pages** após o primeiro
   deploy rodar (aba **Actions** mostra o progresso).

> **Sobre a anon key do Supabase:** ela é feita para ser usada no
> navegador (é assim que o Supabase funciona) — a segurança real vem
> das políticas de RLS (Row Level Security) definidas em
> `supabase_schema.sql`, não do sigilo dessa chave. Ainda assim, o
> `config.js` com valores reais fica fora do controle de versão por
> padrão, e o workflow acima cobre o deploy sem precisar commitá-lo.

## 4. Contas de acesso

O portal tem dois perfis fixos definidos em `index.html`:

| Perfil         | Usuário         | Acesso                                  |
|----------------|-----------------|------------------------------------------|
| Transportadora | `transportadora`| Agendamento, Cancelamento, Acompanhar Status |
| Administrador  | `operacional`   | Acesso completo + Painel Operacional     |

As senhas iniciais estão no próprio `index.html` (procure por
`CREDENCIAIS_PADRAO`) e podem ser alteradas dentro do portal; as
alterações ficam salvas no `localStorage` do navegador de cada usuário.

> **Aviso de segurança:** o login é feito apenas no navegador (as
> credenciais ficam visíveis no código-fonte) e as políticas de RLS do
> `supabase_schema.sql` liberam leitura e escrita para qualquer pessoa
> com a anon key. Isso serve para uso interno simples, mas não impede
> que alguém tecnicamente hábil acesse ou altere os dados. Para uso
> mais sensível, migre o login para o Supabase Auth e restrinja as
> políticas de RLS a usuários autenticados.

## Funcionalidades

- Agendamento de coleta com validação de janelas de horário, feriados
  (nacionais e do Estado de São Paulo) e domingos.
- Cálculo automático e travado da quantidade de ajudantes, conforme:
  - **Caixas:** até 100 → 1 ajudante · 101 a 1000 → 2 ajudantes · acima de 1000 → 3 ajudantes
  - **Paletes:** até 40 → 1 ajudante · acima de 40 → 2 ajudantes
- Cancelamento de coleta (busca por Protocolo, PEDIDO ou Ordem de Coleta).
- Acompanhamento de status da coleta pela transportadora (somente consulta).
- Painel operacional (administrador): KPIs, tabela completa, exportação CSV,
  detalhes de cada coleta.
- Sincronização em tempo real via Supabase Realtime, com fallback por
  polling a cada 20s.
