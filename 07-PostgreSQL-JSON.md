# PostgreSQL e JSON

## Introdução ao JSON no PostgreSQL

O JSON (JavaScript Object Notation) é um formato de dados leve e legível que se tornou um padrão para troca de informações em aplicações modernas. O PostgreSQL oferece suporte nativo ao JSON desde a versão 9.2, com melhorias significativas em versões posteriores.

## Tipos de Dados JSON

### 1. Tipo JSON
O tipo `json` armazena uma cópia exata do texto JSON entrada:
- Mantém espaços em branco
- Mantém a ordem das chaves
- Requer parsing a cada operação
- Útil quando você precisa manter o documento exatamente como foi inserido

```sql
-- Exemplo de coluna json
CREATE TABLE registros (
    id SERIAL PRIMARY KEY,
    documento json
);
```

### 2. Tipo JSONB
O tipo `jsonb` é mais eficiente:
- Remove espaços em branco
- Reordena as chaves
- Armazena em formato binário
- Mais rápido para processar
- Suporta indexação
- Recomendado para a maioria dos casos

```sql
-- Exemplo de coluna jsonb
CREATE TABLE registros_otimizados (
    id SERIAL PRIMARY KEY,
    documento jsonb
);
```

## Criando Estruturas com JSON

### Tabela de Exemplo Completa

```sql
CREATE TABLE loja_virtual (
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    dados_produto jsonb,
    historico_precos json,
    metadados jsonb
);

-- Adicionando restrições
ALTER TABLE loja_virtual 
ADD CONSTRAINT dados_produto_check 
CHECK (jsonb_typeof(dados_produto) = 'object');
```

## Operações de Inserção

### Inserção Básica

```sql
INSERT INTO loja_virtual (dados_produto) VALUES (
    '{
        "nome": "Monitor UltraWide",
        "sku": "MON-UW-001",
        "preco": 2499.99,
        "especificacoes": {
            "tamanho": "34 polegadas",
            "resolucao": "3440x1440",
            "taxa_atualizacao": 144,
            "portas": {
                "hdmi": 2,
                "displayport": 1,
                "usb": 4
            }
        },
        "cores_disponiveis": ["preto", "prata"],
        "em_estoque": true
    }'::jsonb
);
```

### Inserção com Arrays

```sql
INSERT INTO loja_virtual (dados_produto, historico_precos) VALUES (
    '{
        "nome": "Teclado Mecânico RGB",
        "sku": "TEC-MEC-001",
        "especificacoes": {
            "switches": "Blue",
            "layout": "ABNT2",
            "recursos": ["RGB", "USB Pass-through", "Macro Keys"]
        }
    }'::jsonb,
    '[
        {"data": "2024-01-01", "preco": 499.99},
        {"data": "2024-02-01", "preco": 459.99},
        {"data": "2024-03-01", "preco": 479.99}
    ]'::json
);
```

## Operadores JSON

### 1. Operadores de Acesso

```sql
-- Operador -> (retorna JSON)
SELECT 
    dados_produto->'nome' as nome_json,
    dados_produto->'especificacoes'->'tamanho' as tamanho_json
FROM loja_virtual;

-- Operador ->> (retorna texto)
SELECT 
    dados_produto->>'nome' as nome_texto,
    dados_produto->'especificacoes'->>'tamanho' as tamanho_texto
FROM loja_virtual;

-- Acessando arrays
SELECT 
    dados_produto->'cores_disponiveis'->0 as primeira_cor,
    dados_produto->'especificacoes'->'portas'->>'hdmi' as qtd_hdmi
FROM loja_virtual;
```

### 2. Operadores de Contenção e Existência

```sql
-- @> verifica se contém
SELECT * FROM loja_virtual 
WHERE dados_produto @> '{"especificacoes": {"resolucao": "3440x1440"}}';

-- ? verifica se chave existe
SELECT * FROM loja_virtual 
WHERE dados_produto ? 'em_estoque';

-- ?| verifica se qualquer chave existe
SELECT * FROM loja_virtual 
WHERE dados_produto ?| array['preco', 'desconto'];

-- ?& verifica se todas as chaves existem
SELECT * FROM loja_virtual 
WHERE dados_produto ?& array['nome', 'sku'];
```

## Funções JSON Avançadas

### 1. Manipulação de Estrutura

```sql
-- Construindo JSON
SELECT jsonb_build_object(
    'nome', dados_produto->>'nome',
    'preco_atual', dados_produto->>'preco',
    'especificacoes', dados_produto->'especificacoes'
)
FROM loja_virtual;

-- Expandindo arrays
SELECT jsonb_array_elements(dados_produto->'cores_disponiveis')
FROM loja_virtual;

-- Obtendo chaves
SELECT jsonb_object_keys(dados_produto->'especificacoes')
FROM loja_virtual;
```

### 2. Modificação de Dados

```sql
-- Atualizando valores
UPDATE loja_virtual 
SET dados_produto = jsonb_set(
    dados_produto,
    '{preco}',
    '2599.99',
    true
)
WHERE dados_produto->>'sku' = 'MON-UW-001';

-- Adicionando novos campos
UPDATE loja_virtual 
SET dados_produto = dados_produto || 
    '{"desconto_ativo": true, "percentual_desconto": 10}'::jsonb
WHERE dados_produto->>'sku' = 'MON-UW-001';
```

## Indexação Avançada

### 1. Índice GIN Completo

```sql
-- Índice para toda a coluna
CREATE INDEX idx_dados_produto 
ON loja_virtual USING GIN (dados_produto);
```

### 2. Índice GIN Específico

```sql
-- Índice para caminhos específicos
CREATE INDEX idx_specs 
ON loja_virtual USING GIN ((dados_produto->'especificacoes'));

-- Índice para operador de contenção
CREATE INDEX idx_produto_specs 
ON loja_virtual USING GIN (dados_produto jsonb_path_ops);
```

## Exercícios Práticos

### Exercício 1: Sistema de Perfil de Usuários

```sql
-- 1. Criar a estrutura
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    perfil jsonb NOT NULL,
    configuracoes jsonb NOT NULL DEFAULT '{}'::jsonb,
    historico_acessos json
);

-- 2. Inserir dados de exemplo
INSERT INTO usuarios (perfil, configuracoes) VALUES
(
    '{
        "nome": "Maria Silva",
        "email": "maria@email.com",
        "idade": 28,
        "endereco": {
            "rua": "Rua das Flores",
            "numero": 123,
            "cidade": "São Paulo",
            "estado": "SP"
        },
        "interesses": ["tecnologia", "fotografia", "viagens"]
    }',
    '{
        "tema": "escuro",
        "notificacoes": {
            "email": true,
            "push": false
        },
        "privacidade": {
            "perfil_publico": true,
            "mostrar_email": false
        }
    }'
);

-- 3. Consultas práticas
-- Encontrar usuários por interesse
SELECT 
    perfil->>'nome' as nome,
    perfil->>'email' as email
FROM usuarios
WHERE perfil->'interesses' ? 'fotografia';

-- Encontrar usuários com configurações específicas
SELECT 
    perfil->>'nome' as nome
FROM usuarios
WHERE configuracoes @> '{"tema": "escuro"}';

-- 4. Atualização de preferências
UPDATE usuarios 
SET configuracoes = jsonb_set(
    configuracoes,
    '{notificacoes,push}',
    'true',
    true
)
WHERE perfil->>'email' = 'maria@email.com';

-- 5. Criação de índices
CREATE INDEX idx_perfil_interesses 
ON usuarios USING GIN ((perfil->'interesses'));

CREATE INDEX idx_configuracoes 
ON usuarios USING GIN (configuracoes);
```

### Exercício 2: Sistema de Produtos com Variações

```sql
-- 1. Criar estrutura
CREATE TABLE produtos_variacoes (
    id SERIAL PRIMARY KEY,
    base_produto jsonb,
    variacoes jsonb,
    metricas json
);

-- 2. Inserir produto com variações
INSERT INTO produtos_variacoes (base_produto, variacoes) VALUES
(
    '{
        "nome": "Camiseta Básica",
        "marca": "FashionStyle",
        "categoria": "Vestuário",
        "descricao": "Camiseta 100% algodão",
        "preco_base": 49.90
    }',
    '{
        "cores": {
            "preto": {
                "codigo": "#000000",
                "estoque": 50
            },
            "branco": {
                "codigo": "#FFFFFF",
                "estoque": 30
            },
            "azul": {
                "codigo": "#0000FF",
                "estoque": 20
            }
        },
        "tamanhos": {
            "P": {"medidas": {"largura": 45, "altura": 65}},
            "M": {"medidas": {"largura": 50, "altura": 70}},
            "G": {"medidas": {"largura": 55, "altura": 75}}
        }
    }'
);

-- 3. Consultas complexas
-- Produtos com estoque baixo por cor
SELECT 
    base_produto->>'nome' as produto,
    cores.key as cor,
    cores.value->>'estoque' as estoque
FROM 
    produtos_variacoes,
    jsonb_each(variacoes->'cores') cores
WHERE 
    (cores.value->>'estoque')::integer < 30;

-- 4. Atualização de estoque
UPDATE produtos_variacoes 
SET variacoes = jsonb_set(
    variacoes,
    '{cores,preto,estoque}',
    '45',
    false
)
WHERE base_produto->>'nome' = 'Camiseta Básica';

-- 5. Índices específicos
CREATE INDEX idx_produto_cores 
ON produtos_variacoes USING GIN ((variacoes->'cores'));
```

### Exercício 3: Sistema de Logs de Eventos

```sql
-- 1. Criar estrutura
CREATE TABLE eventos_sistema (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    evento jsonb,
    metadados jsonb
);

-- 2. Função para registrar eventos
CREATE OR REPLACE FUNCTION registrar_evento(
    p_tipo TEXT,
    p_dados jsonb,
    p_meta jsonb DEFAULT '{}'::jsonb
) RETURNS void AS $$
BEGIN
    INSERT INTO eventos_sistema (evento, metadados)
    VALUES (
        jsonb_build_object(
            'tipo', p_tipo,
            'dados', p_dados,
            'timestamp', CURRENT_TIMESTAMP
        ),
        p_meta
    );
END;
$$ LANGUAGE plpgsql;

-- 3. Registrar eventos de exemplo
SELECT registrar_evento(
    'login',
    '{
        "usuario_id": 1,
        "ip": "192.168.1.1",
        "dispositivo": "Mozilla/5.0",
        "sucesso": true
    }'::jsonb,
    '{
        "ambiente": "producao",
        "versao_api": "2.1"
    }'::jsonb
);

-- 4. Consultas analíticas
-- Eventos por tipo
SELECT 
    evento->>'tipo' as tipo_evento,
    COUNT(*) as quantidade,
    MIN(timestamp) as primeiro_evento,
    MAX(timestamp) as ultimo_evento
FROM eventos_sistema
GROUP BY evento->>'tipo';

-- 5. Índices para análise
CREATE INDEX idx_evento_tipo 
ON eventos_sistema ((evento->>'tipo'));

CREATE INDEX idx_evento_timestamp 
ON eventos_sistema USING BRIN (timestamp);
```

## Boas Práticas

1. **Estruturação de Dados**
   - Mantenha uma estrutura consistente para documentos do mesmo tipo
   - Documente a estrutura esperada do JSON
   - Use enums ou constantes para valores que se repetem

2. **Performance**
   - Use `jsonb` para dados que serão consultados frequentemente
   - Crie índices apropriados (GYN) para os padrões de consulta mais comuns
   - Evite documentos JSON muito grandes (considere normalização)

3. **Manutenção**
   - Implemente validações na aplicação antes de inserir dados
   - Mantenha um histórico de mudanças estruturais
   - Use versionamento para mudanças no esquema JSON

4. **Segurança**
   - Valide entrada de dados para prevenir injeção de JSON malicioso
   - Considere permissões em nível de campo para dados sensíveis
   - Implemente auditoria para mudanças em dados críticos
