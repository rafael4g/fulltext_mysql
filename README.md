# FULLTEXT MYSQL

## Como funciona a pesquisa

Para cada registro, o MySQL atribui um valor de relevância, que representa a similaridade da string de pesquisa com a linha em questão. Um valor de relevância 0 (zero) significa nenhuma semelhança, fazendo com que o registro não seja exibido. O cálculo de relevância é feito através de um algoritmo projetado para pesquisa em grandes massas de texto, tornando a busca inadequada para pequenas tabelas. Entre as variáveis que são levadas em consideração nesse cálculo, o MySQL considera o número de palavras encontradas em cada campo do índice, o número de palavras encontradas por linha, o número de ocorrências da mesma palavra em todas as linhas, entre outros. Quanto mais rara for a palavra, maior será seu peso no cálculo da relevância.

```sql
# Criando a indexação nos campos desejados.
CREATE FULLTEXT INDEX INDEX_FT_DESCRICAO ON websites (titulo,descricao) ;

# Estrutura da Tabela.
CREATE TABLE websites(
post_id mediumint(8) unsigned NOT NULL,
titulo varchar(100) NOT NULL,
descricao text,
PRIMARY KEY (post_id),
FULLTEXT (titulo, descricao));

# Modo de pesquisa com MATCH - AGAINST.
# Para efetuar a pesquisa através de um índice fulltext utilizamos as funções MATCH e AGAINST, que recebem o nome dos campos e o valor a ser  pesquisado, respectivamente. Veja o exemplo:
SELECT titulo, descricao
FROM websites
WHERE MATCH (titulo, descricao) AGAINST ('SQL Magazine');

# Para exibir o valor de relevância atribuído para cada linha, podemos inserir o comando MATCH na lista de campos:
# Repare que a função MATCH( ) foi utilizada duas vezes. Não se preocupe, pois o otimizador de consultas percebe que as funções são idênticas e a chamada ocorre apenas uma vez. Observe mais alguns exemplos:
SELECT titulo, MATCH (titulo,descricao) AGAINST ('Java Magazine')
FROM websites
WHERE MATCH (titulo, descricao) AGAINST ('Java Magazine');
```

### MATCH vs LIKE

Diversos aspectos diferenciam o mecanismo de MATCH do uso de LIKE. Vejamos:

O comando MATCH é mais veloz, tendo em vista a indexação de cada palavra do campo que faz parte do índice fulltext.
A pesquisa fulltext foi criada com o objetivo de fornecer uma busca semântica em bases que contenham muito texto. Dessa forma, o MySQL desconsidera palavras com menos de quatro caracteres. Expressões como “de”, “que” e “ou” são excluídas automaticamente da pesquisa. Esta restrição é justificável na maioria dos casos, dada a baixa seletividade destas palavras em pesquisas textuais.

`"Uma palavra presente em mais de 50% dos registros será excluída da pesquisa, pois o MySQL considera sua relevância baixa. Exemplo:"`

## Executando pesquisas fulltext em modo booleano

A partir da versão 4.0.1, o MySQL disponibiliza o recurso de pesquisa fulltext com parâmetros booleanos, aumentando significativamente o poder na construção de filtragens de texto.

*NOTA*: A pesquisa booleana desconsidera o filtro de 4 letras mínimas e de 50% de ocorrência no resultado. Portanto, se você precisa de uma busca que não leve em consideração essas restrições, utilize o modo booleano.

### Veja a lista dos operadores disponíveis:

`+` : a string deve estar presente em todos os registros retornados;

`-` : a string não deve estar presente nos registros retornados;

`*` : trabalha com parte da palavra a ser procurada;

`“ ”`: retorna a string entre aspas duplas exatamente da maneira como foi digitada;

`( )`: Agrupa palavras em sub-expressões;

`< >`: Muda a contribuição da string no cálculo da relevância. O operador < decrementa a relevância e o operador > aumenta a relevância;

`~` : age como operador de negação. A contribuição de relevância da string se torna negativa.

### Veja o exemplo:

```sql
# modelo por modo boleano, quando parte do texto é procurado
SELECT titulo, descricao
FROM websites
WHERE MATCH (titulo, descricao) AGAINST ('*SQL*' IN BOOLEAN MODE);

# modelo retorna os registros que não possuem a string “Microsoft” e que possuem obrigatoriamente a string “Brasil”.
SELECT titulo, descricao
 FROM websites
 WHERE MATCH (titulo, descricao)
 AGAINST ('+Brasil -Microsoft' IN BOOLEAN MODE);

# ocorrência exata da string “Microsoft Brasil“.
SELECT titulo, descricao
 FROM websites
 WHERE MATCH (titulo, descricao)
 AGAINST ('"Microsoft Brasil"' IN BOOLEAN MODE);

# modelo com operador de descremento, reduzindo significancia do texto
SELECT titulo,
 MATCH (titulo, descricao) AGAINST ('<Oracle business' IN BOOLEAN MODE)
 FROM websites
 WHERE MATCH (titulo, descricao)
 AGAINST ('<Oracle business' IN BOOLEAN MODE);

# a combinação “Brasil Microsoft” tem um peso menor do que a combinação “Brasil Network”
SELECT titulo, MATCH (descricao,titulo) AGAINST ('+Brasil +(Network)' IN BOOLEAN MODE) as relevancia
FROM websites
WHERE MATCH (descricao,titulo)
AGAINST ('+Brasil +(Network)' IN BOOLEAN MODE)

# O uso de “~” faz com que a palavra perca a importância na pesquisa, sem efetivamente excluir a linha que a contém, como faz o operador “-“. Dessa forma, utilize “~” para diminuir o valor de relevância da frase que contém a palavra em questão.
SELECT titulo,MATCH (titulo,descricao) AGAINST ('dados ~oracle' IN BOOLEAN MODE) AS relevancia
FROM websites
WHERE MATCH (titulo,descricao) AGAINST ('dados ~oracle' IN BOOLEAN MODE);
```

### Visualizando as STOPWORDS

```sql
# as palavras que contem nesta tabela do sistema são ignoradas na pesquisa
SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
```

### Padrão de pesquisa

o FULLTEXT tem o padrão de pesquisa acima de 3 caracteres, sendo assim precisamos alterar a configuração no `my.ini`
```sql
# Dentro do my.ini

# The MySQL server
[mysqld]
ft_min_word_len=1 // para Myisam
innodb_ft_min_token_size=1 // para innodb
```

Reparar a tabela que contem o FULLTEXT
* `REPAIR TABLE  table_name QUICK;`

Em caso de erro, 
* `Recriar os indices FULLTEXT`

## UTILIZANDO EM PROCEDURES

Passos para inclusão:
```sql
BEGIN
SET GLOBAL innodb_optimize_fulltext_only=ON;
OPTIMIZE TABLE table_name;

# SEUS CÓDIGOS

COMMIT;
SET GLOBAL innodb_optimize_fulltext_only=OFF;
```

### FONTES
<!--ts-->
   * [DevMedia](https://www.devmedia.com.br/indices-fulltext-no-mysql/7631) - leitura e explicações
   * [Logic Hacks](https://www.youtube.com/watch?v=ng9E16ufYRs) - configuração my.ini      
<!--te-->
>Leitura e percepção sobre o assunto desde 10/10/2021


