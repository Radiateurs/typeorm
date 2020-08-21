# Mettre en chache les requête

Vous pouvez mettre en cache les resultats émisent par une variété de méthodes de `QueryBuilder`. 
Les méthodes sont les suivantes: `getMany`, `getOne`, `getRawMany`, `getRawOne` et `getCount`.
Vous pouvez également mettre en cache les resultats issuent par un `Respository` en utilisants les méthodes suivantes: `find`, `findAndCount`, `findByIds`, and `count`.

Pour activer la mise en cache, vous devez l'activer de manière éxplicite dans vos options de connection:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: true
}
```

Quand vous activez le mise en cache pour la première fois,
vous devez synchroniser votre schema de base de données (en utilisant le CLI, migrations ou l'option de cnnection `synchronize`).

Ensuite, dans `QueryBuilder` vous pouvez activer la mise en cache des requêtes pour n'importe quelle requête:

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(true)
    .getMany();
```

L'équivalent en utilisant une requête dans un `Repository`:
```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: true
    });
```

Ceci va éxecuter une requête pour charger touts les utilisateurs administrateurs et mettre en cache le résultat.
La prochaine fois que vous éxecutez le même code, il chargera touts les utilisateurs administrateurs à partir du cache. 
Par défaut, la durée de vie du cache est égale à `1000 ms` c-à-d 1 seconde.
Cela veux dire que le cache sera invalide 1 seconde après que le `QueryBuilder` soit appelé.
En pratique, cela veux dire que si des utilisateurs ouvrent la page utilisateurs 150 fois dans un interval de 3 secondes,
 seulement 3 requêtes seront éxecutés pendant cette période.
Touts les utilisateurs insérés pendant la durée de vie du cache (1 seconde) ne sera pas retournés à l'utilisateur.

Vous pouvez changer la durée de vie du cache manuelement via le `QueryBuilder`:

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(60000) // 1 minute
    .getMany();
```

Ou via le `Repository`:

```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: 60000
    });
```

Ou globalement dans les options de connection:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        duration: 30000 // 30 seconds
    }
}
```

Vous pouvez également ajouter un identifiant de cache dans le `QueryBuilder`:

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache("users_admins", 25000)
    .getMany();
```

Ou dans le `Repository`:

```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: {
            id: "users_admins",
            milliseconds: 25000
        }
    });
```

Cela vous donne un contrôle totale de votre cache, 
par exemple nettoyer les résultats mis en cache quand vous ajoutez un nouvel utilisateur:

```typescript
await connection.queryResultCache.remove(["users_admins"]);
```


Par défaut, TypeORM utilise une table séparée appelé `query-result-cache` et ajoute toutes les requêtes et résultats dedans. 
Le nom de la table est configurable, vous pouvez donc le changer en assignant le nom souhaité dans la propriété `tableName`.
Exemple:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "database",
        tableName: "configurable-table-query-result-cache"
    }
}
```

Si une seule table de la base de données pour l'enregistrement du cache ne vous est pas suffisant,
vous pouvez changer le type de cache pour "redis" ou "ioredis" et TypeORM va, à la place, enregistrer toutes les archives du cache dans redis.
Exemple:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "redis",
        options: {
            host: "localhost",
            port: 6379
        }
    }
}
```

"options" peut être [node_redis specific options](https://github.com/NodeRedis/node_redis#options-object-properties) ou [ioredis specific options](https://github.com/luin/ioredis/blob/master/API.md#new-redisport-host-options) en fonction du type que vous souhaitez utiliser.

Dans le cas où vous voulez vous connecter au redis-clister en utilisant les fonctionnalités de cluster de IORedis, vous pouvez tout à fais le faire de la façon suivante:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    cache: {
        type: "ioredis/cluster",
        options: {
            startupNodes: [
                {
                    host: 'localhost',
                    port: 7000,
                },
                {
                    host: 'localhost',
                    port: 7001,
                },
                {
                    host: 'localhost',
                    port: 7002,
                }
            ],
            options: {
                scaleReads: 'all',
                clusterRetryStrategy: function (times) { return null },
                redisOptions: {
                    maxRetriesPerRequest: 1
                }
            }
        }
    }
}
```

Notez que vous pouvez toujours utliser les options en tant que premier argument du constructeur du cluster de IORedis.

```typescript
{
    ...
    cache: {
        type: "ioredis/cluster",
        options: [
            {
                host: 'localhost',
                port: 7000,
            },
            {
                host: 'localhost',
                port: 7001,
            },
            {
                host: 'localhost',
                port: 7002,
            }
        ]
    },
    ...
}
```

Si aucuns des distributeurs de cache intégrés ne vous satisfaits, vous pouvez alors spécifié votre propre distributeur en utilisant une fonction factory appelée `provider` qui implémente l'interface `QueryResultCache`:

```typescript
class CustomQueryResultCache implements QueryResultCache {
    constructor(private connection: Connection) {}
    ...
}
```

```typescript
{
    ...
    cache: {
        provider(connection) {
            return new CustomQueryResultCache(connection);
        }
    }
}
```

Vous pouvez utiliser `typeorm cache:clear` pour effacer tout ce qui est contenu dans le cache.
