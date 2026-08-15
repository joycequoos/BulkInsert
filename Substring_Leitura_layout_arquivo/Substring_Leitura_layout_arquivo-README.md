# Substring — Leitura de Layout de Arquivo

[← Voltar a Bulk Insert](https://github.com/joycequoos/BulkInsert/blob/main/README.md)

Exemplo prático de leitura de um arquivo de layout fixo (posicional, sem delimitador) usando `SUBSTRING` no SQL Server. A ideia é: cada linha do arquivo chega como uma única string, e a posição de cada caractere já indica a qual campo ele pertence — por isso não é possível usar `BULK INSERT` com `FIELDTERMINATOR`, sendo necessário "fatiar" a linha manualmente.

## Índice

- [O layout do arquivo](#o-layout-do-arquivo)
  - [Header](#header)
  - [Cadastro principal](#cadastro-principal)
  - [Trailer](#trailer)
- [A procedure de leitura](#a-procedure-de-leitura)
  - [Mapeamento posição → campo](#mapeamento-posição--campo)
  - [Fluxo da procedure](#fluxo-da-procedure)
- [Arquivos desta pasta](#arquivos-desta-pasta)
- [Próximos passos](#próximos-passos)

---

## O layout do arquivo

O arquivo segue o padrão **Layout de Cadastros para GLD** (base de envio ao sistema PLD), com nome no formato `PXXXICDGLD`, onde `XXX` é a sigla do sistema de origem. Ele é dividido em três blocos: **Header**, **Cadastro Principal** e **Trailer**.

### Header

| Campo | Tipo | Descrição | Obrigatório | Observação |
| --- | --- | --- | --- | --- |
| `TP_REG` | Numérico (1) | Tipo de Registro | Sim | Fixo `1` |
| `DT_ARQ` | Data (8) | Data de referência do arquivo | Sim | Formato `AAAAMMDD` |
| `SG_SIS` | Alfanumérico (3) | Sigla do Sistema que gerou o arquivo | Sim | — |

### Cadastro principal

| Campo | Tipo | Descrição | Obrigatório | Observação |
| --- | --- | --- | --- | --- |
| `TP_REG` | Numérico (1) | Tipo de Registro | Sim | Fixo `2` |
| `DC_PES` | Numérico (14) | Documento (CPF/CNPJ) da Pessoa/Empresa | Sim | — |
| `TP_PES` | Alfanumérico (1) | Tipo da Pessoa | Sim | `F` – Física / `J` – Jurídica |
| `NM_PES` | Alfanumérico (100) | Nome da Pessoa/Empresa | Sim | — |
| `CD_CEP` | Numérico (8) | Código do CEP da Pessoa/Empresa | Sim | — |
| `DS_LOG` | Alfanumérico (50) | Descrição do Logradouro | Sim | — |
| `DS_CPL` | Alfanumérico (50) | Descrição do complemento do Logradouro | Não | — |
| `DS_BAI` | Alfanumérico (50) | Descrição do Bairro | Sim | — |
| `DS_LOC` | Alfanumérico (50) | Descrição da Localidade/Cidade | Sim | — |
| `DS_UFE` | Alfanumérico (2) | Sigla da UF | Sim | — |
| `NU_TEL` | Alfanumérico (15) | Número do Telefone | Não | — |
| `TP_PEP` | Alfanumérico (1) | Indicativo de PEP (Pessoa Exposta Politicamente) | Sim | `S` – Sim / `N` – Não |
| `DS_EMT` | Alfanumérico (50) | Empresa onde a Pessoa Trabalha | Não | — |
| `DT_NAS` | Data (8) | Data de Nascimento/Constituição da Empresa | Não | Formato `AAAAMMDD` |
| `CD_PRF` | Numérico (1) | Código do Porte da Empresa na Receita Federal | Não | Sistema GCB (atributo `RGSIDPTERFB`) |
| `CD_NJU` | Numérico (3) | Código da Natureza Jurídica da Empresa | Não | Sistema GCB (atributo `NJUCD`) |
| `CD_RAT` | Numérico (4) | Código do Ramo de Atividade da Empresa | Não | Sistema GCB (atributo `RATCD`) |
| `CD_ETB` | Numérico (5) | Código do Estabelecimento Comercial | Não | Sistema GBR (atributo `ETBCD`) |
| `CD_ERA` | Numérico (4) | Código do Ramo de Atividade do Estabelecimento | Não | Sistema GCB (atributo `ETBCDRMOATV`) |
| `DS_ETB` | Alfanumérico (25) | Descrição do Estabelecimento Comercial | Não | Sistema GBR (atributo `ETBCD`) |
| `VL_FAT` | Numérico (14,2) | Faturamento da Empresa | Não | Sistema GCB (atributo `RGSVLFTNANU`) |
| `DT_CAD` | Data (8) | Data de Cadastro da Pessoa/Empresa | Não | Sistema GCB (atributo `CLNDTCDT`) |

### Trailer

| Campo | Tipo | Descrição | Obrigatório | Observação |
| --- | --- | --- | --- | --- |
| `TP_REG` | Numérico (1) | Tipo de Registro | Sim | Fixo `9` |
| `QT_RGT` | Numérico (15) | Quantidade de Registros Totais do Arquivo | Sim | — |

> Layout completo em [`Layout_IN74_Arquivo de Cadastro.docx`](https://github.com/joycequoos/BulkInsert/blob/main/Substring_Leitura_layout_arquivo/Layout_IN74_Arquivo%20de%20Cadastro.docx).

## A procedure de leitura

A procedure [`spct_cad_custodia`](https://github.com/joycequoos/BulkInsert/blob/main/Substring_Leitura_layout_arquivo/01_Substring_Layout_Delimitador.sql) recebe uma data de referência e lê as linhas do bloco **Cadastro Principal** (linhas cujo primeiro caractere é `2`), fatiando cada linha nas posições exatas definidas no layout.

### Mapeamento posição → campo

| Campo | `SUBSTRING(dc_linha, início, tamanho)` |
| --- | --- |
| `DC_PES` | `SUBSTRING(dc_linha, 2, 14)` |
| `TP_PES` | `SUBSTRING(dc_linha, 16, 1)` |
| `NM_PES` | `SUBSTRING(dc_linha, 17, 100)` |
| `CD_CEP` | `SUBSTRING(dc_linha, 117, 8)` |
| `DS_LOG` | `SUBSTRING(dc_linha, 125, 50)` |
| `DS_CPL` | `SUBSTRING(dc_linha, 175, 50)` |
| `DS_BAI` | `SUBSTRING(dc_linha, 225, 50)` |
| `DS_LOC` | `SUBSTRING(dc_linha, 275, 50)` |
| `DS_UFE` | `SUBSTRING(dc_linha, 325, 2)` |
| `NU_TEL` | `SUBSTRING(dc_linha, 327, 15)` |
| `TP_PEP` | `SUBSTRING(dc_linha, 342, 1)` |
| `DS_EMT` | `SUBSTRING(dc_linha, 343, 50)` |
| `DT_NAS` | `SUBSTRING(dc_linha, 393, 8)` (convertida para `datetime`) |
| `CD_PRF` | `SUBSTRING(dc_linha, 401, 1)` |
| `CD_NJU` | `SUBSTRING(dc_linha, 402, 3)` |
| `CD_RAT` | `SUBSTRING(dc_linha, 405, 4)` |
| `CD_ETB` | `SUBSTRING(dc_linha, 409, 5)` |
| `CD_ERA` | `SUBSTRING(dc_linha, 414, 4)` |
| `DS_ETB` | `SUBSTRING(dc_linha, 418, 25)` |
| `VL_FAT` | `SUBSTRING(dc_linha, 443, 14)` (convertida para `decimal(14,2)`) |
| `DT_CAD` | `SUBSTRING(dc_linha, 459, 8)` (convertida para `datetime`) |

> As posições e tamanhos batem exatamente com o layout do bloco **Cadastro Principal** descrito no `.docx` — cada `SUBSTRING` "corta" a linha do arquivo no ponto exato em que cada campo começa e termina.

### Fluxo da procedure

1. **Início da carga** — registra um novo controle em `dbo.tgr_cargas`, com o nome do arquivo esperado (`AAAAMMDD` + `CAD_CUSTODIA.TXT`) e a hora de início.
2. **Limpeza da temporária** — esvazia `dbo.ttp_cadastro_custodia` antes de receber os novos dados.
3. **Fatiamento (Substring)** — lê `dbo.ttp_linha_cadastro_custodia` (onde cada linha do arquivo já foi carregada como uma string única) e, para as linhas de cadastro (`TP_REG = '2'`), aplica os `SUBSTRING` da tabela acima para popular `dbo.ttp_cadastro_custodia` com os campos já separados.
4. **Inserção de novos cadastros** — insere em `dbo.tct_cadastro_custodia` (tabela definitiva) os documentos (`DC_PES`) que ainda não existem.
5. **Atualização de cadastros existentes** — para os documentos já cadastrados, atualiza os dados com as informações mais recentes vindas do arquivo.
6. **Fim da carga** — atualiza o registro de controle em `dbo.tgr_cargas` com a hora de término e o status de sucesso.
7. **Tratamento de erro** — em caso de falha em qualquer etapa, o bloco `CATCH` aciona a procedure `dbo.spgr_tratar_erro` para registrar o problema.

## Arquivos desta pasta

| Arquivo | Descrição |
| --- | --- |
| [`01_Substring_Layout_Delimitador.sql`](https://github.com/joycequoos/BulkInsert/blob/main/Substring_Leitura_layout_arquivo/01_Substring_Layout_Delimitador.sql) | Procedure `spct_cad_custodia`, que lê o arquivo posicional e carrega o cadastro |
| [`Layout_IN74_Arquivo de Cadastro.docx`](https://github.com/joycequoos/BulkInsert/blob/main/Substring_Leitura_layout_arquivo/Layout_IN74_Arquivo%20de%20Cadastro.docx) | Documento com o layout completo do arquivo (Header, Cadastro Principal e Trailer) |

## Próximos passos

- Adicionar uma etapa de validação do `Header` e do `Trailer` (ex.: conferir se `QT_RGT` bate com a quantidade de linhas de cadastro lidas).
- Tratar o caso de campos numéricos vazios antes da conversão (`CONVERT`/`CAST`), para evitar erro de conversão em produção.
- Documentar exemplos de linha de arquivo real (mascarados) para facilitar testes.
- Avaliar o uso de `TRY_CONVERT`/`TRY_CAST` para tornar a carga mais resiliente a linhas malformadas.
