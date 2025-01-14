
O `parcer.c` é um arquivo com para auxiiiar a analise  sintática do [[yacc.y|yacc]]

# **Variáveis Globais**

O arquivo **`Parser.c`** contém três estruturas globais:

- **`GLOBAL_DATA`** (tipo: [[types#Struct rc_insert|rc_insert]]): Armazena dados dos tokens.
- **`GLOBAL_PARSER`** (tipo: [[types#Struct rc_parser|rc_parser]]): Auxilia no funcionamento do parser.
- **`SELECT`** (tipo: [[types#Struct inf_select|inf_select]]): Registra informações sobre comandos `SELECT`.

elas servem para guardar dados dos tokens, informações do select e para auxiliar no parser, respectivamente.

----
# Funções

## Connect

Verifica se o connectDb retornou [[Macros#Tratamento de erros|Success]]
## setObjName

O setObjName primeiro verifica que o mode é diferente de 0, ou seja, se é o modo não é nulo. Caso seja um modo não nulo, o nome da tabela é salvo em caixa baixa pelo [[misc#strcpylower|strcpylower]] no [[#Globais|GLOBAL_DATA]] e soma mais um passo "step".

```C
void setObjName(char **nome) {
	if (GLOBAL_PARSER.mode != 0) {
		GLOBAL_DATA.objName = malloc(sizeof(char)*((strlen(*nome)+1)));
		strcpylower(GLOBAL_DATA.objName, *nome);
		GLOBAL_DATA.objName[strlen(*nome)] = '\0';
		GLOBAL_PARSER.step++;
	}
}
``` 

Detalhe: a função [[misc#strcpylower|strcpylower]] já coloca o '\0', não sei por que colocam o '\0' depois da chamada desta função.

## setColumnInsert  
A função setColumnInsert recebe um ponteiro para um ponteiro.

Ela inicia fazendo o realoc do collumn.name para caber +1 ponteiro do tipo char.

Aloca para esse novo ponteiro bytes suficientes para guardar o nome da coluna com o caracter nulo '/0'.

Logo após ele efetivamente salva o nome da coluna no [[#Globais|GLOBAL_DATA]] com o [[misc#strcpylower]], e assim incrementando a contagem de colunas.

```C
void setColumnInsert(char **nome) {

GLOBAL_DATA.columnName = realloc(GLOBAL_DATA.columnName, (GLOBAL_PARSER.col_count+1)*sizeof(char *));

GLOBAL_DATA.columnName[GLOBAL_PARSER.col_count] = malloc(sizeof(char)*(strlen(*nome)+1));

strcpylower(GLOBAL_DATA.columnName[GLOBAL_PARSER.col_count], *nome);
GLOBAL_DATA.columnName[GLOBAL_PARSER.col_count][strlen(*nome)] = '\0';
GLOBAL_PARSER.col_count++;
}
```
  
Detalhe:
- Não entendi por que a função recebe nome como um ponteiro para um ponteiro, geralmente se faz isso com um array de arrays (matriz), e neste caso de um char** o uso mais comum é para um ponteiro que aponta para o inicio de um array de strings. Só que esta função trabalha como um array de 1 posição, o que é verdade, só não entendo o uso do \*\* poderia substituir tranquilamente por um ponteiro comum \*.
- Novamente ele colando o '\0' no final da string sendo que o strcpylower já faz isto.

## setValueInsert
A função setValueInsert recebe uma string e um char representando o tipo do valor.

a função realoca o tamanho de values e type para caber +1 ponteiro para uma string e +1 tipo, respectivamente.

Logo após aloca bytes suficientes para guardar o valor inserido.  

E feito a comparação para saber se o valor inserido é string ou não, se não for string ele copia o dado direto para o [[#Globais|GLOBAL_DATA.values]], se for uma string ele copia o dado para values só que removendo as aspas.

finaliza salvando o typo do atributo e incrementando a contagem de valores inseridos.

```C

void setValueInsert(char *nome, char type) {

int i;
GLOBAL_DATA.values = realloc(GLOBAL_DATA.values,
		(GLOBAL_PARSER.val_count+1)*sizeof(char *));

GLOBAL_DATA.type = realloc(GLOBAL_DATA.type,
		(GLOBAL_PARSER.val_count+1)*sizeof(char));

GLOBAL_DATA.values[GLOBAL_PARSER.val_count] = 
		malloc(sizeof(char)*(strlen(nome)+1));

if (type == 'I' || type == 'D') {
	strcpy(GLOBAL_DATA.values[GLOBAL_PARSER.val_count], nome);
	GLOBAL_DATA.values[GLOBAL_PARSER.val_count][strlen(nome)] = '\0';
} else if (type == 'S') {
	for (i = 1; i < strlen(nome)-1; i++) {
		GLOBAL_DATA.values[GLOBAL_PARSER.val_count][i-1] = nome[i];
	}
	GLOBAL_DATA.values[GLOBAL_PARSER.val_count][strlen(nome)-2] = '\0';
	}
GLOBAL_DATA.type[GLOBAL_PARSER.val_count] = type;
GLOBAL_PARSER.val_count++;
}
```
---


## limparLista

Função para liberar a memória de uma [[Utility#Lista|Lista]] duplamente encadeada.  
Percorre todos os nós, removendo-os com [[Utility#rmvNodoPtr|rmvNodoPtr]], e libera a lista.

```C
void limparLista(Lista *l){
	Nodo *k = l->prim,*j;
	while(k){
		j = k->prox;
		free( rmvNodoPtr(l,k) );
		k = j;
	}
	l->prim = l->ult = NULL;
	free(l);
}
```
---
# resetSelect

Função que reseta a estrutura global **`SELECT`**, liberando memória associada à tabela, tokens e projeções.  

Utiliza [[#limparLista]] para liberar listas dentro de `SELECT`.

```C
if(SELECT.tabela){
	free(SELECT.tabela);
	SELECT.tabela = NULL;
}

if(SELECT.tok) limparLista(SELECT.tok);
SELECT.tok = NULL;
if(SELECT.proj) limparLista(SELECT.proj);
SELECT.proj = NULL;
}
```

---
# clearGlobalStructs

Função que reseta todas as variáveis globais relacionadas ao parser e limpa a memória alocada:

- Chama [[#resetSelect|resetSelect]] para zerar o `SELECT`.
- Libera a memória de `GLOBAL_DATA` e redefine seus campos para `NULL`. Os campos **`type`** e **`attribute`** são realocados como `char` e `int`, respectivamente.
- Destrói o analisador léxico com `yylex_destroy`.
- Reseta os contadores e flags de `GLOBAL_PARSER`.

```c

int i;
resetSelect();
if (GLOBAL_DATA.objName) {
	free(GLOBAL_DATA.objName);
	GLOBAL_DATA.objName = NULL;
}
 
for (i = 0; i < GLOBAL_DATA.N; i++ ) {
	if (GLOBAL_DATA.columnName)
		free(GLOBAL_DATA.columnName[i]);
	if (GLOBAL_DATA.values)
		free(GLOBAL_DATA.values[i]);
	if (GLOBAL_DATA.fkTable)
		free(GLOBAL_DATA.fkTable[i]);
	if (GLOBAL_DATA.fkColumn)
		free(GLOBAL_DATA.fkColumn[i]);
}

free(GLOBAL_DATA.columnName);
GLOBAL_DATA.columnName = NULL;

free(GLOBAL_DATA.values);
GLOBAL_DATA.values = NULL;

free(GLOBAL_DATA.fkTable);
GLOBAL_DATA.fkTable = NULL;

free(GLOBAL_DATA.fkColumn);
GLOBAL_DATA.fkColumn = NULL;

free(GLOBAL_DATA.type);
GLOBAL_DATA.type = (char *)malloc(sizeof(char));

free(GLOBAL_DATA.attribute);
GLOBAL_DATA.attribute = (int *)malloc(sizeof(int));
  
yylex_destroy();


GLOBAL_DATA.N = 0;
GLOBAL_PARSER.mode = 0;
GLOBAL_PARSER.parentesis = 0;
GLOBAL_PARSER.noerror = 1;
GLOBAL_PARSER.col_count = 0;
GLOBAL_PARSER.val_count = 0;
GLOBAL_PARSER.step = 0;

}
```
# setMode

O setMode altera o comando do global parser com base nos comandos definidos em [[Constantes#Parser|Arquivo .h]] e incrementa o passo.

```c 
void setMode(int mode) { 
	GLOBAL_PARSER.mode = mode;
	GLOBAL_PARSER.step++; 
}
```


---
# Interface

## Set das variáveis

- Chama a [[#clearGlobalStructs]];
- Inicia uma variavel **`resultado`** do tipo [[Utility#**Lista**|lista]];
- Chama a connect("uffsdb") iniciando no banco padrão;
- Inicia tok e proj da SELECT;
```C
pthread_create(&pth, NULL, (void*)clearGlobalStructs, NULL);
pthread_join(pth, NULL); \\ faz a tread principal esperar
Lista *resultado;
connect("uffsdb"); // conecta automaticamente no banco padrão
SELECT.tok = SELECT.proj = NULL;
```

> Não faz sentido o uso de thread pois logo depois faz a principal esperar, é como se tivesse chamando a função clearGlobalStructs direto.

---
