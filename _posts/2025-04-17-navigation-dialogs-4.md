---
title: "Navegación con SwiftUI. Dialogs."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

Una vez terminada la feature de eliminar un ejercicio usando alertas, procedemos a hacer una nueva feature. Necesitamos poder cambiar el tipo de ejercicio de una forma más o menos usable.

La idea será crear un nuevo gesto en la celda, y esta vez, en la izquierda, se abrirá lo que antiguamente se decía una `action sheet`, ahora en SwiftUI, se llama `confirmationDialog` o, simplemente, `dialog`.

Nos damos cuenta que la signatura de un confirmationDialog es muy similar a una alerta.

```swift
nonisolated
func confirmationDialog<A, T>(
    _ titleKey: LocalizedStringKey,
    isPresented: Binding<Bool>,
    titleVisibility: Visibility = .automatic,
    presenting data: T?,
    @ViewBuilder actions: (T) -> A,
    @ViewBuilder message: (T) -> M
) -> some View where A : View
```

Aparece un nuevo parámetro, que marca la visibilidad del mensaje. 

Vamos a implementar la feature. Empecemos por añadir un nuevo swipeActions.

```swift
.swipeActions(edge: .leading) {
    Button {
        model.dialogButtonTapped(exercise)
    } label: {
        Label("Flag", systemImage: "flag")
    }
    .tint(.green)
}
```

Ahora, al igual que en el episodio anterior, implementaremos la función dialogButtonTapped.

```swift
var dialogIsPresented = false
var exerciseToUpdate: Exercise?

func dialogButtonTapped(_ exercise: Exercise) {
    dialogIsPresented = true
    exerciseToUpdate = exercise
}
```

Con esta función, ya podemos implementar nuestro `dialog`.

```swift
.confirmationDialog(
    "Change exercise type",
    isPresented: $model.dialogIsPresented,
    titleVisibility: .visible
) {
    Button("Cycling") {
        withAnimation { model.confirmDialogButtonTapped(type: .cycling) }
    }
    Button("Swimming") {
        withAnimation { model.confirmDialogButtonTapped(type: .swimming) }
    }
    Button("Running") {
        withAnimation { model.confirmDialogButtonTapped(type: .running) }
    }
    Button("Walking") {
        withAnimation { model.confirmDialogButtonTapped(type: .walking) }
    }
} message: {
    Text("Choose a new option for the exercise type.")
}
```

Y, finalmente, ya podemos implementar, la función `confirmDialogButtonTapped`. Al modificar el tipo de ejercicio, en el modelo tendremos que poner `var` en vez de `let`. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.dialog.1)

```swift
func confirmDialogButtonTapped(type: Exercise.Mode) {
    guard
        let exercise = exerciseToUpdate,
        let index = exercises.firstIndex(where: { exercise.id == $0.id })
    else { return }
    exercises[index].type = type
    dialogIsPresented = false
    exerciseToUpdate = nil
}
```

# Refactorizar el modelo de datos

La próxima feature va a ser crear un ejercicio, pero antes, pensemos un poco, cómo tenemos el modelo de datos. Para dos navegaciones posibles, estamos utilizando 4 variables.

```swift
var alertIsPresented = false
var dialogIsPresented = false
var exerciseToRemove: Exercise?
var exerciseToUpdate: Exercise?
```

Con lo cual, estamos ofreciendo al sistema la posibilidad de crear estados imposibles, que no tienen sentido en la aplicación. Lo que tenemos que hacer es pensar en un tipo de datos, que simplifique el uso de 4 variables. Y lo tenemos, vamos a utilizar un enum para mapear los diferentes estados posibles.

```swift
enum Destination {
    case alert(Exercise)
    case dialog(Exercise)
}
```

Con lo cual, de las 4 variables que tenía el modelo, nos vamos a quedar con una sola variable.

```swift
var destination: Destination?
var exercises: [Exercise]

enum Destination {
    case alert(Exercise)
    case dialog(Exercise)
}

init(
    destination: Destination? = nil,
    exercises: [Exercise] = []
) {
    self.destination = destination
    self.exercises = exercises
}
```

Nos queda, resolver todos los errores que tenemos en la aplicación. Empecemos por la parte más sencilla. Por homogeneizar las dos funciones, renombramos `onDeleteButtonTapped` por `alertButtonTapped`

```swift
func dialogButtonTapped(_ exercise: Exercise) {
    destination = .dialog(exercise)
}

func alertButtonTapped(_ exercise: Exercise) {
    destination = .alert(exercise)
}
```

Nos quedan dos funciones, y tenemos que hacer un `case let` para obtener el valor del `case` que estamos tratando.

```swift
func confirmAlertButtonTapped() {
    guard
        case let .alert(exercise) = destination,
        let index = exercises.firstIndex(where: { exercise.id == $0.id })
    else { return }
    exercises.remove(at: index)
    destination = nil
}

func confirmDialogButtonTapped(type: Exercise.Mode) {
    guard
        case let .dialog(exercise) = destination,
        let index = exercises.firstIndex(where: { exercise.id == $0.id })
    else { return }
    exercises[index].type = type
    destination = nil
}
```

Por último, queda actualizar la vista. Pero tenemos un problema. `isPresented` recibe un `Binding<Bool>`. La solución es implementar un binding propio.

Un binding tiene un constructor que recibe el valor actual `get` y la posibilidad de actualizar su valor con `set`. Entonces, si queremos construir un Binding propio que funcione con un enum, tendremos que hacer lo siguiente.

```swift
isPresented: Binding(
    get: {
        if case .alert = model.destination {
            return true
        }
        return false
    }, set: { isPresented in
        if !isPresented {
            model.destination = nil
        }
}
),
```

* Por un lado, si el case representa una alerta, devolveremos el valor true, en caso contrario, el false.
* Si el binding, cambia a valor false, solo en ese caso, le diremos al modelo que la alerta ha desaparecido.

```swift
isPresented: Binding(
    get: {
        if case .dialog = model.destination {
            return true
        }
        return false
    }, set: { isPresented in
        model.destination = nil
    }
),
```

No es bueno, propagar el valor `destination` a la vista, y más cuando estamos realizando la misma operación en ambas navegaciones. Crearemos un nuevo método en el modelo y actualizaremos la vista.

```swift
func dismissButtonTapped() {
    destination = nil
}
```

Por último, necesitamos el valor de `presenting` para mostrar la alerta. Aquí, una idea sería crear una extensión dentro de `Destination`.

```swift
presenting: model.destination?.exerciseToRemove

extension ExercisesModel.Destination {
	var exerciseToRemove: Exercise? {
		guard case let .alert(exercise) = self else {
			return nil
		}
		return exercise
	}
}
```

Y ya tendríamos la implementación hecha para los dialogs. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.dialog.2)

Finalmente, con este refactor, los tests se nos han quedado obsoletos y hay que actualizados. 

```swift
@Test func givenExercisesWhenOnAlertSwipeCalledThenAlertShownAndExercisesDeleted() async throws {
    let exercise = Exercise.fake(.random)

    let model = ExercisesModel(
        exercises: [
            exercise
        ]
    )

    model.alertButtonTapped(exercise)
    model.confirmAlertButtonTapped()

    #expect(model.exercises.isEmpty)
}

@Test func givenExercisesWhenOnDialogSwipeCalledThenDialogShownAndExerciseChangedToRunning() async throws {
    let exercise = Exercise.fake(type: .cycling)

    let model = ExercisesModel(
        exercises: [
            exercise
        ]
    )

    model.dialogButtonTapped(exercise)
    model.confirmDialogButtonTapped(type: .running)

    #expect(model.exercises[0].type == .running)
}
```

[Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.dialog.3)