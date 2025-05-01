---
title: "Navegación con SwiftUI. Navigation Destination."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

Finalizamos esta serie, con el ultimo feature. Por hacer push hacia otra pantalla, de forma que podamos interactuar. Típicamente, se utiliza una navegación en profundidad, push o usando un link para editar un elemento si se hace tap en uno de los ejercicios.

## Navigation Destination

Empezamos por una de las formas más sencillas, `.navigationDestination`. Esta forma es similar al `.sheet.`. Vamos a ello.

Crearemos un nuevo case en nuestro `Destination`.

```swift
case edit(ExerciseModel)
```

Podemos aprovechar y crear una nueva property para nuestro case `exerciseToEdit`.

```swift
var exerciseToEdit: ExerciseModel? {
    guard case let .edit(model) = self else {
        return nil
    }
    return model
}
```

Si queremos hacer tap a cada unos de los ejercicios, tendremos que crear un botón. Envolveremos el `ExerciseRowView` con un botón.

```swift
Button {
    model.editButtonTapped(exercise)
} label: {
    ExerciseRowView(exercise: exercise)
        .contentShape(Rectangle())
        .
        .
        .
}
.buttonStyle(.plain)
```

Necesitaremos una nueva función `editButtonTapped` en el modelo

```swift
func editButtonTapped(_ exercise: Exercise) {
    destination = .edit(ExerciseModel(exercise: exercise))
}
```

Y ahora, de una forma similar a lo que haríamos con un `.sheet`, implementaremos un `navigationDestination`.

```swift
.navigationDestination(
    item: Binding(
        get: { model.destination?.exerciseToEdit },
        set: { isPresented in
            if isPresented == nil {
                model.dismissButtonTapped()
            }
        }
    )
) { model in
    NavigationStack {
        ExerciseView(model: model)
            .navigationTitle("Edit exercise")
    }
}
```

> Nota: la preview puede que no te funcione. Por lo visto tiene un bug cuando se ponen muchos modificadores anidados. Después de probar poniendo la `.toolbar` al final me ha funcionado.

Ya lo tendríamos. Sin embargo, al regresar al listado, si hemos modificado algun ejercicio, dichos cambios no se verán reflejados. Vamos a solucionarlo.

Lo primero que tenemos que hacer es ocultar el botón de atrás, en su lugar pondremos un botón cancelar. Y, implementaremos, un botón editar.

```swift
.navigationBarBackButtonHidden()
.toolbar {
    ToolbarItem(placement: .cancellationAction) {
        Button("Cancel") {
            self.model.dismissButtonTapped()
        }
    }
    ToolbarItem(placement: .primaryAction) {
        Button("Edit") {
            self.model.confirmEditButtonTapped(exercise: model.exercise)
        }   
        .disabled(model.addExerciseButtonDisabled)
    }
}
```

Refactorizaremos el nombre de `addExerciseButtonDisabled` por `exerciseButtonDisabled`, ya que ahora la pantalla representa tanto para añadir un nuevo ejercicio como editarlo.

Ahora nos queda, implementar la función `confirmEditButtonTapped`. En este caso, tenemos que hacer una búsqueda para editar el ejercicio.

```swift
func confirmEditButtonTapped(exercise: Exercise) {
    guard
        let index = exercises.firstIndex(where: { $0.id == exercise.id })
    else { return }
    exercises[index] = exercise
    destination = nil
}
```

Por último, podemos crear un test, que valide el trabajo realizado.

```swift
@Test func givenExercisesWhenEditButtonTappedExerciseIsEdited() async throws {
    let exercise = Exercise.fake(type: .cycling)

    let model = ExercisesModel(
        exercises: [
            exercise
        ]
    )

    model.editButtonTapped(exercise)
    model.destination!.exerciseToEdit!.exercise.title = "Edited exercise"
    model.confirmEditButtonTapped(exercise: model.destination!.exerciseToEdit!.exercise)

    #expect(model.exercises[0].title == "Edited exercise")
}
```

## Links

Los links han sido deprecados en las últimas versiones, así que lo mejor es no implementar nada aquí. Aún así, es una alternativa si aún tenemos que soportar versiones anteriores.

La forma adecuada de manejar multiples destinos, no es con `.navigationDestination` sino con `NavigationStack`, pero para este ejemplo, no es el adecuado. En un futuro, miraremos de implementar un pequeño proceso de onboarding con diferentes pantallas y veremos el potencial que tiene construir un stack dinámico donde la navegación en profundidad puede ser a diferentes pantallas en diferentes condiciones.