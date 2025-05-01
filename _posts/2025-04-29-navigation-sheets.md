---
title: "Navegación con SwiftUI. Sheets."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

## Refactorización

Antes de empezar esta parte, vamos a refactorizar dos partes interesantes. La primera trata la ordenación de los ejercicios. ¡No hay!. Así que, haremos una ordenación sencilla con la fecha de inicio de un ejercicio.

```swift
ForEach(model.exercises.sorted(by: { $0.starts > $1.starts }))
```

> Nota: Utilizar una regla de negocio directamente en la vista, imposibilita la creación de un test sencillo. Esto lo resolveremos más adelante. Pero hay que tener en cuenta que, cuanta más lógica haya en una vista, más problemático será su protección a futuros errores no deseados.

La otra parte, está en el dialog. Por un lado, tenemos código repetido. ¡Cuatro botones muy similares! Y por otro, uno de esos botones es redundante. El sistema nos da la posibilidad de actualizar el tipo de ejercicio, !al mismo ejericicio!. Vamos a resolver esta parte, utilizando la misma estrategia que la refactorización anterior, pero utilizando un filter.

Actualiza la clausura donde tenemos los 4 botones, por el siguiente código.

```swift
ForEach(
    Exercise.Mode.allCases,
    id: \.self
) { type in
    Button(type.rawValue.capitalized) {
        withAnimation { model.confirmDialogButtonTapped(type: type) }
    }
}
```

Ahora necesitamos conocer el tipo de ejercicio para filtrar por todos los ejercicios, salvo el que tenga en ese momento el ejercicio a actualizar. Utilizaremos una propiedad de los dialogs, presenting. 

Al igual que la propiedad exerciseToRemove, necesitaremos crear una nueva propiedad exerciseToUpdate.

```swift
extension ExercisesModel.Destination {
    var exerciseToRemove: Exercise? {
        guard case let .alert(exercise) = self else {
            return nil
        }
        return exercise
    }
    var exerciseToUpdate: Exercise? {
        guard case let .dialog(exercise) = self else {
            return nil
        }
        return exercise
    }
}
```

El dialog resultante quedará así. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.sheet.1){:target="_blank"}

```swift
.confirmationDialog(
    "Change exercise type",
    isPresented: Binding(
        get: {
            if case .dialog = model.destination {
            return true
            }
            return false
        }, set: { isPresented in
            model.dismissButtonTapped()
        }
    ),
    titleVisibility: .visible,
    presenting: model.destination?.exerciseToUpdate
) { exercise in
    ForEach(
        Exercise.Mode.allCases.filter { $0 != exercise.type },
        id: \.self
    ) { type in
        Button(type.rawValue.capitalized) {
            withAnimation { model.confirmDialogButtonTapped(type: type) }
        }
    }
} message: { _ in
    Text("Choose a new option for the exercise type.")
}
```

## Sheet

Seguimos con la construcción de nuestra app de ejercicios. Ahora vamos a crear la funcionalidad de insertar un ejercicio nuevo en nuestro listado. Para ello, usaremos lo que se llama un sheet (que viene a ser una modal, que puede ocupar toda la pantalla o parte de ella).

Existen tres tipos (los veremos), son el sheet, el fullScreenCover y el popover, éste último para iPad. Vamos a por el sheet.

Empezaremos por la versión más sencilla del sheet.

```swift
nonisolated
func sheet<Content>(
    isPresented: Binding<Bool>,
    onDismiss: (() -> Void)? = nil,
    @ViewBuilder content: @escaping () -> Content
) -> some View where Content : View
```

Primero lo que haremos será crear un botón con la imagen + en la toolbar o navegación superior derecha. 

```swift
.navigationTitle("Exercises")
.toolbar {
    ToolbarItem(placement: .confirmationAction) {
        Button {
            model.addExerciseButtonTapped()
        } label: {
            Image(systemName: "plus")
        }
    }
}
```

> Nota: Si quieres ver el título en la preview, tendrás que utilizar un NavigationStack.

Luego crearemos un pequeño método en el modelo para mostrar un nuevo sheet. Aquí tenemos que pensar en la estrategia que hemos usado para mostrar una alerta o un dialog. Para mostrar un sheet, utilizaremos la misma estrategia. Crearemos un nuevo case add.

```swift
func addExerciseButtonTapped() {
    destination = .add
}
```

Finalmente, implementaremos el sheet en la vista. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.sheet.2){:target="_blank"}

```swift
.sheet(
    isPresented: Binding(
        get: { model.destination == .add },
        set: { isPresented in
            if !isPresented {
                model.dismissButtonTapped()
            }
        }
    ),
    onDismiss: { model.dismissButtonTapped() },
    content: {
        Text("New sheet")
    }
)
```

> Nota: El compilador nos puede pedir en cualquier momento, que un tipo cumpla con determinados protocolos. Equatable, Hashable o Identifiable. Sin entrar en demasiado detalle en el significado de cada uno de ellos, por lo general, es bueno, entender por qué el compilador nos lo está pidiendo. En este caso, nos pide un Equatable para el Destination porque estamos usando el comparador ==. Así que le asignaremos el protocolo Equatable para ese tipo.

Ahora vamos a crear la pantalla de creación de un ejercicio. Para ello, utilizaremos un pequeño formulario con diferentes opciones. Aquí lo importante para remarcar es que el formulario tiene un ejercicio inicial, que llamaremos empty y, a partir, de ese estado vacío, le vamos a permitir al usuario modificar los valores que desee. Como siempre, todo tiene que funcionar en una preview.

Empezaremos por crear un nuevo fichero, ExerciseView.

```swift
import SwiftUI

@Observable
class ExerciseModel {
    var exercise: Exercise

    init(
        exercise: Exercise = .empty
    ) {
        self.exercise = exercise
    }

    var addExerciseButtonDisabled: Bool {
        exercise.title.isEmpty
        || exercise.distance == 0
    }
}

extension ExerciseModel: Equatable {
    static func == (lhs: ExerciseModel, rhs: ExerciseModel) -> Bool {
        lhs.exercise == rhs.exercise
    }
}

extension ExerciseModel: Identifiable {
    var id: Exercise.ID {
        exercise.id
    }
}

struct ExerciseView: View {
    @Bindable var model: ExerciseModel

    var body: some View {
        VStack {
            Form {
                Section("Write a title") {
                    TextField("The title", text: $model.exercise.title)
                }
                .textCase(nil)
                Section("Select the type") {
                    Picker("Selected Type", selection: $model.exercise.type) {
                        Text("Cycling").tag(Exercise.Mode.cycling)
                        Text("Swimming").tag(Exercise.Mode.swimming)
                        Text("Running").tag(Exercise.Mode.running)
                        Text("Walking").tag(Exercise.Mode.walking)
                    }
                    .pickerStyle(.navigationLink)
                }
                .textCase(nil)
                Section("Set the distance") {
                    TextField("Distance", value: $model.exercise.distance, format: .number)
                }
                .textCase(nil)
                Section("Set starting and finishing dates") {
                    DatePicker("Started", selection: $model.exercise.starts)
                    if model.exercise.isFinished {
                        DatePicker(
                            "Finished",
                            selection: Binding(
                                get: { model.exercise.finished ?? Date() },
                                set: { model.exercise.finished = $0 }
                            )
                        )
                        Button("Not finish the exercise?") {
                            model.exercise.finished = nil
                        }
                    } else {
                        Button("Finish the exercise?") {
                            model.exercise.finished = Date()
                        }
                    }
                }
                .textCase(nil)
            }
        }
    }
}

#Preview {
    NavigationStack {
        ExerciseView(
            model: ExerciseModel(
                exercise: .empty
            )
        )
        .navigationTitle("Doing exercise!")
    }
}
```

Ahora el compilador nos recordará que el modelo no puede tener atributos constantes, así que las haremos variables usando la palabra reservada `var`.

Al igual que hemos hecho al principio, podemos refactorizar un poco más el picker.

```swift
Picker("Selected Type", selection: $model.exercise.type) {
    ForEach(Exercise.Mode.allCases, id: \.self) { type in
        Text(type.rawValue.capitalized).tag(type)
    }
}
```

Ya tenemos la pantalla, ahora tenemos que engancharla a la pantalla principal. Lo primero que tenemos que pensar es qué valor le pasar al sheet. Si en las alertas y en los dialogs, hemos pasado un Exercise, uno podría pensar que lo que tenemos que pasarle es un Exercise. A fin de cuentas, es el modelo que necesitamos mostrar. Pero esto no sería correcto. Estamos mostrando un modelo para mostrar una pantalla y al hacer sheet, mostramos un modelo. El valor correcto que tenemos que mostrar es un ExerciseModel.

Actualizaremos el Destination.

```swift
case add(ExerciseModel)
```

y también la función `addExerciseButtonTapped`.

```swift
destination = .add(ExerciseModel())
```

Por último tenemos que actualizar el `.sheet` de la vista, ahora con los valores del `case .add` y recuperar el modelo para pintarlo dentro del `sheet`.

```swift
.sheet(
    isPresented: Binding(
        get: {
            if case .add = model.destination {
                return true
            }
            return false
        },
        set: { isPresented in
            if !isPresented {
                model.dismissButtonTapped()
            }
        }
    ),
    onDismiss: { model.dismissButtonTapped() },
    content: {
        if case let .add(model) = model.destination {
            ExerciseView(model: model)
        }
    }
)
```

## Comunicación Parent - Child

Podríamos crear ahora un par de botones en la navegación dentro de ExerciseView, pero esto, sería contraproducente. Primero porque limitaríamos la vista a la acción de añadir, y luego porque tendríamos que añadir nueva lógica en forma de estados, si queremos editar, duplicar y añadir.

El truco es sacar toda esta lógica y manejarla en el Parent.

```swift
NavigationStack {
    ExerciseView(model: model)
        .navigationTitle("New exercise!")
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button("Cancel") {
                    self.model.dismissButtonTapped()
                }
            }
            ToolbarItem(placement: .primaryAction) {
                Button("Add") {
                }
                .disabled(model.addExerciseButtonDisabled)
            }
        }
}
```

> Nota: Aquí es importante el uso de `self.` para utilizar, o bien el model de ExercisesModel o bien el model de ExerciseModel.

Como estamos utilizando el destination para cualquier navegación de la vista, podemos reutilizar `dismissButtonTapped` cuando estamos ocultando la vista, bien sea porque hemos hecho tap al botón de cancelar o bien porque hemos usado el gesto hacia abajo para hacerla desaparecer.

Ahora nos queda guardar el ejercicio en el listado. Primero añadiremos la función en el botón de `Add`.

```swift
self.model.confirmAddButtonTapped(exercise: model.exercise)
```

Y luego la implementaremos en ExercisesModel. Como tenemos la información del listado, esta operación es muy sencilla. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.sheet.3)

```swift
func confirmAddButtonTapped(exercise: Exercise) {
    exercises.append(exercise)
    destination = nil
}
```

### Más refactor

Podemos crear una nueva property

```swift
var exerciseToAdd: ExerciseModel? {
    guard case let .add(model) = self else {
        return nil
    }
    return model
}
```

Esto nos da un poco de libertad, para escribir el Binding de una forma más ergonómica, pero no nos termina de escribir el sheet del todo bien.

Apple nos ofrece otro sheet, que se ajusta más a la forma que tenemos de construir las navegaciones.

```swift
nonisolated
func sheet<Item, Content>(
    item: Binding<Item?>,
    onDismiss: (() -> Void)? = nil,
    @ViewBuilder content: @escaping (Item) -> Content
) -> some View where Item : Identifiable, Content : View
```

Así pues, refactoricemos utilizando este nuevo `.sheet`.

```swift
.sheet(
    item: Binding(
        get: { model.destination?.exerciseToAdd },
        set: { isPresented in
            if isPresented == nil {
                model.dismissButtonTapped()
            }
        }
    ),
    onDismiss: { model.dismissButtonTapped() },
    content: { model in
        NavigationStack {
            ExerciseView(model: model)
                .navigationTitle("New exercise!")
                .toolbar {
                    ToolbarItem(placement: .cancellationAction) {
                        Button("Cancel") {
                            self.model.dismissButtonTapped()
                        }
                    }
                    ToolbarItem(placement: .primaryAction) {
                        Button("Add") {
                            self.model.confirmAddButtonTapped(exercise: model.exercise)
                        }
                        .disabled(model.addExerciseButtonDisabled)
                    }
                }
        }
    }
)
```

## Testing

Podemos crear un nuevo test, que simule todo el proceso de añadir un nuevo ejercicio en el listado. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.destination.1)

```swift
@Test func givenExercisesWhenAddButtonTappedNewExerciseAdded() async throws {
    let model = ExercisesModel()

    model.addExerciseButtonTapped()

    #expect(model.destination?.exerciseToAdd!.addExerciseButtonDisabled == true)

    model.destination?.exerciseToAdd!.exercise.title = "New exercise"
    model.destination?.exerciseToAdd!.exercise.distance = 1.5

    #expect(model.destination?.exerciseToAdd!.addExerciseButtonDisabled == false)

    model.confirmAddButtonTapped(exercise: model.destination!.exerciseToAdd!.exercise)

    #expect(model.exercises.count == 1)
}
```