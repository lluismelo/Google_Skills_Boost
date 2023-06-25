# Insights de dados com o BigQuery: laboratório com desafio

## Consulta 1: total de casos confirmados
Crie uma consulta para responder à pergunta: "Qual foi o total de casos confirmados em April 15, 2020?". A consulta precisa retornar uma linha com a soma dos casos confirmados em todos os países. O nome da coluna precisa ser total_cases_worldwide.

Estas são as colunas para referência:

* cumulative_confirmed
* date

```SQL
SELECT SUM(cumulative_confirmed) AS total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = "2020-04-15"
```
<br/>

## Consulta 2: áreas mais afetadas
Crie uma consulta para responder à pergunta: "Quantos estados nos EUA tiveram mais de 150 mortes em April 15, 2020?". A consulta precisa listar a saída no campo count_of_states.

Dica: não inclua valores NULL.

Estas são as colunas para referência:

* country_name
* subregion1_name (para informações sobre o estado)
* cumulative_deceased

```SQL
WITH mortes_p_estado AS (
  SELECT subregion1_name, sum(cumulative_deceased) AS total
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE date = "2020-04-15" AND country_name = "United States of America" AND subregion1_name IS NOT NULL
  GROUP BY subregion1_name
)
SELECT count(*) AS count_of_states 
FROM mortes_p_estado
WHERE total>150
```
<br/>

## Consulta 3: identificação dos pontos de acesso
Crie uma consulta para responder à pergunta: "Quais estados dos EUA tiveram mais de 1500 casos confirmados em April 15, 2020? Liste todos". A consulta precisa retornar o nome do estado e os casos confirmados correspondentes, organizados em ordem decrescente. Nomeie os campos da resposta como state e total_confirmed_cases.

Estas são as colunas para referência:

* country_code
* subregion1_name (para informações sobre o estado)
* cumulative_confirmed

```SQL
SELECT subregion1_name AS state, sum(cumulative_confirmed) AS total_confirmed_cases
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = "2020-04-15" AND country_code="US" AND subregion1_name is NOT NULL
GROUP BY subregion1_name
HAVING total_confirmed_cases > 1500
ORDER BY total_confirmed_cases DESC
```
<br/>

## Consulta 4: taxa de letalidade
Crie uma consulta para responder à pergunta: "Qual foi a taxa de letalidade dos casos na Itália em May de 2020?". A taxa de letalidade dos casos é definida aqui como (total de mortes / total de casos confirmados) * 100. Crie uma consulta que retorne a taxa em May de 2020 e tenha os seguintes campos na saída: total_confirmed_cases, total_deaths, case_fatality_ratio.

Estas são as colunas para referência:

* country_name
* cumulative_confirmed
* cumulative_deceased

```SQL
SELECT sum(cumulative_deceased) AS total_deaths, sum(cumulative_confirmed) AS total_confirmed_cases, (sum(cumulative_deceased))/(sum(cumulative_confirmed))*100 AS case_fatality_ratio 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name ="Italy" AND date BETWEEN "2020-05-01" AND "2020-05-31"
```
<br/>

## Consulta 5: identificação de um dia específico
Crie uma consulta para responder à pergunta: "Em que dia o total de mortes passou de 10000 na Itália?". A consulta precisa retornar a data no formato aaaa-mm-dd.

Estas são as colunas para referência:

* country_name
* cumulative_deceased

```SQL
SELECT date
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name ="Italy" AND cumulative_deceased>10000
ORDER BY date ASC
LIMIT 1
```

<br/>

## Consulta 6: dias sem novos casos
A consulta a seguir foi criada para identificar o número de dias na Índia entre 25, Feb 2020 e 11, March 2020 sem aumento no número de casos confirmados. No entanto, ela não está sendo executada corretamente. Para ver o resultado, você precisa atualizar a consulta para ela ser concluída:
```SQL
WITH india_cases_by_date AS (
  SELECT date, SUM( cumulative_confirmed ) AS cases
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE country_name ="India" AND date between '2020-02-25' and '2020-03-11'
  GROUP BY date
  ORDER BY date ASC 
), india_previous_day_comparison AS (
 SELECT date, cases, LAG(cases) OVER(ORDER BY date) AS previous_day, cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
 FROM india_cases_by_date
)
 
SELECT count(*)
FROM india_previous_day_comparison
WHERE net_new_cases = 0
```

<br/>

## Consulta 7: taxa de duplicação
Com base na consulta anterior, crie uma consulta para descobrir as datas em que os casos confirmados aumentaram mais de 5% em comparação ao dia anterior (indicando a taxa de duplicação de aproximadamente sete dias) nos EUA entre 22 de março e 20 de abril de 2020. A consulta precisa retornar a lista de datas, os casos confirmados no dia pesquisado e no anterior e a porcentagem de aumento de casos entre os dias. Nomeie os campos da resposta da seguinte forma: Date, Confirmed_Cases_On_Day, Confirmed_Cases_Previous_Day e Percentage_Increase_In_Cases.

```SQL
WITH us_cases_by_date AS (
  SELECT date, SUM(cumulative_confirmed) AS cases
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE country_name="United States of America" AND date between '2020-03-22' and '2020-04-20'
  GROUP BY date
  ORDER BY date ASC 
 )
, us_previous_day_comparison AS 
(SELECT date, cases, LAG(cases) OVER(ORDER BY date) AS previous_day, cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases, (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
select Date, cases as Confirmed_Cases_On_Day, previous_day as Confirmed_Cases_Previous_Day, percentage_increase as Percentage_Increase_In_Cases
from us_previous_day_comparison
where percentage_increase > 5
```

<br/>

## Consulta 8: taxa de recuperação
Crie uma consulta para listar as taxas de recuperação dos países em ordem decrescente (20 no máximo) até 10 de maio de 2020. 
Restrinja a consulta aos países com mais de 50 mil casos confirmados. A consulta precisa retornar estes campos: country, recovered_cases, confirmed_cases, recovery_rate.

Estas são as colunas para referência:

* country_name
* cumulative_confirmed
* cumulative_recovered

```SQL
WITH us_cases_by_date AS (
  SELECT date, SUM(cumulative_confirmed) AS cases
  FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE country_name="United States of America" AND date between '2020-03-22' and '2020-04-20'
  GROUP BY date
  ORDER BY date ASC 
 )
, us_previous_day_comparison AS 
(SELECT date, cases, LAG(cases) OVER(ORDER BY date) AS previous_day, cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases, (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
select Date, cases as Confirmed_Cases_On_Day, previous_day as Confirmed_Cases_Previous_Day, percentage_increase as Percentage_Increase_In_Cases
from us_previous_day_comparison
where percentage_increase > 20
```

<br/>

## Consulta 9: taxa de crescimento diário cumulativo (CDGR)
A consulta a seguir tenta calcular a CDGR em June 10, 2020 na França desde que o primeiro caso foi informado. O primeiro registro de caso foi em 24 de janeiro de 2020. A CDGR é calculada desta forma:

((last_day_cases/first_day_cases)^1/days_diff)-1)

Em que:

* last_day_cases é o número de casos confirmados em 10 de maio de 2020.
* first_day_cases é o número de casos confirmados em 24 de janeiro de 2020.
* days_diff é o número de dias entre 24 de janeiro e 10 de maio de 2020.

A consulta não está sendo executada corretamente. Corrija o erro.

```SQL
WITH
  france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="France"
    AND date IN ('2020-01-24',
      '2020-06-10')
  GROUP BY
    date
  ORDER BY
    date)
, summary as (
SELECT
  total_cases AS first_day_cases,
  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)
select first_day_cases, last_day_cases, days_diff, POW((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr
from summary
```