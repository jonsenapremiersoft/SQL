# Bancos de Dados Relacionais

## 1. Introdução e História
Os bancos de dados relacionais (BDRs) foram introduzidos na década de 1970 por **Edgar F. Codd**, um cientista da computação da IBM. Ele apresentou o modelo relacional no artigo *"A Relational Model of Data for Large Shared Data Banks"*. Este modelo revolucionou o armazenamento e a manipulação de dados, utilizando tabelas para representar informações.

**Principais Marcos:**
- **1970:** Publicação do modelo relacional por Codd.
- **1974:** Desenvolvimento do primeiro protótipo de SGBD relacional (System R) pela IBM.
- **1980s:** Lançamento comercial de SGBDs como Oracle e DB2.
- **Atualmente:** Bancos relacionais dominam o mercado empresarial e convivem com bancos NoSQL.

---

## 2. SQL (Structured Query Language)
**SQL** é a linguagem padrão para interação com bancos de dados relacionais. Criada na década de 1970, ela oferece comandos para manipular dados e gerenciar estruturas de banco.

### Comandos SQL Principais:
- **DDL (Data Definition Language):** Define a estrutura do banco.
  - `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`
- **DML (Data Manipulation Language):** Manipula os dados.
  - `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- **DCL (Data Control Language):** Controla permissões.
  - `GRANT`, `REVOKE`
- **TCL (Transaction Control Language):** Gerencia transações.
  - `COMMIT`, `ROLLBACK`

### Exemplo:
```sql
-- Criação de tabela
CREATE TABLE alunos (
    id INT PRIMARY KEY,
    nome VARCHAR(50),
    idade INT
);

-- Inserção de dados
INSERT INTO alunos (id, nome, idade) VALUES (1, 'João', 20);

-- Consulta
SELECT * FROM alunos WHERE idade > 18;
```

---

## 3. SGBDs Mais Conhecidos
**SGBD (Sistema de Gerenciamento de Banco de Dados)** é um software que permite criar, gerenciar e interagir com bancos de dados.

### Principais SGBDs Relacionais:
- **Open Source:**
  - MySQL
  - PostgreSQL
  - SQLite
- **Proprietários:**
  - Oracle Database
  - Microsoft SQL Server
  - IBM Db2

---

## 4. Propriedades ACID

![ACID](https://lh5.googleusercontent.com/zeWQaZBnd2_cixKrQerJrpo_OlovWQ8M9FBFnVJ7Nk-FrfpADN_oYXyvqXHWHkXbgIkxdqr4UoYMAb8vUgzACoo2TUrCjDYsy2fbIyzqC5F45MTr16yYEHZlMrMkIO4KJ6TIkco)

Os bancos relacionais seguem as propriedades **ACID** para garantir a integridade das transações:

---

### **1. Atomicidade**
- **Definição:** Uma transação é **indivisível**: ela deve ser completada integralmente ou ser completamente revertida. Se qualquer parte falhar, nenhuma alteração será aplicada.
- **Exemplo:**
  Imagine que você está transferindo R$100 de uma conta bancária (Conta A) para outra (Conta B):
  - **Passos da transação:**
    1. Deduzir R$100 da Conta A.
    2. Adicionar R$100 à Conta B.
  - **Cenário de falha:** Se o sistema falhar após deduzir o dinheiro da Conta A, mas antes de adicioná-lo à Conta B, a atomicidade garante que a transação seja revertida e a Conta A volte ao estado original.

---

### **2. Consistência**
- **Definição:** As transações levam o banco de dados de um estado válido a outro estado válido, respeitando regras e restrições definidas (como chaves primárias, estrangeiras e regras de negócio).
- **Exemplo:**
  Imagine que, em um sistema de inventário, você tenha uma regra que proíbe o estoque de um produto de ser negativo:
  - Uma venda tenta diminuir o estoque de um produto de 5 para -1.
  - **Cenário de falha:** A transação é rejeitada, pois violaria a regra de consistência, preservando o estado válido do banco de dados.

---

### **3. Isolamento**
- **Definição:** Transações concorrentes não interferem entre si. O resultado final é como se elas tivessem sido executadas uma após a outra.
- **Exemplo:**
  Duas pessoas estão comprando o último ingresso de um show ao mesmo tempo:
  - **Transação 1 (Pessoa A):** Verifica se há 1 ingresso disponível e o compra.
  - **Transação 2 (Pessoa B):** Faz a mesma verificação ao mesmo tempo.
  - **Sem isolamento:** Ambas podem acabar comprando o mesmo ingresso, gerando inconsistência.
  - **Com isolamento:** Uma transação será completada antes da outra iniciar, garantindo que apenas uma pessoa compre o último ingresso.

---

### **4. Durabilidade**
- **Definição:** Após uma transação ser confirmada, seus efeitos persistem, mesmo em caso de falha de energia, sistema ou hardware.
- **Exemplo:**
  Um cliente faz uma compra em uma loja online, e o pagamento é confirmado:
  - O sistema grava as alterações no banco (como atualização do estoque e registro da venda).
  - **Cenário de falha:** Se o sistema travar logo após a confirmação, a durabilidade garante que as alterações permaneçam gravadas quando o sistema for restaurado.

---

## 5. Quando Usar Bancos Relacionais?
### Cenários Ideais:
- Aplicações que exigem consistência rigorosa dos dados (bancos, ERPs).
- Necessidade de relações complexas entre os dados.
- Requisitos de consultas estruturadas e uso intensivo de SQL.
- Sistemas que demandam controle de transações (ex.: e-commerce).

### Cenários Não Recomendados:
- Processamento de grandes volumes de dados não estruturados (ex.: logs, mídias).
- Aplicações que priorizam alta escalabilidade horizontal e tolerância a falhas (ex.: redes sociais).

*Explicação: Alta escalabilidade horizontal refere-se à capacidade de aumentar a capacidade de um sistema adicionando mais unidades de hardware, como servidores ou máquinas, em paralelo. Em vez de melhorar um único servidor (escalabilidade vertical), a escalabilidade horizontal distribui a carga de trabalho entre vários servidores, permitindo que o sistema lide com mais usuários ou transações sem perda significativa de desempenho.*

---

## 6. Vantagens e Desvantagens
### Vantagens:
- **Consistência:** ACID garante integridade dos dados.
- **Flexibilidade:** SQL permite consultas complexas.
- **Padronização:** Ferramentas amplamente compatíveis.
- **Maturidade:** SGBDs relacionais têm décadas de desenvolvimento.

### Desvantagens:
- **Escalabilidade:** Menor flexibilidade para escalabilidade horizontal.
- **Desempenho:** Pode ser menos eficiente com dados não estruturados.
- **Complexidade:** Modelagem de dados pode ser mais trabalhosa.

---

### Referências:
- [Artigo original de Edgar F. Codd (PDF)](https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf)
- Documentação do [PostgreSQL](https://www.postgresql.org/docs/)
- Documentação do [MySQL](https://dev.mysql.com/doc/)
