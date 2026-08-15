# Bulk Insert

[← Voltar a ETL & Integration Service](https://github.com/joycequoos/ETL_Integration_Service/blob/main/README.md)

O **Bulk Insert** é uma técnica utilizada em processos de ETL (Extract, Transform, Load) para otimizar a carga de dados em um banco de dados. Enquanto o ETL extrai dados de várias fontes e os transforma para atender aos requisitos de destino, o Bulk Insert atua especificamente na fase de carregamento, inserindo grandes volumes de dados de uma só vez em vez de registro por registro.

## Índice

- [Como o Bulk Insert ajuda no processo de ETL](#como-o-bulk-insert-ajuda-no-processo-de-etl)
- [Exemplo de Procedure](#exemplo-de-procedure)
- [Próximos passos](#próximos-passos)

---

## Como o Bulk Insert ajuda no processo de ETL

| # | Benefício | Descrição |
| --- | --- | --- |
| 1 | **Desempenho aprimorado** | É geralmente muito mais rápido do que inserir registros um por um, pois minimiza a sobrecarga do sistema ao processar lotes maiores de dados de uma só vez |
| 2 | **Redução da fragmentação de dados** | Inserir dados um por um pode levar à fragmentação no banco, impactando o desempenho das consultas. O Bulk Insert reduz esse efeito ao inserir blocos maiores de uma vez |
| 3 | **Transações eficientes** | Permite executar a inserção em uma única transação ou em transações menores, garantindo integridade e consistência dos dados |
| 4 | **Facilidade de uso** | Ferramentas de ETL geralmente já oferecem suporte ao Bulk Insert como funcionalidade embutida, facilitando sua configuração |
| 5 | **Manipulação de grandes volumes** | Essencial para carregar grandes volumes de dados no destino de forma eficiente, evitando gargalos de desempenho |
| 6 | **Suporte a formatos específicos** | Permite carregar dados a partir de arquivos externos, como CSV e TSV, útil ao importar dados de fontes externas |
| 7 | **Integridade referencial** | Pode ser usado em conjunto com validação de integridade referencial, garantindo que os dados atendam a chaves estrangeiras e restrições de chave primária |

Em resumo, o Bulk Insert é uma técnica valiosa no processo de ETL: melhora o desempenho, simplifica a carga de grandes volumes de dados e ajuda a manter a integridade dos dados durante o carregamento no banco de destino — contribuindo para um processo de ETL mais eficiente e confiável.

## Exemplo de Procedure

Exemplo de procedure que utiliza Bulk Insert para carregamento de dados:

[`01_Procedure_BulkInsert.sql`](https://github.com/joycequoos/BulkInsert/blob/main/ExemploProcedures/01_Procedure_BulkInsert.sql)

## Próximos passos

- Documentar os parâmetros de configuração do Bulk Insert (tamanho de lote, formato de arquivo, delimitadores).
- Adicionar um exemplo comparando o tempo de carga com Bulk Insert vs. inserção linha a linha.
- Descrever como tratar erros e linhas rejeitadas durante a carga em lote.
- Explorar o uso de Bulk Insert junto com validação de chaves estrangeiras antes do carregamento.
