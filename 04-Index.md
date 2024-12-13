# Índices no PostgreSQL

## Introdução aos Índices

Um índice em SQL é como um mapa que ajuda o banco de dados a encontrar informações rapidamente, similar ao índice de um livro. Sem índices, o PostgreSQL precisaria verificar cada linha de uma tabela para encontrar os dados desejados (isso é chamado de "sequential scan"). Com índices, o banco pode ir diretamente para a localização dos dados relevantes.

## Funcionamento Interno dos Índices

Quando você cria um índice, o PostgreSQL mantém uma estrutura de dados separada que contém:
1. Os valores da coluna indexada, armazenados de forma ordenada
2. Ponteiros físicos (TID - Tuple Identifier) que apontam para a localização exata do registro na tabela

Por exemplo, considere esta tabela:

```sql
CREATE TABLE funcionarios (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    salario DECIMAL(10,2)
);
```

Quando criamos um índice no campo salário:

```sql
CREATE INDEX idx_funcionarios_salario ON funcionarios (salario);
```

O PostgreSQL mantém uma estrutura ordenada dos salários com ponteiros para os registros originais. Assim, quando você executa:

```sql
SELECT * FROM funcionarios WHERE salario > 5000;
```

Em vez de verificar todos os registros, o banco usa o índice para encontrar rapidamente os salários maiores que 5000.

## Tipos de Índices em Detalhes

### 1. B-tree (Balanced Tree)
- É o tipo padrão e mais versátil
- Funciona bem para dados que podem ser ordenados
- Suporta operadores: <, <=, =, >=, >
- Exemplo de uso:

```sql
-- Índice B-tree em uma coluna de data
CREATE INDEX idx_pedidos_data ON pedidos (data_pedido);
```

### 2. Hash
- Otimizado exclusivamente para comparações de igualdade (=)
- Geralmente mais rápido que B-tree para buscas exatas
- Exemplo:

```sql
CREATE INDEX idx_usuarios_email_hash ON usuarios USING HASH (email);
```

### 3. GiST (Generalized Search Tree)
- Ideal para dados geométricos e dados customizados
- Muito usado com dados espaciais
- Exemplo com dados geográficos:

```sql
CREATE INDEX idx_locais_posicao ON locais USING GIST (posicao);
```

### 4. GIN (Generalized Inverted Index)
- Perfeito para valores compostos (arrays, jsonb)
- Excelente para busca textual
- Exemplo com JSONB:

```sql
CREATE INDEX idx_documentos_conteudo ON documentos USING GIN (dados_json);
```

## Estratégias de Indexação

### Índices Compostos (Multicoluna)
Úteis quando você frequentemente pesquisa usando múltiplas colunas juntas:

```sql
-- Criando um índice composto
CREATE INDEX idx_funcionarios_dept_cargo 
ON funcionarios (departamento_id, cargo);

-- Este índice será útil para queries como:
SELECT * FROM funcionarios 
WHERE departamento_id = 5 AND cargo = 'Gerente';
```

### Índices Parciais
Indexam apenas um subconjunto dos dados, economizando espaço:

```sql
-- Indexa apenas produtos ativos
CREATE INDEX idx_produtos_ativos_preco 
ON produtos (preco) 
WHERE status = 'ativo';
```

### Índices Únicos
Garantem que não haverá valores duplicados:

```sql
-- Garantindo emails únicos
CREATE UNIQUE INDEX idx_usuarios_email 
ON usuarios (email);
```

## Manutenção e Otimização

### Análise de Uso
Para verificar quais índices estão sendo utilizados:

```sql
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,     -- número de vezes que o índice foi usado
    idx_tup_read, -- número de linhas retornadas pelo índice
    idx_tup_fetch -- número de linhas vivas buscadas pelo índice
FROM pg_stat_user_indexes;
```

### Reconstrução de Índices
Às vezes os índices podem se fragmentar e precisam ser reconstruídos:

```sql
-- Reconstruir um índice específico
REINDEX INDEX idx_nome;

-- Reconstruir todos os índices de uma tabela
REINDEX TABLE nome_tabela;
```

### Análise de Performance com EXPLAIN
Para entender como o PostgreSQL está usando os índices:

```sql
EXPLAIN ANALYZE
SELECT * FROM funcionarios 
WHERE salario BETWEEN 5000 AND 7000;
```

## Considerações Importantes

### Quando Criar Índices
- Em colunas frequentemente usadas em cláusulas WHERE
- Em colunas utilizadas em JOINs
- Em colunas que precisam ser únicas
- Em colunas frequentemente ordenadas (ORDER BY)

### Quando Evitar Índices
- Em tabelas muito pequenas
- Em colunas raramente consultadas
- Em colunas que são frequentemente atualizadas
- Em colunas com muitos valores NULL

## Monitoramento Contínuo

### Identificando Índices Não Utilizados

```sql
SELECT 
    schemaname || '.' || tablename AS table_name,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Verificando o Tamanho dos Índices

```sql
SELECT
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Exemplos Práticos

### Cenário 1: Sistema de E-commerce

```sql
-- Tabela de produtos
CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    preco DECIMAL(10,2),
    categoria_id INTEGER,
    status VARCHAR(20),
    data_cadastro TIMESTAMP
);

-- Índice para busca por categoria e preço
CREATE INDEX idx_produtos_categoria_preco 
ON produtos (categoria_id, preco);

-- Índice parcial para produtos ativos
CREATE INDEX idx_produtos_ativos 
ON produtos (nome, preco) 
WHERE status = 'ativo';

-- Índice para ordenação por data
CREATE INDEX idx_produtos_data 
ON produtos (data_cadastro DESC);
```

### Cenário 2: Sistema de Logs

```sql
-- Tabela de logs
CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP,
    nivel VARCHAR(20),
    mensagem TEXT
);

-- Índice para busca por período
CREATE INDEX idx_logs_timestamp 
ON logs (timestamp DESC);

-- Índice parcial para erros
CREATE INDEX idx_logs_erros 
ON logs (timestamp) 
WHERE nivel = 'ERROR';
```

## Dicas de Performance

1. Atualize as estatísticas regularmente:
```sql
ANALYZE tabela_nome;
```

2. Monitore o crescimento dos índices:
```sql
SELECT pg_size_pretty(pg_total_relation_size('tabela_nome'));
```

3. Use EXPLAIN ANALYZE para verificar o plano de execução:
```sql
EXPLAIN ANALYZE
SELECT * FROM produtos
WHERE categoria_id = 5
AND preco < 100;
```

## Conclusão

Os índices são ferramentas poderosas para otimização de performance no PostgreSQL, mas devem ser usados com critério. O segredo é encontrar o equilíbrio entre:
- Performance de leitura (SELECT)
- Performance de escrita (INSERT/UPDATE)
- Uso de espaço em disco
- Complexidade de manutenção
