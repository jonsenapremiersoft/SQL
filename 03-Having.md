# Having

O **`HAVING`** em SQL é usado para filtrar resultados de agregações, como aquelas geradas por funções como `SUM`, `AVG`, `COUNT`, `MAX` ou `MIN`. Ele é aplicado após o agrupamento realizado pelo `GROUP BY`. 

Por outro lado, o **`WHERE`** é usado para filtrar linhas antes que qualquer agrupamento ou agregação ocorra.

### Diferença essencial:
- **`WHERE`**: Filtra as linhas *antes* do agrupamento.
- **`HAVING`**: Filtra os grupos *após* o agrupamento.

### Exemplo:
Imagine uma tabela `vendas` com as colunas `produto`, `categoria` e `valor`.

#### 1. Usando `WHERE`:
Se você quiser filtrar apenas vendas com valor acima de 100:
```sql
SELECT produto, categoria, valor
FROM vendas
WHERE valor > 100;
```
Aqui, o filtro é feito diretamente nas linhas antes de qualquer agrupamento.

#### 2. Usando `HAVING`:
Agora, suponha que você quer calcular a soma total das vendas por categoria, mas exibir apenas as categorias onde a soma das vendas ultrapassa 500:
```sql
SELECT categoria, SUM(valor) AS total_vendas
FROM vendas
GROUP BY categoria
HAVING SUM(valor) > 500;
```
Nesse caso, o filtro ocorre *após* o agrupamento, pois depende do resultado de `SUM(valor)`.

### Resumindo:
- Use **`WHERE`** para filtrar linhas antes de agrupar.
- Use **`HAVING`** para filtrar agregações (resultados de `GROUP BY`).
