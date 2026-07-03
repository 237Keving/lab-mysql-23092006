# lab_23092006 — Laboratório MySQL

Documentação técnica de um laboratório prático de MySQL 8.0: modelagem relacional, criação de uma **procedure** com cursor e otimização de consulta via **índice composto**, aplicados a um domínio de prontuário clínico simplificado.

**Autor:** Kevin Rhoden
**Matrícula:** 23092006
**Schema:** `lab_23092006`

## Contexto

Este repositório é a entrega de uma atividade avaliativa de banco de dados, com uso de IA como ferramenta de apoio ao longo do processo. A IA foi utilizada para:

- Estruturar e revisar o SQL (criação de schema, tabelas, chaves estrangeiras e a procedure com cursor);
- Auxiliar na análise do plano de execução (`EXPLAIN`) antes e depois da criação do índice composto;
- Gerar a página de documentação (`index.html` + `styles.css`) a partir do conteúdo técnico produzido;
- Organizar este `README.md`.

A modelagem do banco, a escrita e execução das queries no MySQL Workbench e a validação dos resultados (prints em `assets/`) foram feitas pelo autor; a IA atuou como apoio de redação, revisão e formatação, não como fonte dos dados ou resultados apresentados.

## Estrutura do projeto

```
.
├── index.html          # página de documentação do laboratório
├── styles.css           # estilos da página
├── README.md
└── assets/
    ├── explain-antes.png     # EXPLAIN antes do índice composto
    ├── explain-depois.png    # EXPLAIN depois do índice composto
    └── procedure.png         # execução da procedure sp_contar_consultas_2025
```

Para visualizar, basta abrir `index.html` em qualquer navegador (não requer servidor).

## Banco de dados

Schema com quatro tabelas relacionadas por chave estrangeira:

| Tabela | Descrição |
|---|---|
| `pacientes` | cadastro de pacientes (id, nome, cpf, data_nasc) |
| `consultas` | consultas vinculadas a um paciente (paciente_id, data_consulta, especialidade) |
| `prontuarios` | observações vinculadas a uma consulta |
| `auditoria` | tabela de apoio para registro de operações |

Chaves estrangeiras com `ON DELETE CASCADE ON UPDATE CASCADE` entre `consultas → pacientes` e `prontuarios → consultas`.

Massa de dados: 5 pacientes e 10 consultas distribuídas entre 2024 e 2025, cada uma com prontuário correspondente.

## Objeto de banco escolhido: Procedure

`sp_contar_consultas_2025(IN p_paciente_id, OUT p_total)` — usa um `CURSOR` para percorrer as consultas de um paciente no ano de 2025 e retorna a contagem. Valida a existência do `paciente_id` antes de processar; se inválido, dispara erro customizado com `SIGNAL SQLSTATE '45000'`.

## Otimização de consulta

Query alvo filtrando por `paciente_id` e `data_consulta`. Foi criado o índice composto:

```sql
CREATE INDEX idx_consultas_paciente_data
    ON consultas (paciente_id, data_consulta);
```
O `EXPLAIN` mostra a mudança de plano de execução de varredura completa (`type: ALL`) para acesso indexado (`type: ref/range`, `key: idx_consultas_paciente_data`), reduzindo o número de linhas examinadas.

## Evidências

Prints reais em `assets/`: EXPLAIN antes/depois do índice e execução da procedure.

## Ambiente

MySQL 8.0, sem etapa de deploy.