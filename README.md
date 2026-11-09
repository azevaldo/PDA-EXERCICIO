# Modelo Normalizado — CursoÁgil

## 1. Descrição do cenário

O **CursoÁgil** é uma plataforma de cursos online. Cada curso tem nome e carga horária e é dado por um professor (que tem nome e email e pode dar vários cursos). Cada curso tem várias aulas, numeradas, com título e duração em minutos. Os alunos têm nome, email e um ou mais telefones. Um aluno se matricula em vários cursos e cada curso recebe vários alunos; na matrícula, a plataforma registra a data e a nota final. O aluno também marca como concluídas as aulas que já assistiu, e a plataforma guarda quando concluiu cada uma.

## 2. Entidades e relacionamentos identificados

| Relacionamento | Entidades | Cardinalidade | Como foi resolvido |
|---|---|---|---|
| ministra | PROFESSOR → CURSO | 1 : N | FK `id_professor` em CURSO |
| possui | CURSO → AULA | 1 : N | AULA é entidade fraca: PK composta (`id_curso`, `numero`) |
| tem | ALUNO → TELEFONE | 1 : N | Atributo multivalorado vira a tabela TELEFONE_ALUNO |
| matricula-se | ALUNO ↔ CURSO | N : M | Tabela associativa MATRICULA (com `data_matricula` e `nota_final`) |
| conclui | MATRICULA ↔ AULA | N : M | Tabela associativa AULA_CONCLUIDA (com `data_hora_conclusao`) |

> Legenda: **PK** = chave primária · **FK** = chave estrangeira

## 3. Tabelas finais

### PROFESSOR

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_professor | INT | **PK** | auto incremento |
| nome | VARCHAR(100) | | NOT NULL |
| email | VARCHAR(150) | | NOT NULL, UNIQUE |

### CURSO

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_curso | INT | **PK** | auto incremento |
| nome | VARCHAR(150) | | NOT NULL |
| carga_horaria | INT | | NOT NULL, > 0 (em horas) |
| id_professor | INT | **FK** → PROFESSOR(id_professor) | NOT NULL |

### AULA

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_curso | INT | **PK**, **FK** → CURSO(id_curso) | NOT NULL |
| numero | INT | **PK** | NOT NULL, > 0 |
| titulo | VARCHAR(150) | | NOT NULL |
| duracao_minutos | INT | | NOT NULL, > 0 |

PK composta: (`id_curso`, `numero`) — a numeração recomeça em cada curso (Aula 1, Aula 2…).

### ALUNO

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_aluno | INT | **PK** | auto incremento |
| nome | VARCHAR(100) | | NOT NULL |
| email | VARCHAR(150) | | NOT NULL, UNIQUE |

### TELEFONE_ALUNO

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_aluno | INT | **PK**, **FK** → ALUNO(id_aluno) | NOT NULL |
| telefone | VARCHAR(20) | **PK** | NOT NULL |

PK composta: (`id_aluno`, `telefone`) — o mesmo telefone não se repete para o mesmo aluno.

### MATRICULA

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_aluno | INT | **PK**, **FK** → ALUNO(id_aluno) | NOT NULL |
| id_curso | INT | **PK**, **FK** → CURSO(id_curso) | NOT NULL |
| data_matricula | DATE | | NOT NULL |
| nota_final | DECIMAL(4,2) | | NULL (só é preenchida no fim do curso) |

PK composta: (`id_aluno`, `id_curso`) — um aluno se matricula uma vez em cada curso.

### AULA_CONCLUIDA

| Coluna | Tipo | Chave | Restrições |
|---|---|---|---|
| id_aluno | INT | **PK**, **FK** | NOT NULL |
| id_curso | INT | **PK**, **FK** | NOT NULL |
| numero_aula | INT | **PK**, **FK** | NOT NULL |
| data_hora_conclusao | DATETIME | | NOT NULL |

PK composta: (`id_aluno`, `id_curso`, `numero_aula`).
FKs compostas:
- (`id_aluno`, `id_curso`) → MATRICULA(id_aluno, id_curso) — garante que o aluno só conclui aulas de cursos em que está matriculado.
- (`id_curso`, `numero_aula`) → AULA(id_curso, numero).

## 4. Verificação das formas normais

- **1FN** — Todos os atributos são atômicos. O telefone (multivalorado) foi separado em TELEFONE_ALUNO; não há grupos repetidos.
- **2FN** — Nas tabelas com PK composta, os atributos dependem da chave inteira:
  - AULA: `titulo` e `duracao_minutos` dependem de (`id_curso`, `numero`), não só do curso.
  - MATRICULA: `data_matricula` e `nota_final` dependem de (aluno, curso) juntos.
  - AULA_CONCLUIDA: `data_hora_conclusao` depende de (aluno, curso, aula) juntos.
- **3FN** — Não há dependências transitivas: os dados do professor (nome, email) ficam só em PROFESSOR, e CURSO guarda apenas a FK `id_professor`; os dados do aluno ficam só em ALUNO.

## 5. Resumo (notação textual)

```
PROFESSOR      (id_professor PK, nome, email)
CURSO          (id_curso PK, nome, carga_horaria, id_professor FK)
AULA           (id_curso PK/FK, numero PK, titulo, duracao_minutos)
ALUNO          (id_aluno PK, nome, email)
TELEFONE_ALUNO (id_aluno PK/FK, telefone PK)
MATRICULA      (id_aluno PK/FK, id_curso PK/FK, data_matricula, nota_final)
AULA_CONCLUIDA (id_aluno PK/FK, id_curso PK/FK, numero_aula PK/FK, data_hora_conclusao)
```
