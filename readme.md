## ETL_NFT_Market_Cap

Тестовое задание

Описание:   Перед Вами шесть файлов, включая это описание задачи. В docker-compose.yml вы найдете конфигурацию
            двух баз PostgreSQL: база-источник (psql_src) и база-назначение (psql_dst). CSV файлы представляют собой
            нормализованные таблицы для загрузки в базу-источник.

| Задача      | Описание |
| ----------- | ----------- |
| 1      | Загрузить данные из csv файлов в базу-источник       |
| 2   | Рассчитать капитализацию NFT-коллекций в долларах на дневной основе за всю историю сделок по коллекции.        |
| 3 |Загрузить получившиеся данные в базу-назначение |

### Задание 1

Первое, что надо сделать -- перейти в папку с тестовым заданием и запустить докер, используя **.yaml** файл

```shell script
vyacheslavdyrenkov@MacBook-Pro ~ % cd ~/Downloads/test_cnft
vyacheslavdyrenkov@MacBook-Pro test_cnft % docker-compose up                  
```
Вывод будет следующим
<details> 
    <summary markdown="span">Result</summary>
	
```shell script
Starting test_cnft_psql_src_1 ... done
Starting test_cnft_psql_dst_1 ... done
Attaching to test_cnft_psql_dst_1, test_cnft_psql_src_1
psql_dst_1  | 
psql_dst_1  | PostgreSQL Database directory appears to contain a database; Skipping initialization
psql_dst_1  | 
psql_dst_1  | 2022-09-13 21:22:06.936 UTC [1] LOG:  starting PostgreSQL 14.5 (Debian 14.5-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
psql_dst_1  | 2022-09-13 21:22:06.936 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
psql_dst_1  | 2022-09-13 21:22:06.936 UTC [1] LOG:  listening on IPv6 address "::", port 5432
psql_dst_1  | 2022-09-13 21:22:06.939 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
psql_dst_1  | 2022-09-13 21:22:06.945 UTC [26] LOG:  database system was interrupted; last known up at 2022-09-13 21:20:10 UTC
psql_src_1  | 
psql_src_1  | PostgreSQL Database directory appears to contain a database; Skipping initialization
psql_src_1  | 
psql_src_1  | 2022-09-13 21:22:06.980 UTC [1] LOG:  starting PostgreSQL 14.5 (Debian 14.5-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
psql_src_1  | 2022-09-13 21:22:06.981 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
psql_src_1  | 2022-09-13 21:22:06.981 UTC [1] LOG:  listening on IPv6 address "::", port 5432
psql_src_1  | 2022-09-13 21:22:06.984 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
psql_src_1  | 2022-09-13 21:22:06.990 UTC [26] LOG:  database system was interrupted; last known up at 2022-09-13 21:20:10 UTC
psql_dst_1  | 2022-09-13 21:22:07.121 UTC [26] LOG:  database system was not properly shut down; automatic recovery in progress
psql_dst_1  | 2022-09-13 21:22:07.123 UTC [26] LOG:  redo starts at 0/1727BA8
psql_dst_1  | 2022-09-13 21:22:07.123 UTC [26] LOG:  invalid record length at 0/1727BE0: wanted 24, got 0
psql_dst_1  | 2022-09-13 21:22:07.123 UTC [26] LOG:  redo done at 0/1727BA8 system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
psql_dst_1  | 2022-09-13 21:22:07.132 UTC [1] LOG:  database system is ready to accept connections
psql_src_1  | 2022-09-13 21:22:07.151 UTC [26] LOG:  database system was not properly shut down; automatic recovery in progress
psql_src_1  | 2022-09-13 21:22:07.154 UTC [26] LOG:  redo starts at 0/93D8BC8
psql_src_1  | 2022-09-13 21:22:07.154 UTC [26] LOG:  invalid record length at 0/93D8C00: wanted 24, got 0
psql_src_1  | 2022-09-13 21:22:07.154 UTC [26] LOG:  redo done at 0/93D8BC8 system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
psql_src_1  | 2022-09-13 21:22:07.162 UTC [1] LOG:  database system is ready to accept connections
```
	
</details>

Но на самом деле этого недостаточно, нужно добавить **.csv** таблицы в окружение, чтобы потом добавить их в postgres.
Тут есть множество вариантов, но самый простой -- модифицировать **.yml** файл, добавив в него **volumes**, можно сделать так:

```shell script 
services:
  psql_src:
    image: postgres:14.5
    ports:
      - "54321:5432"
    environment:
      POSTGRES_DB: "checknft"
      POSTGRES_USER: "etl"
      POSTGRES_PASSWORD: "etl_contest"
    volumes:
        - /Users/vyacheslavdyrenkov/Downloads/test_cnft/:/data

  psql_dst:
    image: postgres:14.5
    ports:
      - "54322:5432"
    environment:
      POSTGRES_DB: "checknft"
      POSTGRES_USER: "etl"
      POSTGRES_PASSWORD: "etl_contest"
```
Поэтому перезапускаем докер с уже обновленным compose файлом. Теперь проверим, что все файлы деуствительно замаунтились:

```shell script 
vyacheslavdyrenkov@MacBook-Pro test_cnft % docker exec -it $(docker ps -q -f name=test_cnft_psql_src_1) bash
root@67aeea5ba41d:/# ls /data
```

Получим следующий вывод

```text
collection.csv	docker-compose.yml  event.csv  payment_token.csv  public.marketcap  task.txt  token.csv
```

Получается, добавили все, что нужно. Теперь необходимо создать таблицы на основе **.csv** файликов. Тут тоже есть множество вариантов, например, используя [psql](https://hub.docker.com/_/postgres/) можно так:

```
vyacheslavdyrenkov@MacBook-Pro ~ % psql -h localhost -p 54321 -U etl -W checknft 
checknft=# create table public.payment_token (id int, createdAt timestamp, updatedAt timestamp, symbol text, address text, imageUrl text, name text, externalId int, decimals int);
checknft=# COPY public.payment_token FROM '/data/payment_token.csv' CSV HEADER;
```

В итоге получим положительный ответ с копированием таблицы. Однако такой вариант довольно громоздок для широких таблиц, поэтому воспользуемся SQL клиетом и загрузим оставшиеся таблицы.
Я использовал DBeaver со следующими кредами:

```json
{Host: "localhost"
Port: 54321
Database: "checknft"
User: "etl"
Password: "etl_contest"}
```

Далее через import добавляем оставшиеся таблички в базу-источник, меняя немного изменяя дефолтную конфигурацию DDL следующим образом

DDL для public.collection

<details> 
    <summary markdown="span">Result</summary>
    
```bigquery
CREATE TABLE public.collection (
	id integer NULL,
	createdat text NULL,
	updatedat text NULL,
	"name" text NULL,
	externalid text NULL,
	description text NULL,
	logo text NULL,
	creatoraccountid integer NULL,
	ownerfee integer NULL,
	protocolfee integer NULL,
	termsandconditionsurl text NULL,
	totalsupply integer NULL,
	categoryid text NULL,
	externalslug text NULL,
	waitingforremove boolean NULL,
	isverified boolean NULL,
	marketplaceapiurl text NULL,
	marketplacecollectionname text NULL,
	marketplacecollectiondescription text NULL,
	discordurl text NULL,
	externalurl text NULL,
	mediumusername text NULL,
	telegramurl text NULL,
	twitterusername text NULL,
	instagramusername text NULL,
	wikiurl text NULL,
	imageurl text NULL,
	featuredimageurl text NULL,
	largeimageurl text NULL,
	bannerimageurl text NULL,
	onedayvolume real NULL,
	onedaychange real NULL,
	onedaysales integer NULL,
	onedayaverageprice real NULL,
	sevendayvolume real NULL,
	sevendaychange real NULL,
	sevendaysales integer NULL,
	sevendayaverageprice real NULL,
	thirtydayvolume real NULL,
	thirtydaychange real NULL,
	thirtydaysales integer NULL,
	thirtydayaverageprice real NULL,
	totalvolume real NULL,
	totalsales integer NULL,
	numowners integer NULL,
	averageprice real NULL,
	marketcap real NULL,
	floorprice real NULL,
	statsupdatedat text NULL,
	totalsupplyedition real NULL
);
```

</details>

DDL для public.token

<details> 
    <summary markdown="span">Result</summary>
	
```bigquery
CREATE TABLE public."token" (
	id integer NULL,
	createdat text NULL,
	updatedat text NULL,
	externalid integer NULL,
	collectionid integer NULL,
	contractaddress text NULL,
	contractid integer NULL,
	"name" text NULL,
	description text NULL,
	unlockablecontent text NULL,
	iseditablemetadata boolean NULL,
	quantity integer NULL,
	previewurl text NULL,
	animationurl text NULL,
	filetype text NULL,
	url text NULL,
	storagetype text NULL,
	syncedat text NULL,
	mintedat text NULL,
	metaurl text NULL,
	externalurl text NULL,
	orderssyncdate text NULL,
	attributessyncedat text NULL,
	datafeed text NULL,
	collectionname text NULL,
	creatoraccountaddress text NULL,
	creatoraccountname text NULL,
	dailypricegrowth text NULL,
	weeklypricegrowth text NULL,
	monthlypricegrowth real NULL,
	totalpricegrowth real NULL,
	metadataerror boolean NULL,
	statrarityscore real NULL,
	croppedpreviewurl text NULL,
	previewstatus text NULL
);

```
	
</details>

DDL для public.event

<details> 
    <summary markdown="span">Result</summary>
	
```bigquery
CREATE TABLE public."event" (
	id integer NULL,
	createdat text NULL,
	updatedat text NULL,
	externalid text NULL,
	tokenid integer NULL,
	eventtype text NULL,
	"date" text NULL,
	datafeed text NULL,
	auctiontype text NULL,
	currency text NULL,
	usdprice decimal(40) NULL,
	endingprice decimal(40) NULL,
	startingprice decimal(40) NULL,
	totalprice decimal(40) NULL,
	approvedaccount text NULL,
	bidamount text NULL,
	duration integer NULL,
	fromaccount text NULL,
	quantity real NULL,
	seller text NULL,
	toaccount text NULL,
	winneraccount text NULL,
	"transaction" text NULL,
	ownerfee real NULL,
	protocolfee real NULL,
	paymenttokenid real NULL,
	paymenttokenusdprice decimal(40) NULL,
	paymenttokenethprice decimal(40) NULL,
	logindex decimal(40) NULL,
	countrelated integer NULL,
	saleprotocol text NULL,
	batchtokenindex real NULL,
	saleeventindex real NULL,
	internaltype text NULL
);
```

</details>
	
### Задание 2

Теперь надо написать SQL-запрос, который считает капитализацию. Его можно написать через редактор в терминале, как при создании таблицы public.payment_token, а можно продолжить использовать DBeaver, в любом случае запрос будет таким

```bigquery
with 
	t as 
		(select
			id,
			externalid ,
			collectionid
		from public."token"),
	c as 
		(select 
			id ,
			"name" as collectionname
		from public.collection),
	e as 
		(select
			date ,
			tokenid ,
			totalprice ,
			quantity ,
			paymenttokenusdprice ,
			paymenttokenid
		from public."event"
		where paymenttokenusdprice is not null 
			and totalprice is not null),
	pt as 
		(select 
			decimals,
			id
		from public.payment_token pt),
	f as 
		(select distinct
			*
		from e 
		join t on e.tokenid  = t.id
		join c on c.id = t.collectionid 
		join pt on pt.id = e.paymenttokenid)
select 
	date(date) as dte,
	collectionid,
	collectionname,
	sum(totalprice::decimal * 10^(-1 * decimals) * paymenttokenusdprice * quantity) as marketcupusd
from f 
group by 
	dte,
	collectionid,
	collectionname
order by 
	dte,
	collectionid,
	collectionname
```

<details> 
    <summary markdown="span">Result</summary>
	
| dte      | collectionid | collectionname | marketcupusd |
| ----------- | ----------- |  ----------- |  ----------- |
| 2021-04-30 |	504 |	BoredApeYachtClub |	439.4616796875 |
| 2021-05-01 |	504 |	BoredApeYachtClub |	1416200.7639675112 |
| 2021-05-02 |	504 |	BoredApeYachtClub |	4256964.595747707 |
| 2021-05-03 |	504 |	BoredApeYachtClub |	3483298.6952427668 |
| 2021-05-04 |	504 |	BoredApeYachtClub |	1161874.9830673 |
| 2021-05-05 |	504 |	BoredApeYachtClub |	611240.0362086919 |
| 2021-05-06 |	504 |	BoredApeYachtClub |	529984.9787216347 |
| 2021-05-07 |	504 |	BoredApeYachtClub |	331013.49038239766 |
| 2021-05-08 |	504 |	BoredApeYachtClub |	271969.5717960205 |
| 2021-05-09 |	504 |	BoredApeYachtClub |	373938.1253493166 |
| 2021-05-10 |	504 |	BoredApeYachtClub |	154623.78616596683 |
| ... | ... | ... | ... | ...|
</details>

Далее сохраним запрос в виде отдельной таблицы: 

<details> 
    <summary markdown="span">Show query</summary>
	
```bigquery
with 
	t as 
		(select
			id,
			externalid ,
			collectionid
		from public."token"),
	c as 
		(select 
			id ,
			"name" as collectionname
		from public.collection),
	e as 
		(select
			date ,
			tokenid ,
			totalprice ,
			quantity ,
			paymenttokenusdprice ,
			paymenttokenid
		from public."event"
		where paymenttokenusdprice is not null 
			and totalprice is not null),
	pt as 
		(select 
			decimals,
			id
		from public.payment_token pt),
	f as 
		(select distinct
			*
		from e 
		join t on e.tokenid  = t.id
		join c on c.id = t.collectionid 
		join pt on pt.id = e.paymenttokenid),
	res as 
		(select 
			date(date) as dte,
			collectionid,
			collectionname,
			sum(totalprice::decimal * 10^(-1 * decimals) * paymenttokenusdprice * quantity) as marketcupusd
		from f 
		group by 
			dte,
			collectionid,
			collectionname
		order by 
			dte,
			collectionid,
			collectionname)

SELECT
	*
INTO TABLE public.marketcap
FROM
    res
```
</details>

### Задание 3

Остался последний шаг -- загрузить полученный результат в базу-назначение.
Как вариант можно сделать так

```shell script
vyacheslavdyrenkov@MacBook-Pro test_cnft % PGPASSWORD="etl_contest" pg_dump -h localhost -p 54321 -U etl -d checknft -t public.marketcap > public.marketcap
vyacheslavdyrenkov@MacBook-Pro test_cnft % PGPASSWORD="etl_contest" psql -h localhost -p 54322 -U etl -d checknft -f public.marketcap 
```

Теперь проверим, что все действительно работает

```shell script 
vyacheslavdyrenkov@MacBook-Pro psql -h localhost -p 54322 -U etl -W checknft 
checknft=# select * from public.marketcap;
```
Видим, что таблица появилась.

