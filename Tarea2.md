# Tarea 2 
##### Jorge Alberto Fernández Villarreal
##### 172140
#

### 4. Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?

```javascript
db.tweets.aggregate([ 
    { $group: { _id: { "lang": "$user.lang", "hour": { $substr: ["$created_at", 11, 2] } }, "counter": { $count: {} } } }, 
    { $match: { "_id.hour": "20" } }, 
    { $sort: {"counter":-1}}
]);
```

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
```javascript
db.tweets.aggregate([
    
    {$addFields: {
            "hora":{  $toInt: {$substr: ["$created_at", 11, 2] } }    
        }
    },
    {$addFields: 
        { "tipo":
            { 
                $switch : { 
                    branches: [
                        {case: { $or: [{$gte : ["$hora", 7]}, {$lte : ["$hora", 18]}] }, then: "7 AM - 6:59:59 PM"}                    
                    ],
                    default : "7 PM - 6:59:59 AM"
                }                                     
            }
        }
    },
    {$group: {_id: {"lang":"$user.lang", "Intervalo":"$tipo"}, "usuarios":{$count: {}}}},
    {$lookup: {from:"primarydialects","localField":"_id.lang","foreignField":"lang","as":"language"}},
    {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
    {$project: {_id:1, usuarios:1, "fulllocale.languages":1}},
    {$sort: {"usuarios": -1}}
]);
   
```

### 7. De qué país son los tuiteros más famosos de nuestra colección?

```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"pais"}},
  {$group: {_id:{"country":"$pais.country","seguidores":"$user.friends_count"}}},
  {$sort: {"_id.seguidores":-1}},
  {$limit : 15}
]);
```
