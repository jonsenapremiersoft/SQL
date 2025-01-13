# Automatizando Operações no PostgreSQL

### Por que Automatizar Operações no Banco de Dados?

Imagine que você trabalha em uma padaria. Todo dia você precisa:
- Conferir o estoque
- Registrar as vendas
- Calcular o lucro
- Anotar quais produtos estão acabando

Fazer isso manualmente seria muito trabalhoso, não é? No banco de dados é a mesma coisa! Por isso temos ferramentas que nos ajudam a automatizar essas tarefas.

## 1. Functions (Funções) - Nossa Calculadora Especial

### O que é uma Function?
Uma function é como uma calculadora personalizada que você programa para fazer cálculos específicos. Por exemplo, ao invés de toda vez você precisar calcular manualmente o desconto de um produto, você cria uma function que faz isso para você!

### Exemplo da Padaria
Vamos criar uma function que calcula o preço final dos pães considerando a quantidade e possíveis descontos:

```sql
CREATE OR REPLACE FUNCTION calcular_preco_paes(
    quantidade INT,
    preco_unitario DECIMAL
)
RETURNS DECIMAL AS $$
DECLARE
    preco_total DECIMAL;
    desconto DECIMAL;
BEGIN
    -- Primeiro calculamos o preço total
    preco_total := quantidade * preco_unitario;
    
    -- Depois aplicamos o desconto baseado na quantidade
    CASE 
        WHEN quantidade >= 50 THEN
            desconto := 0.20; -- 20% de desconto para 50 ou mais pães
        WHEN quantidade >= 20 THEN
            desconto := 0.10; -- 10% de desconto para 20 ou mais pães
        WHEN quantidade >= 10 THEN
            desconto := 0.05; -- 5% de desconto para 10 ou mais pães
        ELSE
            desconto := 0;    -- Sem desconto
    END CASE;
    
    -- Retornamos o preço com desconto
    RETURN preco_total - (preco_total * desconto);
END;
$$ LANGUAGE plpgsql;

-- Como usar:
SELECT calcular_preco_paes(15, 0.50); -- Calcula o preço de 15 pães a R$0,50 cada
```

## 2. Procedures - Nossa Lista de Tarefas Automática

### O que é um Procedure?
Um procedure é como uma lista de tarefas que o banco de dados vai executar em sequência. É como se você criasse um roteiro: "Primeiro faça isso, depois aquilo, e por fim faça aquilo outro".

### Exemplo da Padaria
Vamos criar um procedure que registra a venda de pães e atualiza o estoque:

```sql
CREATE OR REPLACE PROCEDURE registrar_venda_paes(
    p_tipo_pao VARCHAR,
    p_quantidade INT,
    p_valor_unitario DECIMAL
)
LANGUAGE plpgsql AS $$
DECLARE
    valor_total DECIMAL;
BEGIN
    -- Calcula o valor total usando nossa function anterior
    valor_total := calcular_preco_paes(p_quantidade, p_valor_unitario);
    
    -- Registra a venda
    INSERT INTO vendas_diarias (
        tipo_pao,
        quantidade,
        valor_unitario,
        valor_total,
        data_venda
    ) VALUES (
        p_tipo_pao,
        p_quantidade,
        p_valor_unitario,
        valor_total,
        CURRENT_TIMESTAMP
    );
    
    -- Atualiza o estoque
    UPDATE estoque_paes 
    SET quantidade = quantidade - p_quantidade
    WHERE tipo_pao = p_tipo_pao;
    
    -- Se o estoque ficar baixo, registra um alerta
    INSERT INTO alertas_estoque (
        tipo_pao,
        quantidade_atual,
        data_alerta
    )
    SELECT 
        tipo_pao,
        quantidade,
        CURRENT_TIMESTAMP
    FROM estoque_paes
    WHERE tipo_pao = p_tipo_pao
    AND quantidade < 50;
    
    COMMIT;
END;
$$;

-- Como usar:
CALL registrar_venda_paes('Pão Francês', 20, 0.50);
```

## 3. Triggers - Nosso Vigilante Automático

### O que é um Trigger?
Um trigger é como um vigilante que fica observando o banco de dados. Quando algo específico acontece (como uma venda ou alteração de estoque), ele automaticamente executa uma ação que definimos.

### Exemplo da Padaria
Vamos criar um trigger que monitora o estoque e envia alertas automáticos:

```sql
-- Primeiro criamos uma tabela para os alertas
CREATE TABLE IF NOT EXISTS log_movimentacao_estoque (
    id SERIAL PRIMARY KEY,
    tipo_pao VARCHAR(50),
    quantidade_anterior INT,
    quantidade_nova INT,
    tipo_movimento VARCHAR(20),
    data_movimento TIMESTAMP,
    responsavel VARCHAR(50)
);

-- Criamos a função que o trigger vai executar
CREATE OR REPLACE FUNCTION monitorar_estoque()
RETURNS TRIGGER AS $$
BEGIN
    -- Registra qualquer mudança no estoque
    INSERT INTO log_movimentacao_estoque (
        tipo_pao,
        quantidade_anterior,
        quantidade_nova,
        tipo_movimento,
        data_movimento,
        responsavel
    ) VALUES (
        NEW.tipo_pao,
        OLD.quantidade,
        NEW.quantidade,
        CASE 
            WHEN NEW.quantidade > OLD.quantidade THEN 'ENTRADA'
            ELSE 'SAÍDA'
        END,
        CURRENT_TIMESTAMP,
        CURRENT_USER
    );
    
    -- Verifica se precisa gerar alerta de estoque baixo
    IF NEW.quantidade < 50 THEN
        INSERT INTO alertas_estoque (
            tipo_pao,
            quantidade_atual,
            data_alerta,
            mensagem
        ) VALUES (
            NEW.tipo_pao,
            NEW.quantidade,
            CURRENT_TIMESTAMP,
            'ATENÇÃO: Estoque baixo!'
        );
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Criamos o trigger
CREATE TRIGGER trigger_monitoramento_estoque
AFTER UPDATE ON estoque_paes
FOR EACH ROW
EXECUTE FUNCTION monitorar_estoque();
```

## Simplificando:

1. **Function** é como uma calculadora:
   - Você dá os números para ela
   - Ela faz os cálculos
   - Devolve o resultado

2. **Procedure** é como uma receita de bolo:
   - Tem uma lista de passos para seguir
   - Executa cada passo na ordem
   - Faz várias coisas de uma vez

3. **Trigger** é como um alarme:
   - Fica vigiando o banco de dados
   - Quando algo específico acontece, dispara
   - Executa ações automáticas

## 🎯 Desafio Prático: Sistema de Uma Livraria

Você foi contratado para criar um sistema para uma livraria. Desenvolva:

1. Uma **function** que:
   - Calcule o preço final de um livro considerando:
     - Desconto por quantidade
     - Desconto para estudantes (10%)
     - Desconto para livros com mais de 6 meses em estoque (15%)

2. Um **procedure** que:
   - Registre a venda de um livro
   - Atualize o estoque
   - Registre pontos de fidelidade para o cliente (1 ponto para cada R$ 10 em compras)
   - Gere um cupom de desconto se o cliente atingir 100 pontos

3. Um **trigger** que:
   - Monitore o estoque de livros
   - Gere alertas para livros com menos de 5 unidades
   - Registre em uma tabela de auditoria todas as vendas acima de R$ 200
   - Marque automaticamente para promoção livros que não são vendidos há mais de 3 meses

### Dicas para o Desafio:
- Comece criando as tabelas necessárias
- Desenvolva e teste uma funcionalidade por vez
- Use comentários para documentar seu código
- Pense em validações importantes (ex: estoque não pode ficar negativo)

Lembre-se: Não existe resposta única correta! O importante é que seu código funcione e atenda aos requisitos.

---

### Card 1: Modelagem do Banco de Dados
**Prioridade: Alta**
**Estimativa: 5 pontos**

**Descrição:**  
Criar a estrutura inicial do banco de dados para suportar o sistema da livraria.

**Requisitos:**
1. Criar tabela de livros com:
   - ID, título, autor, preço, quantidade em estoque, data de entrada
2. Criar tabela de clientes com:
   - ID, nome, tipo (estudante/regular), pontos_fidelidade
3. Criar tabela de vendas com:
   - ID, livro_id, cliente_id, quantidade, valor_total, data_venda
4. Criar tabela de cupons com:
   - ID, cliente_id, valor, data_validade, status
5. Criar tabela de auditoria com:
   - ID, tipo_operacao, data, detalhes, valor_venda
6. Criar tabela de alertas_estoque com:
   - ID, livro_id, quantidade, data_alerta, tipo_alerta

**Critérios de Aceite:**
- Todas as tabelas devem ter chaves primárias e estrangeiras apropriadas
- Índices devem ser criados para campos frequentemente consultados
