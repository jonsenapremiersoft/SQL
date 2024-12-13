### Chaves em SQL com PostgreSQL

---

## **O que são chaves em SQL?**

Chaves em SQL são atributos (ou conjunto de atributos) usados para identificar registros de maneira única em uma tabela. Elas desempenham um papel crucial na integridade dos dados e nos relacionamentos entre tabelas.

Existem dois tipos principais de chaves:  
1. **Primary Key (Chave Primária)**  
2. **Foreign Key (Chave Estrangeira)**  

---

## **1. Primary Key (Chave Primária)**

A **Chave Primária** é um identificador único para cada registro em uma tabela.  
**Características:**
- Cada tabela só pode ter uma Primary Key.
- Nenhum registro na coluna da Primary Key pode ser repetido ou `NULL`.

### **Exemplo prático**  

Vamos criar uma tabela `alunos` onde `id_aluno` será a Primary Key:

```sql
CREATE TABLE alunos (
    id_aluno SERIAL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);
```

**Explicação do código:**
- `SERIAL`: Gera automaticamente um número único.
- `PRIMARY KEY`: Define que `id_aluno` é único e obrigatório.

---

## **2. Foreign Key (Chave Estrangeira)**

A **Chave Estrangeira** é usada para criar um vínculo entre duas tabelas.  
**Características:**
- Faz referência a uma coluna de outra tabela (geralmente uma Primary Key).
- Garante que os dados relacionados existam (integridade referencial).

### **Exemplo prático**  

Vamos criar uma tabela `matriculas` que relaciona alunos e cursos:

```sql
CREATE TABLE cursos (
    id_curso SERIAL PRIMARY KEY,
    nome_curso VARCHAR(50) NOT NULL
);

CREATE TABLE matriculas (
    id_matricula SERIAL PRIMARY KEY,
    id_aluno INT REFERENCES alunos(id_aluno),
    id_curso INT REFERENCES cursos(id_curso),
    data_matricula DATE NOT NULL
);
```

**Explicação do código:**
- `REFERENCES alunos(id_aluno)`: Garante que `id_aluno` na tabela `matriculas` deve existir na tabela `alunos`.
- `REFERENCES cursos(id_curso)`: Cria a relação com a tabela `cursos`.

---

## **Integridade Referencial**

A integridade referencial assegura que as relações entre tabelas sejam consistentes:
- Não é permitido inserir uma Foreign Key que não tenha correspondência na tabela referenciada.
- Se um registro referenciado for apagado ou atualizado, é possível definir regras (como `CASCADE` ou `SET NULL`).

### **Exemplo com CASCADE**  

```sql
ALTER TABLE matriculas
ADD CONSTRAINT fk_aluno
FOREIGN KEY (id_aluno)
REFERENCES alunos(id_aluno)
ON DELETE CASCADE;
```

**Explicação:**
- `ON DELETE CASCADE`: Se um aluno for apagado da tabela `alunos`, suas matrículas também serão apagadas automaticamente.

---

## **Atividade Prática**

1. Crie as tabelas `professores` e `disciplinas`.  
   - `professores` deve ter uma Primary Key chamada `id_professor`.  
   - `disciplinas` deve ter uma Foreign Key que referencia `professores`.

2. Insira dados nas tabelas e teste os relacionamentos:
   - Tente inserir uma Foreign Key inválida.  
   - Apague um registro da tabela `professores` e veja o efeito na tabela `disciplinas`.

---

## **Resumo**

- **Primary Key**: Identifica registros únicos.
- **Foreign Key**: Conecta tabelas garantindo integridade referencial.
- Use regras como `ON DELETE CASCADE` para gerenciar alterações entre tabelas relacionadas.
