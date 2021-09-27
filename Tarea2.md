# Tarea 2 
##### Jorge Alberto Fernández Villarreal
##### 172140
#

### 4. Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?


De 03:00:00 am a 18:59:59 pm:

```javascript
db.tweets.aggregate([
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[3-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9]/}},
  {$group: {_id:"$fulllocale.languages", "num": {$count:{}}}}
])
```
##### 978 tweets


##### Consideramos como "noche" de 7pm a 3am: 
De 7:00:00 pm a 11:59:59 pm:
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3])+:+[0-5]+[0-9]+:+[0-5]+[0-9]/}},
  {$group: {_id:"$fulllocale.languages", "num": {$count:{}}}}
])
```
##### 1407 tweets


De 00:01 am a 3 am
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[0-3])+:+[0-5]+[0-9]+:+[0-5]+[0-9]/}},
  {$group: {_id:"$fulllocale.languages", "num": {$count:{}}}}
])
```
##### 0 tweets

#### Por los resultados que obtuvimos, podemos ver que los hispanohablantes twittean más en la noche (7pm a 3 am)



### 5. Cómo podemos saber de dónde son los tuiteros que más tiempo tienen en la plataforma?
```javascript
db.tweets.aggregate([
    {$addFields: { "user.created_at": { "$toDate": "$user.created_at" }}},
    {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
    {$group: {_id:{"user_created_at":"$user.created_at","location":"$user.location","country":"$countryy.country"}}},
    {$sort: {"_id.user_created_at":1}}
]);
```

### 6. En intervalos de 7:00:00pm a 6:59:59am y de 7:00:00am a 6:59:59pm, de qué paises la mayoría de los tuits?


7:00:00 am a 6:59:59 pm:

```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[7-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9]/}},
  {$group: {_id:"$countryy.country", "num": {$count:{}}}},
  {$sort: {"num":-1}}
])

```
##### 10848 tweets



7:00:00 pm a 6:59:59 am: 

```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3]|[0]+[0-6])+:+[0-5]+[0-9]+:+[0-5]+[0-9]/}},
  {$group: {_id:"$countryy.country", "num": {$count:{}}}},
  {$sort: {"num":-1}}
])
```
##### 13984 tweets



### 7. De qué país son los tuiteros más famosos de nuestra colección?

```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"pais"}},
  {$group: {_id:{"country":"$pais.country","seguidores":"$user.friends_count"}}},
  {$sort: {"_id.seguidores":-1}},
  {$limit : 15}
]);
```
