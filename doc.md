# Desenvolvimento de sistema em conjunto (TBD)

## Definições de sistemas / metodologias/ tecnologias / afins

| Tema | Recurso |
|---|---|
| Controle de versão | `github` |
| Estrutura de repositórios | Repos separados (front / back / mobile / extensão) |
| Database | `PostgreSQL` (Neon ou Supabase, free tier) |
| ORM | `Prisma` |
| Database Cache | `Redis` (Upstash, free tier) |
| Tecnologia Front | `Next` |
| Tecnologia Back | `NestJS` |
| Contrato Front↔Back | OpenAPI/Swagger (gerado pelo NestJS) + client TS gerado (`orval`) |
| Auth | JWT próprio (Passport.js) no backend |
| Realtime (kanban colaborativo) | Supabase Realtime ou Socket.io |
| Storage (PDFs, comprovantes) | Supabase Storage ou Cloudflare R2 |
| Leitura de PDF | `pdf-parse`/`pdf.js` (+ Tesseract.js p/ escaneados) + LLM p/ extração estruturada |
| Integração cartão de crédito (BR) | Pluggy ou Belvo (Open Finance) |
| Mobile | Expo (React Native) |
| Extensão de browser | Plasmo ou WXT |
| Sistema de controle de tasks | `Trello` |

## Considerações

 - Compartilhamento dos gerenciamentos de tarefas entre usuários
 - Compartilhamento SEPARADO dos gerenciamentos financeiros entre usuários
 - Uso paralelo e att em tempo real
 - Repos separados de front e back: tipos compartilhados via client gerado a partir do OpenAPI, sem monorepo

## Definições de aplicação

1. Criar sistema de usuários (login) separando a responsabilidade de cada usuário e autorização de acesso a API (Verificar como funciona com outras tecnologias)

2. Gerenciamento de tarefas
    - Visualização de kanban 
    - Visualização de calendário

3. Gerenciamento financeiro
    - Possibilitar importar fotos/documentos para inserir os dados
    - integrar com possíveis cartões de crédito para inserir gastos (nubank?)
    - Gerenciamento de empréstimos/dívidas
    - Compartilhar alguma visão com certos dados entre usuários (ex: por conta de algum evento)

4. Gerenciamento de financiamento ou relacionados
    - Financiamentos

5. Gestão de empréstimos


====== Fora de contexto

6. Sistema de linha temporal de eventos históricos (feat wikipedia)

======

## Stories

- Criar documento de padrão de projeto (ADR)
- Criar doc PRD (Features do sistema)
- 


## Definições técnicas

### GitHub

- Fluxo:
    1. Criar branch a partir de `DEVELOP`
    2. Desenvolver
    3. Abrir PR para `DEVELOP`
    4. ...

- Padrão de formato de branchs:
    - ex: `features/#0001-Tarefa-tal`
- Commits: `feat`, `fix`, `bugfix` ... 
    - ex: `#0001 fix: Removendo comentarios desnecessarios`
    - seguindo https://www.conventionalcommits.org/en/v1.0.0/
- 


## Prompts de uso para o projeto

Estou querendo criar um sistema do zero, simples, com a funcionalidade de gerenciamento de tarefas separando por quadros, onde é possível visualizar em forma de kanban ou em forma de calendário, permitindo multiplos usuários utilizarem quadros em conjunto. Além disso, outro tipo de quadro para gerenciador de finanças, onde quero poder importar arquivos pdf que serão lidos para buscar dados específicos para preencher informações financeiras, quero também poder integrar com outros sistemas de cartão de crédito, onde importarei dados de gastos de cartões para adicionar no controle financeiro. Quero poder trabalhar em conjunto com outros desenvolvedores. 

Comece dando sugestões de quais tecnologias são melhores para serem usadas e que facilitem o desenvolvimento, para que o sistema seja de uso rápido, se possível consiga escalar para outras plataformas como mobile ou extensões de browsers. De sugestões de banco de dados para serem utilizados, e lembre-se de sugerir tecnologias que não tenham custo de uso
