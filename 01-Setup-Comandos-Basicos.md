# Introdução a Bancos de Dados Relacionais e PostgreSQL

## Preparando o Ambiente

### 1. Instalando o PostgreSQL

1. Acesse o [site oficial do PostgreSQL](https://www.postgresql.org/download/windows/)
2. Clique no link do instalador do Windows
3. Selecione a versão mais recente (atualmente 16.2)
4. Execute o instalador baixado
5. Durante a instalação:
   - Mantenha a porta padrão (5432)
   - Defina uma senha para o usuário 'postgres' (ANOTE ESTA SENHA!)
   - Mantenha a codificação padrão (UTF-8)
   - Selecione seu local/região

### 2. Instalando o DBeaver - Nossa Interface Visual

O DBeaver é uma ferramenta gratuita que nos ajudará a visualizar e manipular o banco de dados.

1. Acesse o [site do DBeaver](https://dbeaver.io/download/)
2. Baixe a versão Community Edition para Windows
3. Execute o instalador
4. Siga o assistente de instalação com as opções padrão

## Configurando nossa Primeira Conexão

1. Abra o DBeaver
2. Na tela inicial:
   - Clique no ícone "Nova Conexão" (plug com '+')
   - Selecione "PostgreSQL"
   - Clique "Próximo"
3. Configure a conexão:
   - Host: localhost
   - Port: 5432
   - Database: postgres
   - Username: postgres
   - Password: (a senha que você definiu na instalação)
4. Clique em "Test Connection" para verificar se está tudo certo
5. Se o teste for bem sucedido, clique em "Finish"

## Criando Nosso Primeiro Banco de Dados

1. No DBeaver, clique com o botão direito em "Databases"
2. Selecione "Create New Database"
3. Nome: loja_virtual
4. Clique em "OK"

Agora vamos aprender os comandos SQL básicos e já praticar!

## Comandos SQL Básicos

### Criando Nossa Primeira Tabela

1. No DBeaver, expanda a conexão PostgreSQL
2. Expanda o banco "loja_virtual"
3. Clique com botão direito em "Tables"
4. Selecione "SQL Editor" -> "New SQL Script"
5. Cole o seguinte código:

```sql
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    estoque INTEGER DEFAULT 0,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

6. Clique no botão "Execute SQL Script" (ícone de play)
7. Para ver a tabela criada:
   - Clique com botão direito em "Tables"
   - Selecione "Refresh"
   - Sua tabela "produtos" aparecerá na lista

### Inserindo Dados

No mesmo editor SQL (ou crie um novo), vamos inserir alguns produtos:

```sql
-- Inserindo um único produto
INSERT INTO produtos (nome, preco, estoque)
VALUES ('Notebook Dell', 3500.00, 5);

-- Inserindo vários produtos de uma vez
INSERT INTO produtos (nome, preco, estoque) VALUES 
    ('Mouse Gamer', 199.90, 20),
    ('Teclado Mecânico', 350.00, 15),
    ('Headset', 180.00, 10);
```

### Consultando Dados

Vamos aprender diferentes formas de consultar nossos dados:

```sql
-- Ver todos os produtos
SELECT * FROM produtos;

-- Ver apenas nome e preço
SELECT nome, preco FROM produtos;

-- Ver produtos com preço maior que 200
SELECT nome, preco 
FROM produtos 
WHERE preco > 200;

-- Ordenar produtos por preço (do mais barato ao mais caro)
SELECT nome, preco 
FROM produtos 
ORDER BY preco ASC;
```

### Atualizando Dados

Vamos atualizar o estoque de um produto:

```sql
-- Diminuir o estoque do Notebook em 1 unidade
UPDATE produtos 
SET estoque = estoque - 1 
WHERE nome = 'Notebook Dell';

-- Verificar a alteração
SELECT nome, estoque 
FROM produtos 
WHERE nome = 'Notebook Dell';
```

### Deletando Dados

```sql
-- Deletar produtos com estoque zerado
DELETE FROM produtos 
WHERE estoque = 0;
```

## Criando um Sistema de Loja Virtual

Agora vamos criar um sistema mais completo com várias tabelas relacionadas:

1. Primeiro, as categorias:

```sql
CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL,
    descricao TEXT
);
```

2. Modificar a tabela produtos para incluir categoria:

```sql
-- Primeiro, vamos excluir a tabela antiga
DROP TABLE produtos;

-- Agora criar a nova versão
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    categoria_id INTEGER REFERENCES categorias(id),
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    estoque INTEGER DEFAULT 0,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

3. Criar tabela de clientes:

```sql
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    telefone VARCHAR(20),
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

4. Criar tabela de pedidos:

```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER REFERENCES clientes(id),
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'Pendente'
);

CREATE TABLE itens_pedido (
    id SERIAL PRIMARY KEY,
    pedido_id INTEGER REFERENCES pedidos(id),
    produto_id INTEGER REFERENCES produtos(id),
    quantidade INTEGER NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL
);
```

### Inserindo Dados de Exemplo

1. Categorias:
```sql
INSERT INTO categorias (nome, descricao) VALUES 
    ('Informática', 'Produtos de computador e acessórios'),
    ('Eletrônicos', 'Gadgets e dispositivos eletrônicos'),
    ('Games', 'Jogos e consoles');
```

2. Produtos:
```sql
INSERT INTO produtos (categoria_id, nome, preco, estoque) VALUES 
    (1, 'Notebook Dell', 3500.00, 5),
    (1, 'Mouse Gamer', 199.90, 20),
    (1, 'Teclado Mecânico', 350.00, 15),
    (2, 'Smartphone Samsung', 2500.00, 8),
    (3, 'PlayStation 5', 4500.00, 3);
```

3. Clientes:
```sql
INSERT INTO clientes (nome, email, telefone) VALUES 
    ('João Silva', 'joao@email.com', '11999998888'),
    ('Maria Santos', 'maria@email.com', '11999997777');
```

4. Pedidos:
```sql
-- Criar um pedido
INSERT INTO pedidos (cliente_id) VALUES (1);

-- Adicionar itens ao pedido
INSERT INTO itens_pedido (pedido_id, produto_id, quantidade, preco_unitario) VALUES 
    (1, 1, 1, 3500.00),  -- Notebook
    (1, 2, 2, 199.90);   -- 2 Mouses
```

### Consultas Úteis

1. Ver produtos por categoria:
```sql
SELECT c.nome as categoria, p.nome as produto, p.preco
FROM produtos p
JOIN categorias c ON p.categoria_id = c.id
ORDER BY c.nome, p.nome;
```

2. Ver pedidos de um cliente:
```sql
SELECT 
    c.nome as cliente,
    p.data_pedido,
    pr.nome as produto,
    ip.quantidade,
    ip.preco_unitario,
    (ip.quantidade * ip.preco_unitario) as total
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
JOIN itens_pedido ip ON p.id = ip.pedido_id
JOIN produtos pr ON ip.produto_id = pr.id
WHERE c.id = 1;
```

## Dicas Importantes

1. No DBeaver:
   - Use F5 para atualizar a visualização das tabelas
   - Ctrl + Enter executa a consulta SQL onde o cursor está
   - Clique duas vezes em uma tabela para ver seus dados
   - Clique com botão direito em uma tabela -> "View Table" para ver/editar dados
   
2. Sempre faça backup dos seus dados importantes:
   - No DBeaver: Botão direito no banco -> "Tools" -> "Backup"
   
3. Caso cometa algum erro:
   - Use ROLLBACK para desfazer alterações não confirmadas
   - Mantenha backups regulares
   - Teste comandos perigosos (UPDATE, DELETE) primeiro com SELECT

## Exercícios Práticos

1. Crie uma nova categoria chamada "Acessórios"
2. Adicione 3 produtos nesta categoria
3. Crie um novo cliente
4. Faça um pedido para este cliente com pelo menos 2 produtos
5. Consulte:
   - Total de produtos por categoria
   - Valor total de todos os pedidos
   - Produtos com estoque baixo (menos de 5 unidades)
