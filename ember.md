# EMBER

## Core concepts
![alt text](https://guides.emberjs.com/images/ember-core-concepts/ember-core-concepts.png "Logo Title Text 1")

### Routa y Manejador de rutas
No importa como se establezca una ruta, la primera cosa que ocurre ember mapea la ruta con el mapeador de rutas. El manejador de rutas se encarga de:
* Renderizar una plantilla.
* Cargar el modelo para que este disponible en la plantilla.

### Template
Ember utiliza `Handlebars`.

### Models
Representan los estados persistentes.

### Components
Mientras que las `templates` definen como se muestra la interfaz, los componentes controlan como se comporta la interfaz de usuario. Los componentes consisten en:
* Un template.
* Fichero escrito en JS


### Hooks
Se refiere al termino `hooks` para métodos que son automáticamente llamados dentro de la aplicacion de Ember. Algunos ejemplos pueden ser los hooks del ciclo de vida (como `willRender()`), router hooks (como `model`)

## Object en Ember
La objeto `Ember` extiende del objeto JS para proporcionar funcionalidades del core. La mayoría de los objetos en Ember extiende `EmberObject` como rutas, modelos, mixins... La principal razón para esto es que un objeto `Ember` puede ser visto para cambios.

### Clases e Instancias

#### Definiendo clases
Para definir una nueva clase Ember, se usa el método `extend` proporcionado por `EmberObject`

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  say(thing) {
    alert(thing);
  }
});
```

Se puede crear cualquier subclase desde cualquier clase existente llamando a `extend`

```javascript
import Component from '@ember/component';

export default Component.extend({})
```

### Sobrescribir métodos del padre
Con la definicion de una subclase, se puede sobreescribir métodos pero tambien poder acceder a la implementación que tiene el padre para ese método llamando al método `_super()`. 

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  say(thing) {
    alert(`${this.name} says: ${thing}`);
  }
});

const Soldier = Person.extend({
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ', sir!' to the variable `thing` passed in
    this._super(`${thing}, sir!`);
  }
});

let yehuda = Soldier.create({
  name: 'Yehuda Katz'
});

yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
```

### Creando instancias
Cuando se define una clase, se pueden generar instancias de la clase por medio de `create()`. Cuando se crea una instancia, se puede inicializar valores para las propiedades pasandoselos al método.

```javascript
let tom = Person.create({
  name: 'Tom Dale'
});
```

No se pueden redefinir propiedades calculadas (computed properties), solo se debe de usar con propiedades simples.

### Inicializando instancias
Cuando se crea una intancia, el método `init()` es invocado automáticamente. Es el lugar idóneo para implementar una configuración requerida por las intancias.

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  init() {
    alert(`${this.name}, reporting for duty!`);
  }
});
```

Si se está sobreescribiendo una clase del framework, como `Ember.component`, y se sobreescribe el método `init()`, se tiene que realizar una llamada a `this._super(...arguments)`.

### Accediendo a propiedades del objeto
Cuando se lee una propiedad de un objeto, en la mayoría de los cassos se utiliza la notación de JS, `myObject.myProperty`.

Los objetos proxy Ember son una excepción a esta regla. Si se está trabajando con los objetos proxy Ember, incluyendo proxy promesas para Ember Data, se tiene que utilizar `get()`. Por ejemplo se va definir un modelo para `blogPost`:

```javascript
import Model from 'ember-data/model';
import attr from 'ember-data/attr';
import { hasMany } from 'ember-data/relationships';

export default Model.extend({
  title: attr('string'),
  body: attr('string'),
  comments: hasMany('comment', { async: true }),
});
```

Para acceder al titulo de un post, se puede escribir `blogPost.title`, mientras que solo la notación `blogPost.get('comments')` devolverá los comentarios del post.

Siempre utiliza el método `set()` para actualizar los valores de las propiedades. Esto propagará el valor cambiado a las propiedades calculadas, observer, templates, etc... Si solo se utiliza la notación `.` de JS, las propidades calculadas no se recalcularán, los observer no se lanzarán y la plantilla no se actualizará.

```javascript
import EmberObject from '@ember/object';

const Person = EmberObject.extend({
  name: 'Robert Jackson'
});

let person = Person.create();

person.name; // 'Robert Jackson'
person.set('name', 'Tobias Fünke');
person.name; // 'Tobias Fünke'
```

### Propiedades calculadas (computed properties)
En una palabra, las propiedades calculadas te permiten declaras funciones como propiedades. Se crea una definiendo una computed property como una función, la cuál Ember llamará automáticamente cuando se accede a la propiedad

#### Computed properties en acción
```javascript
import EmberObject, { computed } from '@ember/object';

Person = EmberObject.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: computed('firstName', 'lastName', function() {
    return `${this.firstName} ${this.lastName}`;
  })
});

let ironMan = Person.create({
  firstName: 'Tony',
  lastName:  'Stark'
});

ironMan.fullName; // "Tony Stark"
```

Se ha declarado una computed property `fullName`, con `firstname` and `lastName` como propiedades de las que depende. La primera vez que se accede `fullName`, la función es llamada y el resultado cacheado. Las siguientes veces que se llame se lee desde la cache. Cualquier cambio en las propiedades de las que depende invalida la cache por lo que la siguiente vez que se acceda a la propiedad, la función se volverá a llamar.


#### Una computed property solo se recalcula cuando es consumida
Las propiedaes son consumidas en 2 modos:

* Cuando son accedidas, via `ironMan.fullName`
* Cuando es referenciado en handlebars template para ser renderizado, `{{ironMan.fullName}}`

Fuera de estas circunstancias, el código en esta propiedad no se ejecuta, incluso si una de las propiedades de las que depende ha cambiado. Se ha modificado la función anterior para que muestre un mensaje cuando se invoca la función.

```javascript
import Ember from 'ember';
…
  fullName: computed('firstName', 'lastName', function() {
    console.log('compute fullName'); // track when the property recomputes
    return `${this.firstName} ${this.lastName}`;
  })
…
```

Usando la nueva propiedad, solo se muestra el mensaje despues de `fullName` haya accedido, y entonces solo si una de las propiedades `firstName` and `lastName` han sido cambiadas previamente.

```javascript
let ironMan = Person.create({
  firstName: 'Tony',
  lastName:  'Stark'
});

ironMan.fullName; // 'compute fullName'
ironMan.set('firstName', 'Bruce') // no console output

ironMan.fullName; // 'compute fullName'
ironMan.fullName; // no console output since dependencies have not changed
```

#### Multiples dependencias del mismo objeto
```javascript
import EmberObject, { computed } from '@ember/object';

const Home = EmberObject.extend({
  location: {
    streetName: 'Evergreen Terrace',
    streetNumber: 742
  },

  address: computed('location.streetName', 'location.streetNumber', function() {
    return `${this.location.streetNumber} ${this.location.streetName}`;
  })
});

let home = Home.create()

home.address // 742 Evergreen Terrace
home.set('location.streetNumber', 744)
home.address // 744 Evergreen Terrace
```

Es importante observar las propiedades del objeto, no el objeto en sí ya que si se utiliza el objeto como dependencia, las computed properties no serán recalculadas cuando las propiedades `streetName` y `streetNumber` cambien.

```javascript
import EmberObject, { computed } from '@ember/object';

const Home = EmberObject.extend({
  location: {
    streetName: 'Evergreen Terrace',
    streetNumber: 742
  },

  address: computed('location', function() {
    return `${this.location.streetNumber} ${this.location.streetName}`;
  })
});

let home = Home.create()

home.address // 742 Evergreen Terrace
home.set('location.streetNumber', 744)
home.address // 742 Evergreen Terrace
home.set('location', {
  streetName: 'Evergreen Terrace',
  streetNumber: 744
})
home.address // 744 Evergreen Terrace
```

Ya que las propiedades `streetNumber`  y `streetName` son propiedades para el objeto `location`, se puede utilizar la sintaxis abreviada.

```javascript
import EmberObject, { computed } from '@ember/object';

const Home = EmberObject.extend({
  location: {
    streetName: 'Evergreen Terrace',
    streetNumber: 742
  },

  address: computed('location.{streetName,streetNumber}', function() {
    return `${this.location.streetNumber} ${this.location.streetName}`;
  })
});
```

#### Encadenando computed properties
Se pueden crear computed properties para crear computed properties. 

```javascript
import EmberObject, { computed } from '@ember/object';

Person = EmberObject.extend({
  firstName: null,
  lastName: null,
  age: null,
  country: null,

  fullName: computed('firstName', 'lastName', function() {
    return `${this.firstName} ${this.lastName}`;
  }),

  description: computed('fullName', 'age', 'country', function() {
    return `${this.fullName}; Age: ${this.age}; Country: ${this.country}`;
  })
});

let captainAmerica = Person.create({
  firstName: 'Steve',
  lastName: 'Rogers',
  age: 80,
  country: 'USA'
});

captainAmerica.get('description'); // "Steve Rogers; Age: 80; Country: USA"
```

#### Establcer computed properties
Se puede definir que debe de hacer Ember cuando cuando se setea una computed property. Si se intenta setear un valor para una propiedad para una computed property, se tiene que especificar el nombre de la propiedad y su valor. Se debe de devolver el nuevo valor previsto de la computed property desde la función set

```javascript
import EmberObject, { computed } from '@ember/object';

Person = EmberObject.extend({
  firstName: null,
  lastName: null,

  fullName: computed('firstName', 'lastName', {
    get(key) {
      return `${this.firstName} ${this.lastName}`;
    },
    set(key, value) {
      let [firstName, lastName] = value.split(/\s+/);
      this.set('firstName', firstName);
      this.set('lastName',  lastName);
      return value;
    }
  })
});


let captainAmerica = Person.create();
captainAmerica.set('fullName', 'William Burnside');
captainAmerica.firstName; // William
captainAmerica.lastName; // Burnside
```

#### Computed property macros
Algunas computed properties son muy comunes, por lo que Ember proporciona algunos tipos, los cuales son formas abreviadas de expresar tipos de computed properties.

```javascript
import EmberObject, { computed } from '@ember/object';
import { equal } from '@ember/object/computed';

Person = EmberObject.extend({
  fullName: 'Tony Stark',

  isIronManLongWay: computed('fullName', function() {
    return this.fullName === 'Tony Stark';
  }),

  isIronManShortWay: equal('fullName', 'Tony Stark')
});
```

### Computed Properties and Aggregate Data.
Cuando una computed property depende de un Array, hay unos métodos extra que se necesitan usar para correctamente reconocer cuando el contenido del array cambia. Los Arrays tienen dos propiedades especiales que se pueden agregar a las propiedades de los Arrays para seguir los cambios en ellos, `[]` y `@each`.

#### `[]`
Algunas veces una computed property necesita actualizarse cuando un item es añadido, eliminado o reemplazado en el Array. Para estos casos se puede utilizar la key `[]` para indicarle a la propiedad cuando sea necesario. 

```javascript
import { A } from '@ember/array';
import EmberObject, { computed } from '@ember/object';
import Component from '@ember/component';

export default Component.extend({
  todos: null,

  init() {
    this._super(...arguments);
    this.set('todos', A([
      EmberObject.create({ title: 'Buy food', isDone: true }),
      EmberObject.create({ title: 'Eat food', isDone: false }),
      EmberObject.create({ title: 'Catalog Tomster collection', isDone: true }),
    ]));
  },

  titles: computed('todos.[]', function() {
    return this.todos.mapBy('title');
  })
```

La computed property tiene a la propiedad `todos.[]` como dependencia, la cuál indica a Ember que actualice los binding y lance los observer cuando alguno de los siguientes eventos ocurra:
* La propiedad `todos` cambie a un diferente Array.
* Un item es añadido al Array `todos`.
* Un item es eliminado del Array `todos`.
* Un item  es reemplazado en el Array `todos`.

En particular, la computed property no se actualizará si un todo es mutado, para eso se utiliza la key especial `@each`.

#### `@each`
Algunas veces se tiene una computed property cuyos valores depende de las propiedades de los elementos de un Array. Por ejemplo, por ejemplo se puede tener un array de `todos`, en la que se quiere calcular la lista de `todos` incompletos basados en las propiedad `isDone`.

```javascript
import { A } from '@ember/array';
import EmberObject, { computed } from '@ember/object';
import Component from '@ember/component';

export default Component.extend({
  todos: null,

  init() {
    this._super(...arguments);
    this.set('todos', A([
      EmberObject.create({ isDone: true }),
      EmberObject.create({ isDone: false }),
      EmberObject.create({ isDone: true }),
    ]));
  },

  incomplete: computed('todos.@each.isDone', function() {
    let todos = this.todos;
    return todos.filterBy('isDone', false);
  })
});
```

Con la instruccion `todos.@each.isDone` se le indica a Ember para actualizar bindings y lanzar los observer cuando algun de los siguientes eventos ocurre:
* La propiedad `todos` es cambie a un diferene Array.
* Se añade un item al Array `todos`.
* Un item es eliminado del Array `todos`.
* Un item es remplazado en el array `todos`.
* La propiedad `isDone` en cualquiera de los objetos en el array `todos` ha cambiado.

#### Multiples dependencias.
Es importante añadir que la propiedad `@each` puede depender de mas de una key. Por ejemplo, si se esta usando `Ember.computed` para ordenar un array por multiples keys, se puede declarar la dependencia  con : `todos.@each.{priority,title}`


#### Cuando usar `[]` y `@each`
Ambos actualizarán sus binding cuando el array es remplazado y cuando los miembros del array cambian. Si se está utilizando `@each` para una particular propiedad, no es necesario utilizar `[]`

```javascript
//specifying both '[]' and '@each' is redundant here
incomplete: computed('todos.[]', 'todos.@each.isDone', function() {
...
})
```

Usar `@each` es mas expresivo que `[]`, por lo que usa `[]` si no se quiere observar cambios en las propiedades en los miembros individuales de un array.


#### Computed Property Macros
Ember proporciona una computed property `computed.filterBy`, que es una abreviación de la anterior computed property

```javascript
import { A } from '@ember/array';
import EmberObject, { computed } from '@ember/object';
import { filterBy } from '@ember/object/computed';
import Component from '@ember/component';

export default Component.extend({
  todos: null,

  init() {
    this._super(...arguments);
    this.set('todos', A([
      EmberObject.create({ isDone: true }),
      EmberObject.create({ isDone: false }),
      EmberObject.create({ isDone: true }),
    ]));
  },

  incomplete: filterBy('todos', 'isDone', false)
});
```

En ambos ejemplos anteriores, `incomplete` es un array que contiene la lista de incompletos `todos`.

```javascript
import TodoListComponent from 'app/components/todo-list';

let todoListComponent = TodoListComponent.create();
todoListComponent.get('incomplete.length');
// 1
```

Si se cambia la propiedad `isDone`, la propiedad `incomplete` es actualizado automáticamente.

```javascript
import EmberObject from '@ember/object';

let todos = todoListComponent.get('todos');
let todo = todos.objectAt(1);
todo.set('isDone', true);

todoListComponent.get('incomplete.length');
// 0

todo = EmberObject.create({ isDone: false });
todos.pushObject(todo);

todoListComponent.get('incomplete.length');
// 1
```

Observa que `@each` solo funciona en un nivel profundo. No se puede encadenar como `todos.@each.owner.name` o `todos.@each.owner.@each.name`.


#### `[]` vs `@each`
Cuando no importa si tienes que observar una propiedad individual, es mejor utilizar `[]` en lugar de `@each`. 

Varios macros que proporciona Ember sobre computed, utilizan `[]` para implementar los comunes casos de uso. Por ejemplo, para crear una computed property que mapee las propiedades de un array, se puede utilizar `Ember.computed.map` o construirlo tu mismo

```javascript
import EmberObject, { computed } from '@ember/object';

const Hamster = EmberObject.extend({
  excitingChores: computed('chores.[]', function() {
    return this.chores.map(function(chore, index) {
      return `CHORE ${index + 1}: ${chore.toUpperCase()}!`;
    });
  })
});

const hamster = Hamster.create({
  chores: ['clean', 'write more unit tests']
});

hamster.excitingChores; // ['CHORE 1: CLEAN!', 'CHORE 2: WRITE MORE UNIT TESTS!']
hamster.chores.pushObject('review code');
hamster.excitingChores; // ['CHORE 1: CLEAN!', 'CHORE 2: WRITE MORE UNIT TESTS!', 'CHORE 3: REVIEW CODE!']
```

Y usando la macro de Ember se puede hacer de esta manera:

```javascript
import EmberObject from '@ember/object';
import { map } from '@ember/object/computed';

const Hamster = EmberObject.extend({
  excitingChores: map('chores', function(chore, index) {
    return `CHORE ${index + 1}: ${chore.toUpperCase()}!`;
  })
});
```

El macro `map` espera usar un array, por lo que no tiene necesidad de añadir `[]`. Sin embargo, en la construccion propia si requiere utilizarlo.


### Observer
Los observer son bastante utilizados por los desarrolladores en Ember, pero la mayoría de las problemas que se afrontan en una app en Ember, se puede realizar mejormente con computed properties.

Ember soporta observar cualquier propiedad, incluso computed properties.

Los observer deben contener un comportamiento que reaccione a los cambios de cualquier propiedad. Son especialmente utiles cuando se necesita realizar algun comportamiento específico despues de que el binding haya finalizado de sincronizar.

```javascript
import EmberObject, {
  computed,
  observer
} from '@ember/object';

Person = EmberObject.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: computed('firstName', 'lastName', function() {
    return `${this.firstName} ${this.lastName}`;
  }),

  fullNameChanged: observer('fullName', function() {
    // deal with the change
    console.log(`fullName changed to: ${this.fullName}`);
  })
});

let person = Person.create({
  firstName: 'Yehuda',
  lastName: 'Katz'
});

// observer won't fire until `fullName` is consumed first
person.get('fullName'); // "Yehuda Katz"
person.set('firstName', 'Brohuda'); // fullName changed to: Brohuda Katz
```

Puesto que la computed property `fullName` depende de la propiedad `firstName`, actualizando `firstName` se lanzará tambien `fullName`.

#### Observer y sincronía.
Los observer en ember son síncronos, por lo que ellos son lanzados tan pronto una de las propiedades de las que observa cambia. Por este motivo, se pueden introducir bugs en las propiedades que aún no están sincronizadas.

```javascript
import { observer } from '@ember/object';

Person.reopen({
  lastNameChanged: observer('lastName', function() {
    // The observer depends on lastName and so does fullName. Because observers
    // are synchronous, when this function is called the value of fullName is
    // not updated yet so this will log the old value of fullName
    console.log(this.fullName);
  })
});
```

Este comportamiento sincrono puede hacer que los observer sean lanzados multiples veces cuando los observer observan multiples propiedades.

```javascript
import { observer } from '@ember/object';

Person.reopen({
  partOfNameChanged: observer('firstName', 'lastName', function() {
    // Because both firstName and lastName were set, this observer will fire twice.
  })
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

Para solventar estos problemas, se debe de hacer uso de `Ember.run.once()`. Esto asegura que cualquier procesosamiento que se necesite hacer solo se realizara una vez y ocurrira en el siguiente loop en el que todos los bidning sean sincronizados.

```javascript
import { observer } from '@ember/object';
import { once } from '@ember/runloop';

Person.reopen({
  partOfNameChanged: observer('firstName', 'lastName', function() {
    once(this, 'processFullName');
  }),

  processFullName() {
    // This will only fire once if you set two properties at the same time, and
    // will also happen in the next run loop once all properties are synchronized
    console.log(this.fullName);
  }
});

person.set('firstName', 'John');
person.set('lastName', 'Smith');
```

#### Observer y la inicializacion
Los observer nunca se ejecutarán hasta que la inicialización del objeto este completa.

Si se necesita lanzar un observer en el proceso de inicialización, no se puede confiar en los efectos colaterales de `set`. En su lugar,se especifica que un observer debe ejecutarse despues de `init` con `Ember.on()`.

```javascript
import EmberObject, { observer } from '@ember/object';
import { on } from '@ember/object/evented';

Person = EmberObject.extend({
  init() {
    this.set('salutation', 'Mr/Ms');
  },

  salutationDidChange: on('init', observer('salutation', function() {
    // some side effect of salutation changing
  }))
});
```

#### Computes Properties no consumidas no activan a los observer
Si nunca `get` una computed property, el observer no será lanzado incluso si depende de la key que ha cambiado

Esto no suele afectar a la aplicación puesto que computed properties son casi siempre observadas al mismo tiempo que son consumidas. Por ejemplo, se obtiene el valor de una computed property, se pone en el DOM, y luego lo observas para que se pueda actualizar el DOM una vez que la propiedad cambia.

Si se necesita observar una computed property pero aun no se ha obtenido, obtenla en el método `init()`.

#### Fuera de la definicion de la clase.
Se puede añadir observer a un objeto fuera de su definicion, usando `addObserver`

```javascript
person.addObserver('fullName', function() {
  // deal with the change
});
```

