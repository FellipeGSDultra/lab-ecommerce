# 🛒 Lab E-Commerce: Banco de Dados Relacional e Recursos Avançados SQL

Este repositório contém o script SQL para a criação e configuração do **`lab_ecommerce`**, um ambiente de laboratório projetado para demonstrar boas práticas de modelagem de dados e a implementação de recursos avançados em bancos de dados relacionais (compatível com MySQL / MariaDB).

O projeto aborda desde a estruturação básica de tabelas até conceitos avançados de automação, otimização de performance e governança de dados, tais como **Views, Functions, Triggers, Stored Procedures, Views Materializadas Simuladas** e **Controle de Acesso (DCL)**.

---

## 📌 Sumário
1. [Estrutura do Banco de Dados](#-estrutura-do-banco-de-dados)
2. [Recursos Implementados](#-recursos-implementados)
    - [1. Views](#1-views)
    - [2. Functions](#2-functions)
    - [3. Triggers](#3-triggers)
    - [4. Stored Procedures](#4-stored-procedures)
    - [5. Materialized Views (Simulação de Cache)](#5-materialized-views-simulação-de-cache)
    - [6. Governança e Permissões (DCL)](#6-governança-e-permissões-dcl)
3. [Como Executar e Testar](#-como-executar-e-testar)

---

## 🗂️ Estrutura do Banco de Dados

O ecossistema do banco de dados é composto por 3 tabelas principais de operação e uma tabela de suporte técnico:

* **`produtos`**: Armazena o catálogo de itens, preços base e o nível atual de estoque.
* **`vendas`**: Registra as transações financeiras, vinculando o produto, quantidade comprada e o valor total calculado.
* **`auditoria_precos`**: Tabela de histórico que rastreia automaticamente qualquer alteração de preços na tabela de produtos para fins de auditoria e compliance.

---

## 🚀 Recursos Implementados

### 1. Views
**Objetivo:** Camada de abstração para segurança e simplificação de consultas.
* **`vw_produtos_disponiveis`**: Filtra e exibe apenas os produtos que possuem estoque ativo (`estoque > 0`). Ela mascara os IDs internos e expõe apelidos amigáveis (`Produto`, `Valor`, `Quantidade Disponivel`), protegendo a estrutura física dos dados contra acessos diretos de ferramentas de BI ou analistas.

### 2. Functions
**Objetivo:** Encapsulamento de regras de negócio reutilizáveis.
* **`fn_calcular_desconto`**: Função determinística que automatiza a política de descontos do e-commerce. Se o cliente comprar uma quantidade igual ou superior a 3 unidades de um mesmo produto, o sistema aplica automaticamente **10% de desconto** sobre o valor total daquela linha de venda.

### 3. Triggers
**Objetivo:** Automação preventiva e integridade referencial histórica.
* **`trg_auditoria_preco`**: Disparado automaticamente **após qualquer atualização (`AFTER UPDATE`)** na tabela de `produtos`. Ele compara o preço antigo (`OLD.preco`) com o novo preço (`NEW.preco`). Se houver divergência, ele popula a tabela `auditoria_precos` registrando o ID do produto, o valor anterior, o novo valor e o timestamp exato da alteração.

### 4. Stored Procedures
**Objetivo:** Modularização de processos operacionais e consistência de escrita.
* **`sp_adicionar_estoque`**: Centraliza a entrada de mercadorias no estoque. Em vez de permitir comandos `UPDATE` livres na aplicação, esta procedure recebe o `id` do produto e a `quantidade` a ser incrementada, executando a alteração de forma controlada e retornando uma mensagem de confirmação para o usuário/sistema.

### 5. Materialized Views (Simulação de Cache)
**Objetivo:** Otimização de performance para consultas analíticas pesadas (OLAP).
* Como o MySQL nativamente não possui suporte a Views Materializadas, foi implementada uma engenharia de contorno utilizando uma tabela física de cache (`mv_resumo_vendas`) e uma rotina de atualização via procedure (`sp_refresh_mv_resumo_vendas`).
* A procedure limpa os dados antigos (`TRUNCATE`) e consolida o faturamento total acumulado por produto através de um agrupamento pesado (`SUM` e `JOIN`), permitindo que relatórios gerenciais consultem a tabela física instantaneamente sem reprocessar milhões de linhas de vendas em tempo real.

### 6. Governança e Permissões (DCL)
**Objetivo:** Princípio do menor privilégio e segurança da informação.
* O script simula um cenário real de administração de banco de dados (DBA), onde é criado um usuário restrito (`analista_dados`).
* Utilizando comandos **DCL (Data Control Language)**, o usuário recebe permissão de leitura exclusivamente na View de produtos disponíveis (`GRANT SELECT`), sem acesso às tabelas base. O script também demonstra como revogar (`REVOKE`) esses privilégios caso haja mudança de escopo na função do colaborador.

---

## 💻 Como Executar e Testar

### Pré-requisitos
* MySQL Server 8.0+ ou MariaDB equivalente.
* Um cliente SQL de sua preferência (DBeaver, MySQL Workbench, VS Code, etc.).

### Passo a Passo

1.  **Clone ou copie** o script SQL fornecido.
2.  Execute o bloco de **Estrutura Base** para criar o banco `lab_ecommerce` e popular os dados iniciais.
3.  Execute cada seção de recursos avançados sequencialmente (Views, Functions, Triggers, Procedures e DCL).
4.  Utilize os comandos de **Teste** comentados ao longo do script para validar o comportamento de cada recurso. Por exemplo:
    ```sql
    -- Testando o gatilho de auditoria de preço
    UPDATE produtos SET preco = 3200.00 WHERE id = 1;
    SELECT * FROM auditoria_precos;
    
    -- Testando o cache da View Materializada
    CALL sp_refresh_mv_resumo_vendas();
    SELECT * FROM mv_resumo_vendas;
    ```

---

## 🛠️ Tecnologias Utilizadas
* **Linguagem:** SQL (Structured Query Language)
* **Dialeto:** MySQL / MariaDB
* **Paradigma:** Banco de Dados Relacional (RDBMS)

---
*Desenvolvido para fins educacionais e de demonstração técnica de arquitetura SQL.*
