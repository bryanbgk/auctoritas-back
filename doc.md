# Desenvolvimento de sistema em conjunto (TBD)

## Definições de sistemas / metodologias/ tecnologias / afins

| Tema | Recurso |
|---|---|
| Controle de versão | `github` |
| Database | TDB |
| Database Cache | `Redis` |
| Tecnologia Front | `Next` |
| Tecnologia Back | TDB |
| Sistema de controle de tasks | `Trello` |

## Considerações

 - Compartilhamento dos gerenciamentos de tarefas entre usuários
 - Compartilhamento SEPARADO dos gerenciamentos financeiros entre usuários
 - Uso paralelo e att em tempo real

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

## Prompts de uso para o projeto

- Estou querendo criar um sistema simples com a funcionalidade de kanban com multiplos usuários, para gerenciamento de tasks que seja possível visualizar as tasks como kanban, ou em forma de agenda de acordo com as datas definidas de cada task, além de um gerenciador de finanças, onde quero poder importar arquivos pdf que serão lidos para buscar dados específicos para preencher informações financeiras, quero também poder integrar com outros sistemas de cartão de crédito, onde importarei dados de gastos de cartões para adicionar no controle financeiro. Quero poder trabalhar em conjunto com outros desenvolvedores. De sugestões de tecnologias para serem usadas que facilitem o desenvolvimento, para que o sistema seja de uso rápido, se possível consiga escalar para outras plataformas como mobile ou extensões de browsers. De sugestões de banco de dados para serem utilizados, e lembre-se de sugerir tecnologias que não tenham custo de uso
