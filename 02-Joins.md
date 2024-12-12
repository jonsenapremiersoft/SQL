# JOINS

## Introdução

JOINS são operações que nos permitem combinar dados de diferentes tabelas baseados em condições específicas.

## Estrutura Base

Para entender melhor os diferentes tipos de JOINS, vamos usar duas tabelas como exemplo:

```sql
CREATE TABLE clientes (
    id_cliente SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE pedidos (
    id_pedido SERIAL PRIMARY KEY,
    id_cliente INT,
    valor DECIMAL(10,2),
    data_pedido DATE,
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);
```

## Tipos de JOINS

### INNER JOIN

O INNER JOIN é o tipo mais comum de JOIN. Ele retorna apenas os registros que têm correspondência em ambas as tabelas.

```sql
SELECT c.nome, p.id_pedido, p.valor
FROM clientes c
INNER JOIN pedidos p ON c.id_cliente = p.id_cliente;
```

Este comando retornará apenas os clientes que fizeram pedidos e seus respectivos pedidos.

### LEFT JOIN (ou LEFT OUTER JOIN)

O LEFT JOIN retorna todos os registros da tabela à esquerda (primeira tabela mencionada) e os registros correspondentes da tabela à direita. Se não houver correspondência, os campos da tabela à direita serão preenchidos com NULL.

```sql
SELECT c.nome, p.id_pedido, p.valor
FROM clientes c
LEFT JOIN pedidos p ON c.id_cliente = p.id_cliente;
```

Este comando retornará todos os clientes, mesmo aqueles que nunca fizeram pedidos.

### RIGHT JOIN (ou RIGHT OUTER JOIN)

O RIGHT JOIN funciona de maneira similar ao LEFT JOIN, mas retorna todos os registros da tabela à direita e os registros correspondentes da tabela à esquerda.

```sql
SELECT c.nome, p.id_pedido, p.valor
FROM clientes c
RIGHT JOIN pedidos p ON c.id_cliente = p.id_cliente;
```

### FULL JOIN (ou FULL OUTER JOIN)

O FULL JOIN retorna todos os registros quando houver uma correspondência em qualquer uma das tabelas. Se não houver correspondência, os campos serão preenchidos com NULL.

```sql
SELECT c.nome, p.id_pedido, p.valor
FROM clientes c
FULL JOIN pedidos p ON c.id_cliente = p.id_cliente;
```

## Exemplos Práticos

### Exemplo 1: Encontrar clientes e seus últimos pedidos

```sql
SELECT 
    c.nome,
    p.id_pedido,
    p.valor,
    p.data_pedido
FROM clientes c
LEFT JOIN pedidos p ON c.id_cliente = p.id_cliente
WHERE p.data_pedido = (
    SELECT MAX(data_pedido)
    FROM pedidos p2
    WHERE p2.id_cliente = c.id_cliente
);
```

### Exemplo 2: Relatório de vendas por cliente

```sql
SELECT 
    c.nome,
    COUNT(p.id_pedido) as total_pedidos,
    SUM(p.valor) as valor_total
FROM clientes c
LEFT JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.nome
ORDER BY valor_total DESC;
```

## Dicas Importantes

1. Performance: INNER JOIN geralmente tem melhor performance que OUTER JOINs.

2. Índices: Sempre crie índices nas colunas utilizadas nas condições de JOIN para melhor performance:
```sql
CREATE INDEX idx_pedidos_cliente ON pedidos(id_cliente);
```

3. Alias: Use aliases para tornar suas queries mais legíveis e evitar ambiguidade:
```sql
SELECT c.nome, p.valor
FROM clientes AS c
JOIN pedidos AS p ON c.id_cliente = p.id_cliente;
```

## Exercícios Práticos

Para fixar o conhecimento, tente realizar os seguintes exercícios:

1. Liste todos os clientes e a quantidade de pedidos que cada um fez
2. Encontre todos os clientes que nunca fizeram pedidos
3. Calcule o valor médio dos pedidos por cliente
4. Liste os clientes que fizeram pedidos nos últimos 30 dias
