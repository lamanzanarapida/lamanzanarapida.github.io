---
title: "Navegación con SwiftUI. Tabs."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

Empezamos un nuevo tema hablando de una de las partes más importantes en cualquier aplicación móvil, la navegación. 

Lo primero vamos a definir que es la navegación. En resumen, navegación es cómo un usuario se mueve a través de su móvil. Por defecto, Apple nos ofrece una serie de navegaciones predefinidas. Éstas pueden ser alerts, dialogs, sheets o links [^1]. Pero también pueden haber otros tipos de navegaciones, incluidas navegaciones construidas de forma propia, como podría ser un menú lateral. El diseño adecuado de cada uno de estos objetos es fundamental para el buen desarrollo de la aplicación. Un mal diseño, puede ocasionar efectos no deseados en la construcción, por ejemplo, de un deeplink.

En esta serie vamos a construir una aplicación por defecto, donde nos centraremos, entre otras cosas de la navegación y el primer objeto que vamos a construir es un TabView. 

El objeto TabView tiene el objetivo de mostrar varias pantallas a modo de pestañas con su contenido. Por ejemplo, en un iPhone, una TabView tendrá las pestañas en la parte inferior de la pantalla.

[^1]: Utilizamos los nombres en inglés, por claridad, a la hora de recordar dichos conceptos en el código.

| ![iOS](https://docs-assets.developer.apple.com/published/3b94e40a70cf512f26ed1900ff2e5e71/Enhancing-your-app-content-with-tab-navigation-iOS@2x.png){:height="30%" width="30%"} | 
|:--:| 
| El objeto TabView se muestra en un iPhone en la parte inferior de la pantalla. |


En otros dispositivos, como puede ser en macOS o con visionOS su visualización puede ser diferente.

| ![macOS](https://docs-assets.developer.apple.com/published/d86d64648482d8e9501242564d8d27ba/Enhancing-your-app-content-with-tab-navigation-macOS@2x.png){:height="50%" width="50%"} ![visionOS](https://docs-assets.developer.apple.com/published/0c8e9497ef4f8b04e1a24a43058e293b/Enhancing-your-app-content-with-tab-navigation-visionOS.png){:height="50%" width="50%"} | 
|:--:| 
| Tanto en macOS, como en visionOS, el TabView se muestran de formas diferentes. |

Pero aunque, la visualización de una difiera, el código para construir una TabView es el mismo.

## Creando el proyecto

A fecha de la redacción de este artículo, usamos Xcode con versión 16.3. Crearemos un proyecto nuevo para iOS.

* En Product Name escribiremos Do Exercise.
* En Interface, seleccionaremos SwiftUI.
* En Language, seleccionaremos Swift.
* En Testing System, seleccionaremos None. Crearemos tests cuando tengamos suficiente código para testear, de momento, mejor no crear el target de test.
* En Storage, seleccionaremos None.

Pulsaremos Next y seleccionaremos el directorio donde se guardará el proyecto. Tendremos algo parecido a la siguiente imagen. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.tab.1){:target="_blank"}

![iOS](https://docs-assets.developer.apple.com/published/a311c2ad23768cbb9cf5bc4056725ba4/creating-your-app-s-interface-with-swiftui-1@2x.png
)

## Creando la TabView

Borraremos el contenido del body del ContentView y definiremos un TabView.

```swift
struct ContentView: View {
  var body: some View {
    TabView {}
  }
}
```

> Nota: Es importante saber entender la definición de cada estructura que estamos utilizando con SwiftUI. Y la mejor manera es utilizar la documentación oficial. En este caso, TabView se inicia como una función que devuelve un builder de TabContent, pero también con un valor seleccionado. De momento, nos quedaremos con la primera versión. [Ver en Apple](https://developer.apple.com/documentation/swiftui/tabview){:target="_blank"}

```swift
struct TabView<SelectionValue, Content> where SelectionValue : Hashable, Content : View

init<C>(
  @TabContentBuilder<Never> content: () -> C
) where SelectionValue == Never, Content == TabContentBuilder<Never>.Content<C>, C : TabContent

init<C>(
    selection: Binding<SelectionValue>,
    @TabContentBuilder<SelectionValue> content: () -> C
) where Content == TabContentBuilder<SelectionValue>.Content<C>, C : TabContent
```

Si todo ha ido bien, veremos una pantalla blanca en la preview y, en la parte inferior, se percibirá nítidamente la TabView, con un color gris claro. Ahora vamos a escribir un par de items dentro de la TabView. Vamos a necesitar dos, pero puedes probar a meter unos cuantos más.

> Note: ¿Por qué no pruebas a añadir más de 5 items dentro de una TabView? Verás lo que ocurre.

Si todo ha ido bien, tendremos algo similar a esto. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.tab.2){:target="_blank"}

```swift
struct ContentView: View {
var body: some View {
  TabView {
    Tab("Exercises", systemImage: "figure.run") {
    } 
    Tab("Settings", systemImage: "gear") {
    }
  }
}
```

## Creando la primera funcionalidad con TabView

Ahora ya tenemos algo funcional, podemos hacer tap en cada uno de los dos items que hemos creado, pero la pantalla no muestra nada. Podemos hacer algo sencillo para empezar. 

* Crearemos un texto `Exercises` en el tab `Exercises`.
* Crearemos un botton con texto `Settings` en el tab de `Settings`. 

El objetivo es que al presionar en el botón, automáticamente nos lleve al primer tab.

```swift
struct ContentView: View {
  var body: some View {
    TabView {
      Tab("Exercises", systemImage: "figure.run") {
        Text("Exercises")
      }
      Tab("Settings", systemImage: "gear") {
        Button("Tap to exercises") {
        }
      }
    }
  }
}
```

Lo primero que tenemos que hacer es crear un estado para que la TabView sepa donde está. Eso lo conseguiremos con el parámetro de la TabView `selection`. `selection` es un Binding de un SelectionValue, que puede representar un Hashable. No vamos a entrar en lo que significa un Hashable por debajo, pero diremos que un entero o un string es un hashable. Entonces, podemos utilizar un entero como valor de `selection`.

Y para guardar el valor del estado, utilizaremos la forma más sencilla en SwiftUI, @State. Finalmente, nos quedará asignar un estado a cada uno de los tabs. Esto se consigue con el parametro `value`. Utilizaremos, 0 para Exercises y 1 para Settings. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.tab.3){:target="_blank"}

```swift
struct ContentView: View {
  @State var selectedTab = 0

  var body: some View {
    TabView(selection: $selectedTab) {
      Tab("Exercises", systemImage: "figure.run", value: 0) {
        Text("Exercises")
      }
      Tab("Settings", systemImage: "gear", value: 1) {
        Button("Tap to exercises") {
          selectedTab = 0
        }
      }
    }
  }
}
```

Ya lo tenemos, podemos jugar a darle al tab de Settings. Luego, al botón Tap to exercises y volver al tab de Exercises. Fácil, ¿no?

## Creando previews

Vamos a quedarnos aquí un momento. Es una muy buena práctica utilizar las previews para cada uno de los estados que tengamos en nuestra aplicación. Tiene sentido, pues, crear un par de previews con cada una de las tabs.

```swift
#Preview("Exercises") {
  ContentView(
    selectedTab: 0
  )
}

#Preview("Settings") {
  ContentView(
    selectedTab: 1
  )
}
```

## Creando los primeros tests

De la misma forma que tener previews listas para ver nuestro trabajo sin tener que depender de un simulador o dispositivo es buena idea. Lo es también, crear nuestros primeros tests. Pero tenemos un problema. @State dentro de la vista, no nos permite testear la funcionalidad. Para ello, crearemos un modelo de la vista `ContentModel` y sacaremos la lógica hacia dicho modelo.

```swift
@Observable
final class ContentModel {
  var selectedTab: Int

  init(
    selectedTab: Int = 0
  ) {
    self.selectedTab = selectedTab
  }

  func exercisesButtonTapped() {
    selectedTab = 0
  }
}

struct ContentView: View {
  @Bindable var model: ContentModel

  var body: some View {
    TabView(selection: $model.selectedTab) {
      Tab("Exercises", systemImage: "figure.run", value: 0) {
        Text("Exercises")
      }
      Tab("Settings", systemImage: "gear", value: 1) {
        Button("Tap to exercises") {
          model.exercisesButtonTapped()
        }
      }
    }
  }
}

#Preview("Exercises") {
  ContentView(
    model: ContentModel()
  )
}

#Preview("Settings") {
  ContentView(
    model: ContentModel(
      selectedTab: 1
    )
  )
}
```

Ahora ya podemos crear nuestro primer test. 

Lo primero que haremos es crear el target de test llamado Unit Testing Bundle. Hay varias formas de crear un target. Puedes ir a File/New/Target y de ahí buscar el target de test. Si quieres crear el target con un nombre particular, bien puedes, pero también puedes dejar el que te ofrece, así como el resto de propiedades que salen. Eso si, asegúrate que el Testing System es Swift Testing.

Ya podemos crear el primer test. Una forma de crear buenos tests es utilizar la notación Given-When-Then tanto para los nombres de los tests como en el cuerpo de la función de test. Se puede dejar una linea vacía que representa la separación entre el given, el when y el then. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.tab.4){:target="_blank"}

```swift
import Testing
@testable import NavigationDoExercise

struct NavigationDoExerciseTests {
  @Test func givenSettingsTabSelectedWhenTapToExercisesThenExercisesTabSelected() async throws {
    let model = ContentModel(
      selectedTab: 1
    )

    model.exercisesButtonTapped()

    #expect(model.selectedTab == 0)
  }
}
```

## Primer problema, primer refactor

Ya tenemos la primera funcionalidad terminada, con nuestra primera navegación funcionando. Pero, tenemos un problema. selection acepta cualquier entero. ¿Qué pasa si, en vez de un 0, ponemos, sin querer un -1? O, ¿en vez de un 1, un 10?. Si, es un problema potencial futuro y en programación, tenemos una regla que es utilizar justo los tipos con los valores necesarios, ni uno menos, ni uno más.

Como dijimos antes, selectionValue puede representar un Hashable, esto es, bien puede ser un número o un string, y también puede ser un tipo. Un tipo que represente dos valores concretos. Esto lo vamos a conseguir, creando un enum.

```swift
enum Tab {
	case exercises, settings
}
```

¿Te atreves a actualizar el proyecto tu solo? [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.tab.5){:target="_blank"}