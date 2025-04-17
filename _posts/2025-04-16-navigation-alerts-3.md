---
title: "Navegación con SwiftUI. Alerts."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

Seguimos con una nueva entrega. Esta vez empezamos con las alertas. Habíamos dejado el proyecto de forma que, podíamos borrar un elemento de la lista. Sin embargo, la forma de hacerlo, era un poco peligrosa porque una vez hecho el gesto de borrar, el elemento se borra sin opción a recuperarlo. Una opción para hacer más seguro el proceso es levantar una alerta.

Para crear una alerta usaremos el siguiente modifier en SwiftUI.

```swift
nonisolated
func alert<S, A, M, T>(
    _ title: S,
    isPresented: Binding<Bool>,
    presenting data: T?,
    @ViewBuilder actions: (T) -> A,
    @ViewBuilder message: (T) -> M
) -> some View where S : StringProtocol, A : View, M : View
```

* Gracias a `nonisolated` la alerta puede aceptar actores (y otros tipos) siempre y cuando no se modifiquen dentro.
* `title` representa el título de la alerta.
* `isPresented` representa el binding que determina si se muestra o no la alerta.
* `presenting` representa un objeto que le podemos pasar para que luego los ViewBuilders puede acceder a su valor y así aprovecharse. Hay que ir con cuidado, ya que si le pasamos este objeto pero luego dicho objeto vale `nil` los ViewBuilders no funcionarán.
* `actions` van a ser los botones que tenga la alerta.
* `message` será el mensaje, el subtítulo que tendrá la alerta.

Empecemos por mostrar la alerta. Por el siguiente código después de la lista en ExercisesView.

```swift
.alert(
  "Delete exercise",
  isPresented: .constant(true)
  ) {
    Button("Remove", role: .destructive) {}
  } message: {
    Text("Are you sure that you want to remove this exercise?")
  }
```

Analicemos un momento el código anterior. Se muestra un título y un subtítulo y luego dos botones, Remove en rojo destructivo y un botón cancelar. El botón de cancelar se muestra siempre, pero si quieres customizarlo tienes que definirlo.

```swift
.alert(
  "Delete exercise",
  isPresented: .constant(true)
  ) {
    Button("Remove", role: .destructive) {}
    Button("Back", role: .cancel) {}
  } message: {
    Text("Are you sure that you want to remove this exercise?")
  }
```

Si defines `presenting`como nil, entonces la alerta se mostrará rara. `presenting` siempre tiene que tener un valor asignado. Si el mensaje de un botón de una alerta, entonces la alerta se posiciona verticalmente. Esto hay que tenerlo en cuenta si localizamos los textos en varios idiomas. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.alert.1)

```swift
.alert(
  "Delete exercise",
  isPresented: .constant(true),
  presenting: Exercise.fake(type: .cycling)
) { exercise in
  Button("Remove \(exercise.type.rawValue)", role: .destructive) {}
} message: { exercise in
  Text("Are you sure that you want to remove this exercise? Total distance \(exercise.distanceFormatted).")
}
```

Ahora que ya tenemos claro como funciona una alerta, vamos a montar la lógica realizando una serie de cambios. Lo primero que vamos a hacer es cambiar el método de borrar porque no termina de funcionar bien si luego mostramos una alerta. Usaremos un swipeAction, que es otro modifier en el que nos permite mostrar botones si hacemos un gesto, bien sea en la izquierda o en la derecha. Borraremos el `onDelete` ya que no lo vamos a usar. Y pondremos el siguiente código justo despues de ExerciseRowView. Ahora cada elemento del listado tendrá un gesto con un botón destructivo, muy parecido al `onDelete`. Pero, sigue teniendo el mismo fallo. Si realizamos el swipe y luego presionamos el botón, aún no teniendo ninguna acción asignada, la celda se elimina y se pierde la síncronía con el modelo.

```swift
.swipeActions(edge: .trailing) {
  Button(role: .destructive) {
  } label: {
    Label("Trash", systemImage: "trash")
  }
}
```

> Nota: Para ocultar la alerta, tendrás que poner en el isPresented el valor .constant(false)

Para simular un botón funcional que nos sirva perfectamente en nuestra aplicación, haremos lo siguiente. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.alert.2)

```swift
.swipeActions(edge: .trailing) {
  Button {
  } label: {
    Label("Trash", systemImage: "trash")
  }
  .tint(.red)
}
```

Ahora crearemos el siguiente método en ExercisesModel.

```swift
func onDeleteButtonTapped(_ exercise: Exercise) {
}
```

Para que funcione la alerta, vamos a necesitar un `isPresented` y un `presenting`, los crearemos.

```swift
var alertIsPresented = false
var exerciseToRemove: Exercise?
```

Con lo cual, el método anterior, se quedará de la siguiente forma.

```swift
func onDeleteButtonTapped(_ exercise: Exercise) {
  alertIsPresented = true
  exerciseToRemove = exercise
}
```

> Nota: No te preocupes, por ahora, si el código anterior se puede simplificar. Si, pero lo haremos más adelante.

Ahora, en la vista, ya podemos meter esta función en el `swipeActions`.

```swift
.swipeActions(edge: .trailing) {
  Button {
    model.onDeleteButtonTapped(exercise)
  } label: {
    Label("Trash", systemImage: "trash")
  }
  .tint(.red)
}
```

Al igual que con la TabView, isPresented es un binding, por lo tanto, tenemos que actualizar el modelo a un @Bindable.

```swift
@Bindable var model: ExercisesModel
```

Por último, ya podemos actualizar la alerta. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.alert.3)

```swift
.alert(
  "Delete exercise",
  isPresented: $model.alertIsPresented,
  presenting: model.exerciseToRemove
) { exercise in
  Button("Remove", role: .destructive) {}
} message: { exercise in
  Text("Are you sure that you want to remove this exercise? Total distance \(exercise.distanceFormatted).")
}
```

Ya casi tenemos el flujo entero. Solo nos falta realizar la última acción. Que es borrar la fila seleccionada para borrar. Empezamos por el modelo, añadiremos la siguiente función. Tendremos que buscar el ejercicio a eliminar y luego el indice donde se encuentre en el listado de ejercicios. Borraremos el elemento y luego eliminaremos la alerta.

```swift
func confirmDeleteButtonTapped() {
  guard
    let exercise = exerciseToRemove,
    let index = exercises.firstIndex(where: { exercise.id == $0.id })
  else { return }
  exercises.remove(at: index)
  alertIsPresented = false
  exerciseToRemove = nil
}
```

Finalmente, usaremos la función dentro del botón de la alerta. Envolveremos dicha función para que se realiza una animación al borrar el ejercicio. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.alert.4)

```swift
withAnimation {
  model.confirmDeleteButtonTapped()
}
```

Pues ya lo tendríamos. Seguiremos con dialogs para actualizar el tipo de ejercicio. Pero eso será en otra entrada.