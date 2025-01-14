O types tem todos os tipos definidos no sistema, sendo eles: 
- [[#db_connected]] usada para gerenciar informações da conexão;
- data_base usada para gerenciar as databases.


# Structs
## db_connected
#main

Utilizado no sistema para gerenciar informações da conexão ativa.
### Atributos
- **`db_directory`**: String com tamanho definido em [[Macros#Tamanhos]] que armazena o diretório.
- **`db_name`**: Ponteiro para uma string que armazena o nome do banco de dados.
- **`conn_active`**: Inteiro que indica se o usuário é superusuário (`#`) ou não (`<`).
 
```C
typedef struct db_connected {
	char db_directory[LEN_DB_NAME_IO];
	char *db_name;
	int conn_active;
}db_connected;
```

LEN_DB_NAME_IO é definido como [[Macros#Tamanhos|40]].
___
## data_base

Utilizada para trabalhar com database. 
### Atributos
* `valid`: char para saber se é valido 1 ou 0.
* `db_name`: string com tamanho definido em [[Macros#Tamanhos]] que armazena o diretório.
* **`db_directory`**: String com tamanho definido em [[Macros#Tamanhos]] que armazena o diretório.

```C
typedef struct data_base{
	char valid;
	char db_name[LEN_DB_NAME_IO];
	char db_directory[LEN_DB_NAME_IO];
}data_base;
```
___