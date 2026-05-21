# 🛒 Laboratório Prático de Banco de Dados - E-commerce

Este repositório contém uma atividade prática de programação de banco de dados desenvolvida em **MySQL**, utilizando o cenário clássico de um sistema de E-commerce para explorar recursos e estruturas avançadas de manipulação e segurança de dados.

## 📝 Descrição da Atividade
Atividade prática de banco de dados MySQL baseada em um e-commerce. Aborda a criação de estruturas avançadas como Views, Functions, Triggers e Procedures, além de simular Materialized Views para otimização de relatórios e aplicar conceitos essenciais de segurança com controle de acessos (Grant/Revoke).

---

## 🛠️ Recursos Implementados

O script está estruturado nas seguintes etapas:

1. **Estrutura Base:** Criação do banco de dados `lab_ecommerce` e das tabelas de `produtos`, `vendas` e `auditoria_precos`.
2. **Views:** Criação da `vw_produtos_disponiveis` para exibir produtos em estoque de forma segura, ocultando dados sensíveis.
3. **Functions:** Implementação da função `fn_calcular_desconto` para aplicar dinamicamente 10% de desconto em compras acima de 3 unidades.
4. **Triggers:** Configuração do gatilho `trg_auditoria_preco` para monitorar e registrar automaticamente qualquer alteração de preços no histórico.
5. **Procedures:** Criação da `sp_adicionar_estoque` para atualização controlada de saldo de mercadorias.
6. **Materialized Views:** Simulação de cache físico através de tabela e da procedure `sp_refresh_mv_resumo_vendas` para otimizar consultas pesadas de relatórios.
7. **DCL (Controle de Acesso):** Criação de usuário analista e gerenciamento de permissões restritas com comandos `GRANT` e `REVOKE`.

---

## 💻 Código SQL Completo

```sql
-- =====================================================================
-- ESTRUTURA BASE
-- =====================================================================

CREATE DATABASE IF NOT EXISTS lab_ecommerce;
USE lab_ecommerce;

CREATE TABLE produtos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100),
    preco DECIMAL(10,2),
    estoque INT
);

CREATE TABLE vendas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT,
    quantidade INT,
    total DECIMAL(10,2),
    data_venda DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES produtos(id)
);

CREATE TABLE auditoria_precos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT,
    preco_antigo DECIMAL(10,2),
    preco_novo DECIMAL(10,2),
    data_alteracao DATETIME DEFAULT CURRENT_TIMESTAMP
