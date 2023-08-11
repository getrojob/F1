# F1

POC - Java Spring com arquitetura hexagonal e SQL entre outras tecnologias.

#####Atividades

NOTA:

Requisitos.
Antes de começar a escrever qualquer código, entender quais são os objetivos do projeto, não só para ter uma compreensão clara de onde o projeto irá e sua função, mas também para que não ocorra desvio do caminho e comece escrever recursos adicionais que possam ser implementados em versões futuras e evitar aumento de escopo na versão atual.

#####Mandatorio:

- Diagrama de sequencia
- Diagramma de classe

- FrontEnd: Angular
- BackEnd: Java
- SQL

- Log Tecnico
- Log Negocio

O banco de dados 
- Um aplicativo da web simples que permitirá
    - Inserção de resultados de corrida e registro de resultados contra pilotos individuais
    - Visualização de uma tabela geral de resultados de corrida para uma determinada temporada
- A capacidade de registrar cada divisão de corrida em um fim de semana inteiro.
    - Sessões Práticas
    - Qualificação
    - Dia de corrida

A aplicação deve ter a capacidade de calcular os pontos para um piloto com base em sua posição final.

    	○ 1 (25)
    	○ 2 (18)
    	○ 3 (15)
    	○ 4 (12)
    	○ 5 (10)
    	○ 6 (8)
    	○ 7 (6)
    	○ 8 (4)
    	○ 9 (2)
    	○ 10 (1)
    	○ FL (1)

A aplicação deve ter a capacidade de rastrear os pilotos e a equipe na qual eles estão dirigindo entre as temporadas.

- Os pilotos terão uma data de início e término para cada equipe para a qual competem.
- Um motorista com uma data de término preenchida para uma equipe indicará que o motorista não dirige mais para aquela equipe.

A aplicação deve ter a capacidade de rastrear quais motores as equipes estão usando entre as temporadas.
- Motores muito parecidos com os motoristas terão uma data de início e término para cada equipe.
- Uma data de término preenchida significa que o mecanismo não está mais em uso por esta equipe.

A aplicação deve ter a capacidade de rastrear circuitos de corrida e incluí-los ou não em uma temporada individual.

SQL

```
Use Master
CREATE DATABASE F1Tracker;
CREATE LOGIN [RaceTracker] WITH PASSWORD=N'StrongPassword1234!', DEFAULT_DATABASE=[F1Tracker], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF;
GO
USE F1Tracker;
CREATE USER [RaceTracker] FOR LOGIN [RaceTracker];
    ALTER ROLE [db_owner] ADD MEMBER [RaceTracker];
GO
```

A tabela relacionada ao piloto conterá todas as informações sobre os pilotos;

- Primeiro nome
- Sobrenome
- Nacionalidade
- Aposentado

A tabela também terá uma chave primária que configurei como driver_id. Não dei uma identidade à chave primária porque quero especificar minhas ids inteiras para drivers.

```
CREATE TABLE Drivers
(
    Driver_ID INT NOT NULL,
    Forename varchar(20),
    Surname varchar(20),
    Nationality varchar(20),
    Retired BIT
);
```

Muito parecido com a tabela de piloto, esta contém todas as equipes; 

- ID da equipe FK
- Nome do time 
- Bandeira Ativa

Assim como a tabela de pilotos, não há coluna de identidade especificada para esta tabela, pois desejo especificar meus próprios IDs inteiros para Team_ID, que também é a chave primária desta tabela.

```
CREATE TABLE Teams
 (
     Team_ID TINYINT NOT NULL,
     Team_Name varchar(20),
     Active BIT DEFAULT 1
 );

INSERT INTO Teams (Team_ID,Team_Name) VALUES
    (1,'Alfa Romeo'),
    (2,'Ferrari'),
    (3,'Haas'),
    (4,'McLaren'),
    (5,'Mercedes'),
    (6,'Racing Point'),
    (7,'Red Bull Racing'),
    (8,'Renault'),
    (9,'Scuderia Toro Rosso'),
    (10,'Williams');
```

A tabela de equipe dos pilotos é uma tabela de ligação, esta tabela irá conectar todos os nossos pilotos às nossas equipes,
- ID da equipe FK
    - Especifiquei um Team_ID que é uma chave estrangeira, referencia o Team_ID da tabela do time
- Diver ID FK
    - o Driver_ID também é uma chave estrangeira que faz referência ao Drivers_ID da tabela do driver
- Data de início
- Data final

Também especifiquei uma data de início e de término para cada piloto com cada equipe nesta tabela de links, isso porque os pilotos de F1 podem começar e terminar com uma determinada equipe após cada temporada.

Nota: Se End_Date não for preenchido, o motorista ainda está dirigindo para a equipe especificada.

```
CREATE TABLE Driver_Team
(
    Team_ID INT,
    Driver_ID INT,
    Start_date DATETIME,
    End_date DATETIME
);

INSERT INTO Driver_Team (Driver_ID,Team_ID,Start_Date) VALUES
(1,9,GETDATE()),
(2,5,GETDATE()),
(3,7,GETDATE()),
(4,1,GETDATE()),
(5,5,GETDATE()),
(6,8,GETDATE()),
(7,10,GETDATE()),
(8,9,GETDATE()),
(9,2,GETDATE()),
(10,2,GETDATE()),
(11,4,GETDATE()),
(12,6,GETDATE()),
(13,1,GETDATE()),
(14,8,GETDATE()),
(15,10,GETDATE()),
(16,4,GETDATE()),
(17,6,GETDATE()),
(18,7,GETDATE()),
(19,2,GETDATE());
```

Tal como acontece com os pilotos, as equipes na F1 podem muitas vezes mudar o motor que estão usando em seus carros de uma temporada para outra. Com isso em mente, preciso ser capaz de registrar isso, a tabela de links Team_Engine nos permitirá fazer isso. 

- ID da equipe FK 
- Motor 
- Data de início 
- Data final

O Team_ID é uma chave estrangeira que faz referência à tabela do time. 
A data de início e a data de término funcionam da mesma forma que no Driver_Team.

Nota: Se End_Date não for preenchido, o mecanismo listado ainda está sendo usado pela equipe especificada.

```
CREATE TABLE Team_Engine
(
   Team_ID TINYINT,
   Engine varchar(15),
   Start_Date DATETIME,
   End_date DATETIME
);

INSERT INTO Team_Engine (Team_ID, Engine,Start_date) VALUES
(1,'Ferrari',GetDate()),
(2,'Ferrari',GetDate()),
(3,'Ferrari',GetDate()),
(4,'Renault',GetDate()),
(5,'Mercedes',GetDate()),
(6,'BWT Mercedes',GetDate()),
(7,'Honda',GetDate()),
(8,'Renault',GetDate()),
(9,'Honda',GetDate()),
(10,'Mercedes',GetDate());
```

A tabela de circuitos conterá todas as informações sobre os circuitos 

- Circuito ID PK 
- Nome do Circuito 
- Tipo de Circuito
    - Trilha ou cidade
- Direção
    - A direção ao redor da pista em que o piloto corre 
- Localização do circuito
    - Localização do Circuito 
- Último comprimento usado
    - O último comprimento que foi usado para correr 
- Nome do Grande Prêmio 
- Data de início 
- Data final

Há um pequeno número de colunas especificadas usadas no aplicativo final. Capturei mais informações do que o necessário, permitindo relatórios adicionais, caso eu queira fazer mais tarde.

Nota: Se End_Date não estiver preenchido, o circuito ainda está incluído na lista da temporada.

```
CREATE TABLE Circuit
(
    Circuit_ID TINYINT NOT NULL,
    Circuit_Name nvarchar(60),
    Circuit_Type varchar(15),
    Direction varchar(50),
    Circuit_Location nvarchar(50),
    Last_length_used DECIMAL(5,3),
    Grands_Prix_Name nvarchar(70),
    Start_date DATETIME,
    End_date DATETIME
); 
```

Tipos de corrida é uma tabela de referência que vai conter os tipos de corrida, que incluirá sessões de prática e qualificação, juntamente com o evento de corrida real, a razão para isso é para que eu possa registrar cada etapa do fim de semana dos pilotos na tabela de corrida e diferenciar entre os eventos de corrida e pré-corrida.

Race_Type_ID é a chave primária nesta tabela novamente, não há identidade especificada, pois quero que cada linha referencial tenha um ID inteiro de minha escolha.

```
CREATE TABLE Race_Types
(
    Race_Type_ID TINYINT NOT NULL,
    Race_Type NVARCHAR(10)
);

INSERT INTO Race_Types (Race_Type_ID, Race_Type)
VALUES
(1,'Practice 1'),
(2,'Practice 2'),
(3,'Practice 3'),
(4,'Qualifying'),
(5,'Race');
```

A tabela de corrida é onde todos os dados da corrida de fim de semana serão registrados, nesta tabela temos Race_ID como a chave primária, no entanto, ela tem uma identidade definida, começando em 1 e aumentando em 1 cada vez que uma linha é inserida. 

- Raça ID PK 
- Data da Corrida 
- ID do motorista FK 
- ID da equipe FK 
- Circuito ID FK 
- Posição Final 
- Pontos
    - O número de pontos, se houver, acumulado para esta corrida
Race Type FK

```CREATE TABLE Race
(
    Race_ID INT IDENTITY(1,1) NOT NULL,
    Race_Date DATETIME,
    Driver_ID INT,
    Circuit_ID TINYINT,
    Final_Position TINYINT,
    Points TINYINT,
    Season INT,
    Race_Type TINYINT
);
```

keys
Agora que temos todas as tabelas criadas e os dados preenchidos, precisamos ter certeza de que todas as chaves estao aplicadas de acordo com o design, preciso aplicar um número de chaves primárias às colunas definidas em cada tabela, isso tem que ser feito antes que as chaves estrangeiras possam ser definidas.

```
    ALTER TABLE Drivers ADD CONSTRAINT PK_Drivers_ID PRIMARY KEY (Driver_ID);
    ALTER TABLE Circuit ADD CONSTRAINT PK_Circuit_ID PRIMARY KEY (Circuit_ID);
    ALTER TABLE Race_Types ADD CONSTRAINT PK_Race_Type_Ref PRIMARY KEY (Race_Type_ID);
    ALTER TABLE Teams ADD CONSTRAINT PK_Team_ID PRIMARY KEY (Team_ID);
    ALTER TABLE Race ADD CONSTRAINT PK_Race_ID PRIMARY KEY (Race_ID);
```

Agora que as chaves primárias estão definidas, posso especificar as chaves estrangeiras para as tabelas que as requerem

```

    ALTER TABLE Team_Engine ADD CONSTRAINT FK_Team_ID FOREIGN KEY (Team_ID) REFERENCES Teams (Team_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_Type_TypeRef FOREIGN KEY (Race_Type) REFERENCES Race_Types(Race_Type_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_DriveID FOREIGN KEY (Driver_ID) REFERENCES Drivers(Driver_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_Team FOREIGN KEY (Team_ID) REFERENCES Teams(Team_ID);
    ALTER TABLE Race ADD CONSTRAINT FK_Race_CircuitID FOREIGN KEY (Circuit_ID) REFERENCES Circuit(Circuit_ID);
```

podemos executar um pequeno teste nesses dados para garantir que os dados esperados sejam retornados. Abaixo está uma consulta que retornará todos os motoristas e sua equipe atual da tabela de link Driver_Team usando as chaves estrangeiras.

```
    SELECT
      D.Forename,
      D.Surname,
      D.Nationality,
      T.Team_Name,
      DT.Start_date,
      DT.End_date
    FROM
      Driver_Team DT

INNER JOIN Drivers D ON
D.Driver_ID = DT.Driver_ID

    INNER JOIN Teams T ON
      T.Team_ID = DT.Team_ID
```



Drivers
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Driver_ID	     |    TINYINT	 |           	        | Sim | Não | Não	     |        |
| Forename    	 |    VARCHAR	 |       20	            | Não | Não | Não	     |        |
| Surname 	     |    VARCHAR	 |       20	            | Não | Não | Não	     |        |
| Nationality 	 |    VARCHAR	 |       20	            | Não | Não | Não	     |        |
| Retired 	     |    Bit	     |       	            | Não | Não | Não	     |        |


Driver_Team
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Team_ID	     |    TINYINT	 |		                |     | sim | Não	     |        |
| Driver_ID	     |    TINYINT	 |		                |     | sim | Não	     |        |
| Start_date	 |    DATETIME	 |	                    |	  | Não	| Não	     |        |
| End_date	     |    DATETIME	 |	                    |	  | Não	| Não	     |        |



Teams
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Team_ID	     |    TINYINT	 |           	        | sim |	Não	| Não        |        |
| Team_Name	     |    VARCHAR	 |        20	        | Não |	Não | Não        |        |
| Active 	     |    Bit		 |                      | Não |	Não | Não	     |   1    |


Team_Engine
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Team_ID	     |    TINYINT    |		                | Não |	sim	| Não        |        | 
| Engine 	     |    VARCHAR	 |           15	        | Não | Não | Não        |        | 
| Start_date  	 |    DATETIME	 |       	            | Não | Não | Não        |        | 
| End_date	     |    DATETIME	 |       	            | Não | Não | Não        |        | 


Circuit
| Nome da coluna    | Tipo de dados | Comprimento de dados |  PK |	FK | Identidade | Padrão |
| ----------------- | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Circuit_ID	    |    TINYINT	|	                   | sim | Não | Não        |        |
| Circuit_Name	    |    NVARCHAR	|        60	           | Não | Não | Não        |        | 
| Circuit_Type	    |    VARCHAR	|        15	           | Não | Não | Não        |        | 
| Direction	        |    VARCHAR	|        50	           | Não | Não | Não        |        | 
| Circuit_Location  |    NVARCHAR	|        50	           | Não | Não | Não        |        | 
| Last_Length_Used  |    DECIMAL	|        5,3	       | Não | Não | Não        |        | 
| Grand_Prix_Name	|    NVARCHAR	|        70	           | Não | Não | Não        |        | 
| Start_date	    |    DATETIME	|        	           | Não | Não | Não        |        | 
| End_date	        |    DATETIME	|    	               | Não | Não | Não        |        | 


Race_types
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Race_Type_ID	 |    TINYINT	 |	                    | sim |	Não | Não        |        |
| Race_Type	     |    NVARCHAR	 |       10	            | Não |	Não	| Não        |        |


Race
| Nome da coluna | Tipo de dados | Comprimento de dados	|  PK |	FK  | Identidade | Padrão |
| -------------  | ------------- | -------------------- | --- | --- | ---------- | ------ |
| Race_ID	     |    TINYINT	 |	                    | sim |	Não	| sim        |        |
| Race_Date	     |    DATETIME	 |	                    | Não |	Não	| Não        |        |        
| Driver_ID	     |    TINYINT	 |                      | Não |	sim	| Não        |        |
| Circuit_ID	 |    TINYINT	 |	                    | Não |	sim	| Não        |        |
| Final_Position |    TINYINT	 |	                    | Não |	Não	| Não        |        |
| Points 	     |    TINYINT	 |	                    | Não |	Não	| Não        |        |
| Season         |    INT        |                      |     |     |            |        |
| Race_Type	     |    TINYINT    |                      | Não |	Não	| Não        |        |
