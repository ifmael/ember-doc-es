# Templating
En esta sección se aprenderá  sobre como escribit HTML,  como añadirle iteracción, estilos, contenido que cambia dinámicamente...

## Escribiendo HTML plano
Las plantillas en Ember vienen con superpoderes, pero vamos algo de HTML básico. En los ficheros con extensión `hbs` en Ember se puede escribir HTML como si fuersen ficheros `html`. `hbs` provie de la herramienta Handlebar.  Por ejemplo en el fichero `application.hbs`, se puede escribir:

```html
<h1>Starting simple</h1>
<p>
  This is regular html markup inside an hbs file
</p>
```

Cuando se arranca la aplicación con `ember serve`, los templates son compilados en algo que el motor de renderizado de Ember procesa de una manera más fácil. El compilador nos ayuda a capturar errores, por ejemplo.

## Tipos de templates
Hay 2 tipos de templates, Route template y Component template.

Una Route template determina que se tiene que mostrar cuando se visita una particular URL. Un component template tiene un trozo de contenido que puede ser reusado en múltiples lugares a través de la app, como botones y formularios.

En una app se pueden encontrar muchos templates por diferentes lugares de la estructura de ficheros, esto es así para mejorar la organización de la app. La mejor manera de saber si una template es de Route o de un Componente, es mirar el path donde se encuentra el template.

## Genear template
Por medio de Ember cli, podemos generar estos ficheros y asegurarnos de que están situados en el lugar correcto.

```bash
ember generate component my-component-name
ember generate route my-route-name
```

## Restricciones en las templates
Las aplicaciones actuales tienen docenas de ficheros, que en combinación unos con otros generan algo que el navegador puede entender. En Ember no se necesita configurar nada para que esto funcione, pero a cambio se tienen que seguir algunas reglas en los ficheros.

No se pueden usar etiquetas `script` dentro de un template ya que se deben de usar actions o hooks del ciclo de vida de los componentes para hacer que la aplicación responda a las interacciones de usuario y los nuevos datos..
No se pueden añadir enlaces locales de `css` dentro de un fichero `hbs`. Los estilos deben de estar localizados en la ruta `app/styles`. Para los fichoes `css` incluidos en este directorio, se puede crear multiples hojas de estilos y usar `import` para unirlas juntas.


