
# Cálculo dos dias úteis no Postgresql
*Como criar uma VIEW com parâmetros de entrada para calcular os dias úteis de uma data, e além*

Você também pode ler a este artigo no **medium** 
[Cálculo dos dias úteis no Postgresql](https://rickrochaso.medium.com/c%C3%A1lculo-dos-dias-%C3%BAteis-no-postgresql-76be47470647)

## Solução do Marcos 

```sql
  select count(*)
  from generate_series('2021-04-01'::date, '2021-04-01'::date + '1 month -1 day'::interval, '1 day'::interval) as g(g)
  where extract(dow from g) between 1 and 5
```
## Criando uma function 

```sql
create or replace	
function diasuteis(data_base date) returns int as $body$
select
	count(*)::integer
from
	generate_series($1, $1 + '1 month -1 day'::interval, '1 day'::interval) as g(g)
where
	extract(dow from g) between 1 and 5 
$body$ language sql;

```
## Testando a function 

```sql
select diasuteis('2021-03-01') 
```

```cmd
> returns 23
```

## Listando todos os meses de um período, e os dias úteis daquele mês
```sql
select subselect.dia, diasuteis(subselect.dia) from 
(SELECT date_trunc('month', dd):: date dia
FROM generate_series
        ( :inicio ::date
        , :termino ::date
        , '1 month'::interval) dd
        )subselect;
```
## Lidando com o problema dos feriados
```sql  
create table feriados (
                    codigo serial,
                    mes date, --Apenas o primeiro dia do mes
                    feriados integer,
                    primary key(codigo)
                  )
```

## Transformando a solução numa API que retorna JSON 

```sql 
select to_jsonb(api) from 
(select subselect.mes, diasuteis(subselect.mes) from 
(SELECT date_trunc('month', dd):: date mes
FROM generate_series
        ( :inicio ::date
        , :termino ::date
        , '1 month'::interval) dd
        )subselect) api;
```

## Resultado da saída no console 
### Parâmetros :inicio='2020-01-01' :termino='2022-12-31' 
```json
[
{"mes": "2020-01-01", "diasuteis": 23}
{"mes": "2020-02-01", "diasuteis": 20}
{"mes": "2020-03-01", "diasuteis": 22}
{"mes": "2020-04-01", "diasuteis": 22}
{"mes": "2020-05-01", "diasuteis": 21}
{"mes": "2020-06-01", "diasuteis": 22}
{"mes": "2020-07-01", "diasuteis": 23}
{"mes": "2020-08-01", "diasuteis": 21}
{"mes": "2020-09-01", "diasuteis": 22}
{"mes": "2020-10-01", "diasuteis": 22}
{"mes": "2020-11-01", "diasuteis": 21}
{"mes": "2020-12-01", "diasuteis": 23}
{"mes": "2021-01-01", "diasuteis": 21}
{"mes": "2021-02-01", "diasuteis": 20}
{"mes": "2021-03-01", "diasuteis": 23}
{"mes": "2021-04-01", "diasuteis": 22}
{"mes": "2021-05-01", "diasuteis": 21}
{"mes": "2021-06-01", "diasuteis": 22}
{"mes": "2021-07-01", "diasuteis": 22}
{"mes": "2021-08-01", "diasuteis": 22}
{"mes": "2021-09-01", "diasuteis": 22}
{"mes": "2021-10-01", "diasuteis": 21}
{"mes": "2021-11-01", "diasuteis": 22}
{"mes": "2021-12-01", "diasuteis": 23}
{"mes": "2022-01-01", "diasuteis": 21}
{"mes": "2022-02-01", "diasuteis": 20}
{"mes": "2022-03-01", "diasuteis": 23}
{"mes": "2022-04-01", "diasuteis": 21}
{"mes": "2022-05-01", "diasuteis": 22}
{"mes": "2022-06-01", "diasuteis": 22}
{"mes": "2022-07-01", "diasuteis": 21}
{"mes": "2022-08-01", "diasuteis": 23}
{"mes": "2022-09-01", "diasuteis": 22}
{"mes": "2022-10-01", "diasuteis": 21}
{"mes": "2022-11-01", "diasuteis": 22}
{"mes": "2022-12-01", "diasuteis": 22}]
```
