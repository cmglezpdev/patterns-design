# Patrones de Diseños 


## Indice

- Patrones de diseño
    - [__Factory (Fábrica):__](#factory) Proporciona una interfaz para poder crear diferentes tipos de objetos.
    - __Abstract Factory (Fábrica Abstracta):__ Proporciona una interficie para crear familias de objetos relacionados o que dependen entre si, sin especificar clases concretas.
    - [__Singleton (Único):__](#singleton) Garantiza que una clase solo tenga una instancia y proporciona un punto de acceso global a ella.
    - __Adapter (Adaptador):__ Convierte la interfaz de una clase en otra distinta que es la que espera los clientes. Permite que cooperen clases que de otra manera no podrian por tener interfaces incompatibles.
    - __Bridge (Puente):__ Desacopla una abstracción de su implementación, de manera que ambas puedan variar de forma independiente.
    - __Strategy (Estrategia):__ Define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables. Permite que un algoritmo varíe independientemente de los clientes que lo usan.
    - __Builder (Constructor):__ Separa la construcción de un objeto complejo de su representación, de forma que el mismo proceso de construcción pueda crear diferentes representaciones.
    - __Command (Orden):__ Encapsula una petición en un objeto, permitiendo asi parametrizar a los clientescon distintas peticiones, encolar o llevar un registro de las peticiones y poder deshacer las operaciones.
    - __Composite (Compuesto):__ Combina objetos en estructuras de arbol para representar jerarquías de parte-todo. Permite que los clientes traten de manera uniforme a los objetos individuales y a los compuestos.
    - __Decorator (Decorador):__ Añade dinámicamente nuevas responsabilidades a un objeto, proporcionando una alternativa flexible a la herencia para extender la funcionalidad.
    - __Observer (Observador):__ Define una dependencia de uno a muchos entre objetos, de forma que cuando un objeto cambie de estado se notifica y se actualizan automáticamente todos los objetos que dependen de él.
    - __Strategy (Estrategia):__ Define una familia de algoritmos, encapsula cada uno de ellos y lo hace intercambiables. Permite que un algoritmo varie independientemente de los clientes que lo usan.




### Factory

En un videojuego existen mucha variedad de enemigos cada uno con sus características y su dificulad para los jugadores. Por lo tanto tendremos varias clases que representan los diferentes enemigos, por ejemplo podemos tener:
```ts
class Boo implements Entity {

}

class Koopa implements Entity {

}

class Goomba implements Entity {

}
```

Estos enemigos son entidades del juego que interactuan con el jugador por eso heredan de `Entity`. Esta es una clase que define cualquier cosa en el juego que tenga lógica o que tenga algún tipo de interacción con el juego.

```ts
class Entity {
    updateLogic(): void;
}
```

Que todos los enemigos hereden de `Entity` nos permite aplicar __polimorfismo__ y poder tratar a todas las entidades por igual. Por ejemplo, actualizar el estado de los mismos:

```ts
function processGameLogic( entities: Entity[] ) {
    for( let entity in entitites ) {
        entity.updateLogic();
    }
}
```

Ahora digamos que queremos generar enemigos aleatorios en el juego para sorprender al jugador. En principio simplemente podemos usar una fución que reciba un número y con algunos `if` decidir que tipo de enemigos queremos crear.

```ts
function spawnEnemy(random: Number): Entity {
    if( random <= 0.40 ) return new Boo();
    if( random <= 0.80 ) return new Koopa();
    return new Goomba(); 
}

```

Esto en principio funciona, pero que pasa si tenemos 100 enemigos?, si queremos cambiarle la dificultad a algunos jugadores y a otros no? Tendríamos que modificar el código, además de que este método para generar 100 enemigos con constructores diferentes no es una buena estrategia. Aquí es donde entre el patrón __Factory__. La __Factory__ es la clase encargada de tener la lógica de construir enemigos, la cual podemos reutilizar y usarla de forma _polimórfica_.

En nuestro caso pudiéramos tener dos clases _Factory_ para generar enemigos:

```ts
class RandomEnemyFactory implements EnemyFactory {
    createEnemy(): Entity {
        // retrun enemies randomly
    }
}

class RandomDificultEnemyFactory implements EnemyFactory {
        createEnemy(): Entity {
        // retrun only difficult enemies
    }
}
```

Asi podemos tener tantas clases clases como generadores diferentes de enemigos querramos. En el ejemplo tenemos `RandomEnemyFactory` que devuelve enemigos aleatoriamente y `RandomDificultEnemyFactory` que devuelve enemigos aleatoriamente pero con mayor dificultad para el jugador. Todas estas clases implementan la interfaz `EnemyFactory` que es la interfaz que define el método _creador de enemigos_.

```ts
interface EnemyFactory {
    createEnemy(): Entity
}
```

De esta forma podemos usando polimorfismo generar los enemigos de la forma que deseemos.

```ts
class Game {
    private enemyFactory: EnemyFactory;

    // get any EnemyFactory 
    constructor( enemyFactory: EnemyFactory ) {
        this.enemyFactory = enemyFactory;
    }

    function gameLogic() {
        // ...more code here
        if( shouldSpawnEnemy() ) {
            // generate an enemy
            const enemy = this.enemyFactory.createEnemy();
        }
    }
}
```

Asi podemos pasarle por constructor cualquier tipo de generación de enemigos y usarla en nuestro código sin necesidad de modiciar el mismo. Por tanto podemos tener cientos de formas diferentes de generar enemigos y nuestro código sigue siendo el mimsmo.


### Abstract Factory

Supongamos ahora que queremos crear un juego como __Super Mario__. Tenemos toda la funcionalidad del juego pero queremos que existan varios tipos de gráficos y que sea el usuario el que escoja el que desee, pongamos como ejemplo los gráficos que tiene una nintendo __Gameboy__ y un __NintendoDS__. Si usamos el patrón __Factory__ tendremos clase _Factory_ que crea el elemeto en los distintos gráficos. Si fuéramos por ejemplo a renderizar el _bloque interrogante_ tendríamos un código así:

```ts
interface Factory<T> {
    createItem(): T;
}

interface BloqueInterrogante {
    spawnItem();
    render();
}

class GameboyBloqueInterrogante implements BloqueInterrogante {
}

class NintendoDSBloqueInterrogante implements BloqueInterrogante {
}

class GameboyBloqueFactory implements Factory<BloqueInterrogante> {
    createItem(): BloqueInterrogante {
        return new GameboyBloqueInterrogante();
    }
}

class NintendoDSBloqueFactory implements Factory<BloqueInterrogante> {
    createItem(): BloqueInterrogante {
        return new NintendoDSBloqueInterrogante();
    }
}
```

Pero que pasa si queremos renderizar otro elemento? Por ejemplo una moneda. Tendremos que crear otra _Factoy_ y crear las dos implementaciones respectivas a cada tipo de gráfico. Por tanto para __n__ elementos tendremos __n__ _Factories_, una para cada elementos. Esto es muy engorrozo, ademas de que nada nos evita que podamos renderizar dos elementos con gráficos diferentes, y esto no debería pasar.

Aquí es donde entra __Abstract Factory__. Con esta podemos crear familias de objetos relacionados o independientes entre ellos, sin especificar su clase concreta. O sea, en vez de tener __n__ factories, una para cada elemento, solo dos, un para cada tipo de gráficos, y que esta sea quien nos devuelva todos los items en un solo tipo de gráfico, evitando asi que se puedan usar items en distintos gráficos al mismo tiempo.

```ts
interface AbstractFactory {
    createCoin(): Coin;
    createBloqueInterrogante(): BloqueInterrogante;
}

class GameBoyItemFactory implements AbstractFactory {
    createCoin() {
        return new GameBoyCoin();
    }
    createBloqueInterrogante() {
        return new GameBoyBloqueInterrogante();
    }
}

class NintendoDSItemFactory implements AbstractFactory {
    createCoin() {
        return new NintendoDSCoin();
    }
    createBloqueInterrogante() {
        return new NintendoDSBloqueInterrogante();
    }
}
```

Ahora solo tenemos dos factories, uno para cada gráfico y es esta la que nos devuelve la instancia de cada una de los tipos de items en el gráfico correspondiente.

Hay muchos ejemplo en donde se puedan usar este patrón. Un ejemplo es al crear aplicaciones en donde puedas seleccionar entre __Dark Mode__ o __Light Mode__, aqui podemos crear dos factories, una para cada modo y que estos nos devuelvan los objectos como se verían en ambos modos, y asi cuando se cambia entre un modo u otro, solo tenemos que cambiar la instancia de la _Factory_ de un modo por la instancia del otro, y esto se puede hacer muy facilmente porque estamos usando _polimorfismo_.

Otro ejemplo es en los __frameworks__ de cración de aplicaciones multiplataformas. Por ejemplo, en __React Native__ creamos un único codigo y este al compilarse a __android__ o a __IOS__ adapta la interfaz de los objetos en dependencia de a donde se esté compilando, si a __android__ o a __IOS__. Una implementación de esto puede ser tener dos factories, una para cada plataforma, y que cuando se valla a compliar la aplicación, se le pasa la __Factory__ correspondiente y este devuelve los elementos adaptados ya al tipo de dispositivo.

### Singleton

Este patrón lo que nos permite es poder crear una única instancia de un objeto y que no se pueda de ninguna manera volver a instanciar el objecto, para poder garantizar así que siempre se use la misma instancia en todas las partes en donde esta es utilizada.

Para esto hacemos el constructor de la clase privada, asi sólo la propia clase tiene el control de poder crear una instancia de la misma, evitando que cualquiera fuera de la clase pueda crear una instancia de la misma.

Y ahora para obtener la instancia de la clase crearemos el método `getInstance` que se encargará de crear una única instancia de la clase. Asi cuando deseemos acceder a la instancia, simplemente llamamos a `Singleton.getInstance()` y este nos devuelve siempre la misma.

```ts
class Singleton {
    private static instance: Singleton;

    private constructor() {}

    public static getInstance(): Singleton {
        if( !Singleton.instance ) 
            Singleton.instante = new Singleton();
        return Singleton.instance;
    }
}
```
