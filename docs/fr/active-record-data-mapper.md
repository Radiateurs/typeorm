# Enregistrement Actif vs Data Mapper

- [Enregistrement Actif vs Data Mapper](#enregistrement-actif-vs-data-mapper)
  - [Qu'est ce qu'est l'Enregistrement Actif?](#quest-ce-quest-lenregistrement-actif)
  - [Qu'est ce qu'est le modèle de Mappeur de Données?](#quest-ce-quest-le-modèle-de-mappeur-de-données)
  - [Lequel devrais-je utiliser?](#lequel-devrais-je-utiliser)

## Qu'est ce qu'est l'Enregistrement Actif?

Dans TypeORM, vous pouvez utiliser à la fois l'Enregistrement Actif et les Mappeur de Données.

En utilisant les modèles d'Enregistrement Actif, vous definissez toutes vos méthodes de requêtes dans un modèle et vous pouvez enregistrer, effacer et charger les objets en utilisants les méthodes de ce modèle.

Pour faire plus simple, le modèle d'Enregistrement Actif est une approche pour acceder votre base de données dans vos modèles.
Vous pouvez en apprendre plus sur l'Enregistrement Actif sur [Wikipedia](https://fr.wikipedia.org/wiki/Patron_de_conception).

Exemple:

```typescript
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User extends BaseEntity {
       
    @PrimaryGeneratedColumn()
    id: number;
    
    @Column()
    firstName: string;
    
    @Column()
    lastName: string;
    
    @Column()
    isActive: boolean;

}
```

Toutes les entités dans un Enregistrement actif doivent étendre la classe `Base-Entity`, qui offrent les méthodes permettant la manipulation des entités.
Exemple de manipulation avec de telles entités:
```typescript

// exemple comment enregister une entite EA (Enregistrement Actif)
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await user.save();

// exemple comment effacer une entite EA (Enregistrement Actif)
await user.remove();

// exemple comment charger une entite EA (Enregistrement Actif)
const users = await User.find({ skip: 2, take: 5 });
const newUsers = await User.find({ isActive: true });
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });
```

La classe `BaseEntity` contient la plupart des méthodes que possède le standard `Repository`.
En général, si vous utilisez des entités d'Enregistrement Actif, vous n'aurez pas besoin d'utiliser `Repository` ou `EntityManager`.

Imaginons que vous voulez créer une fonction qui retourne les utilisateurs par leur prénoms et noms de famille.
Nous pouvons créer cette fonction en tant que méthode statique de la classe `User`:

```typescript
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User extends BaseEntity {
       
    @PrimaryGeneratedColumn()
    id: number;
    
    @Column()
    firstName: string;
    
    @Column()
    lastName: string;
    
    @Column()
    isActive: boolean;
    
    static findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }

}
```

Et nous pouvons l'utiliser comme toutes les autres méthodes:

```typescript
const timber = await User.findByName("Timber", "Saw");
```

## Qu'est ce qu'est le modèle de Mappeur de Données?

Dans TypeORM, vous pouvez utiliser à la fois l'Enregistrement Actif et les Mappeur de Données.

En utilisant l'approche du Mappeur de Données, vous définissez toutes vos méthodes de requêtes dans des classes séparées
appelées "Répertoires", et vous pouvez enregistrer, effacer et charger vos objets en utilisant ces répertoires.
Dans le Mappeur de Données, vos entités sont très stupides - elles ne définise que leur propriétés et peuvent avoir des
méthodes "dummy" (stupide en anglais).

De maniére simple, le Mappeur de Données est une approche d'accès à votre base de données dans des répertoires au lieux de 
passer par des modèles.
Vous pouvez en apprendre plus à propos des Mappeurs de Données sur [Wikipedia](https://fr.wikipedia.org/wiki/Data_mapping).

Exemple:

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {
       
    @PrimaryGeneratedColumn()
    id: number;
    
    @Column()
    firstName: string;
    
    @Column()
    lastName: string;
    
    @Column()
    isActive: boolean;

}
```
Exemple pour manipuler une telle entité:

```typescript
const userRepository = connection.getRepository(User);

// exemple comment enregistrer une entité MD (Mappeur de Données)
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await userRepository.save(user);

// exemple comment effacer une entité MD
await userRepository.remove(user);

// exemple comment charger une entité MD
const users = await userRepository.find({ skip: 2, take: 5 });
const newUsers = await userRepository.find({ isActive: true });
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

Imaginons que vous voulez créer une fonction qui retourne les utilisateurs par leur prénoms et noms de famille.
Nous pouvons créer une telle fonction dans un répertoire personalisé.

```typescript
import {EntityRepository, Repository} from "typeorm";
import {User} from "../entity/User";

@EntityRepository()
export class UserRepository extends Repository<User> {
       
    findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }

}
```

Et l'utiliser de cette manière:

```typescript
const userRepository = connection.getCustomRepository(UserRepository);
const timber = await userRepository.findByName("Timber", "Saw");
```

Apprenez-en plus sur les répertoires personnalisés [ici](custom-repository.md).

## Lequel devrais-je utiliser?

C'est à vous de choisir.
Les deux stratégies ont leur avantages et inconvéniens.

Une chose que nous devons garder en tête avec le dévelopement logiciel est comment nous allons maintenir nos applications.
L'approche `Mappeur de données` aide la maintenabilité, ce qui est plus efficace sur les grosses applications.
L'approche `Enregistrement actif` aide à garder les choses simple ce qui marche bien dans les petites applications.
Et la simplicité la clé pour une meilleur maintenabilité.
