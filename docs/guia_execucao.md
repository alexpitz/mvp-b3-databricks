# Guia de execução no Databricks Free Edition

## 1. Publicar no GitHub

1. Crie um repositório público, por exemplo `mvp-b3-databricks`.
2. Envie todo o conteúdo deste projeto, exceto os dados.
3. Confirme em uma janela anônima que o README e os notebooks estão públicos.

## 2. Conectar ao Databricks

1. No workspace, abra **Workspace > Create > Git folder**.
2. Informe a URL HTTPS do repositório público.
3. Use a branch `main`.
4. Se solicitado, configure uma credencial GitHub em **User settings > Linked accounts**.

## 3. Preparar dados

No Hub de Dados Públicos da B3, baixe o COTAHIST anual de 2025 e descompacte o TXT. Prepare também `setores_b3.csv` em UTF-8 com este cabeçalho:

```csv
ticker,empresa,setor,subsetor,segmento
ABCD3,Empresa Exemplo,Financeiro,Bancos,Bancos
```

O exemplo acima é apenas estrutural e não deve ser tratado como dado real.

## 4. Configurar e executar

1. Importe/abra `notebooks/00_setup.py`.
2. Ajuste `catalog`, `schema` e `volume` nos widgets, se necessário.
3. Execute 00 e envie os arquivos para as pastas exibidas.
4. Execute 01, 02, 03, 04 e 05 nesta ordem.
5. Opcionalmente, crie um Workflow com uma task por notebook e dependências lineares.

## 5. Capturar evidências

Use nomes idênticos aos placeholders do README. As imagens devem mostrar o contexto suficiente (nome da tabela/notebook, resultado e status), sem expor tokens ou dados pessoais.

1. Volume e arquivos de entrada.
2. Catalog Explorer com schemas/tabelas.
3. Colunas e comentários de tabela Gold.
4. Lineage.
5. Workflow concluído.
6. `SHOW TABLES` e contagens.
7. Resultados de qualidade.
8. Gráficos/tabelas das três perguntas.

## 6. Fechar a documentação

Copie para o README os resultados numéricos mostrados pelo notebook 05. Revise a autoavaliação em primeira pessoa e confirme que todas as imagens aparecem no GitHub.

