
# Model
Se puede hablar de 3 `model` en Ember:
* El concepto de model, que es una represantación del domain de datos de la aplicación.
* model(): es el hook que esta disponible en la rutas y que es automáticamente llamado por Ember en respuesta a una petición de una ruta
* La propiedad model que se encuentra en el controller/template. `model()` puede ser asincrono. Una vez que se ha devuelto el valor o se ha resuelto la promesa, el resultado es establecido en la propiedad `model` del controller/template.

https://emberigniter.com/5-essential-ember-concepts/


# Acceder al model dentro de una route

No se puede acceder al modelo dentro de una ruta con `this.get('model')`, porque con esto se estaría accediendo al hook `model()` de la route. Por lo que:
* Enviar la action desde el controller/templae
* `this.modelFor(this.routename)`
* Se puede acceder al modelo por meido de `this.controller.get('model')`.
* Se puede implementar el hook `setupController` para acceder al modelo, y guardar este en una propiedad.

https://stackoverflow.com/questions/35630842/ember-how-to-get-route-model-inside-route-action/35631373

