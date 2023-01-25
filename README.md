# Patrones de Diseños 


## Indice

- Catálogo de patrones de diseño
    - __Abstract Factory (Fábrica Abstracta):__ Proporciona una interfaz para crear familias de objetos relacionados o que dependen entre si, sin especificar clases concretas.
    - __Adapter (Adaptador):__ Convierte la interfaz de una clase en otra distinta que es la que espera los clientes. Permite que cooperen clases que de otra manera no podrian por tener interfaces incompatibles.
    - __Bridge (Puente):__ Desacopla una abstracción de su implementación, de manera que ambas puedan variar de forma independiente.
    - __Strategy (Estrategia):__ Define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables. Permite que un algoritmo varíe independientemente de los clientes que lo usan.
    - __Builder (Constructor):__ Separa la construcción de un objeto complejo de su representación, de forma que el mismo proceso de construcción pueda crear diferentes representaciones.
    - __Command (Orden):__ Encapsula una petición en un objeto, permitiendo asi parametrizar a los clientescon distintas peticiones, encolar o llevar un registro de las peticiones y poder deshacer las operaciones.
    - __Composite (Compuesto):__ Combina objetos en estructuras de arbol para representar jerarquías de parte-todo. Permite que los clientes traten de manera uniforme a los objetos individuales y a los compuestos.
    - __Decorator (Decorador):__ Añade dinámicamente nuevas responsabilidades a un objeto, proporcionando una alternativa flexible a la herencia para extender la funcionalidad.
    - __Observer (Observador):__ Define una dependencia de uno a muchos entre objetos, de forma que cuando un objeto cambie de estado se notifica y se actualizan automáticamente todos los objetos que dependen de él.
    - __Singleton (Único):__ Garantiza que una clase solo tenga una instancia y proporciona un punto de acceso global a ella.
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