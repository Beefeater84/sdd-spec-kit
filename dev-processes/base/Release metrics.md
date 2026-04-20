2026-04-20 12:22
Tags: #devops 


Метрики релиза помогают нам понять - как какие-то изменения в процессе влияют на скорость разработки. 

## Понятия

Commit to PR - от коммита до PR
Commit to MR - от коммита до Merdge request


## Фреймворки для отслеживания релиза

Самый популярный: DORA (DevOps Research and Assessment). 

Метрики:
- **Deployment Frequency**
- **Lead Time for Changes** - время от коммита до продакшин
- **Change Failure Rate** - % неудачных изменений. Какая доля деплоев приводит к сбоям, когда нужен hotfix или rallback
- Time to Restore Service - Как быстро команда восстанавливает сервис после инцидента

|Метрика|Elite|Low|
|---|---|---|
|Deployment Frequency|Несколько раз в день|Раз в полгода и реже|
|Lead Time|< 1 часа|6+ месяцев|
|Change Failure Rate|< 5%|46–60%|
|Time to Restore|< 1 часа|1–6 месяцев|


----
### Zero Links


----
### Links
[DORA | Get Better at Getting Better](https://dora.dev/)

----
### Source