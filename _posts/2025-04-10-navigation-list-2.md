---
title: "Navegación con SwiftUI. List."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
  - Navegacion
---

Seguimos con esta segunda parte y antes de empezar con las alertas, vamos a crear un pequeño listado de items, que nos servirá como base para implementar el resto de navegaciones de esta serie.

## Creando el modelo del ejercicio

Iniciemos con el modelo, un ejercicio. Es un modelo sencillo. Vamos a tener cuatro tipos de ejercicio, con una fecha de inicio y quizás una fecha final.

Crearemos un nuevo fichero Exercise.

```swift
struct Exercise: Identifiable {
	let id: UUID
	let distance: Double
	let title: String
	let starts: Date
	let finished: Date?
	let type: Mode
	
	var isFinished: Bool {
		finished != nil
	}
	
	enum Mode: String, CaseIterable {
		case cycling = "Cycling", running = "Running", swimming = "Swimming", walking = "Walking"
	}
}

extension Exercise.Mode {
	var systemImage: String {
		switch self {
			case .cycling:
				return "figure.outdoor.cycle.circle"
			case .running:
				return "figure.run.circle"
			case .swimming:
				return "figure.open.water.swim.circle"
			case .walking:
				return "figure.walk.circle"
		}
	}
	
	var color: Color {
		switch self {
			case .cycling:
				return .yellow
			case .running:
				return .green
			case .swimming:
				return .blue
			case .walking:
				return .orange
		}
	}
}

extension Exercise {
	var startsFormatted: String {
		starts.formatted(date: .numeric, time: .omitted)
	}
	
	var duration: String {
		if let finished {
			let diff = Int(finished.timeIntervalSince1970 - starts.timeIntervalSince1970)
			return "\(diff / 60) min"
		} else {
			return "In progress"
		}
	}
	
	var distanceFormatted: String {
		"\(String(format: "%.2f", distance)) KM"
	}
	
	var color: Color {
		type.color
	}
}
```

> Nota: En relación con la definición de una arquitectura limpia, un modelo no debería depender de un framework. En nuestro caso, estamos dependendiendo de SwiftUI porque el modelo guarda el color. Aunque no es un problema crítico, si que es importante que seamos capaces de reconocer los patrones para luego poderlos solucionar. 

## Creando la ExerciseRowView en SwiftUI

Ahora, crearemos un nuevo fichero de SwiftUI, que va a representar cada row en un listado. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.list.1)

> Nota: Si vienes de UIKit, habrás usado la notación Cell en los nombres de las celdas de las tablas y colecciones. Sin embargo, en SwiftUI se utiliza mucho la notación Row.

La row va a necesitar un ejercicio. Para empezar, podemos poner un texto.

```swift
struct ExerciseRowView: View {
  let exercise: Exercise

  var body: some View {
    Text("ExerciseRowView")
  }
}
```

Ahora podemos crear una row sencillita, de forma que se muestre la información de una forma adecuada.

```swift
var body: some View {
  HStack(alignment: .top) {
    Image(systemName: exercise.type.systemImage)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .frame(width: 50, height: 50)
      .foregroundStyle(exercise.color)
    Spacer()
    VStack(alignment: .leading) {
      HStack {
        Text(exercise.title)
          .lineLimit(1)
        Spacer()
        Text(exercise.startsFormatted)
          .foregroundStyle(.gray)
      }
      HStack(alignment: .lastTextBaseline) {
        Text(exercise.distanceFormatted)
          .font(.system(size: 24))
          .foregroundStyle(exercise.color)
        Spacer()
        Text(exercise.duration)
          .foregroundStyle(.gray)
      }
    }
  }
}
```

Aunque no lo parezca, tener una buena preview es super importante para verificar el trabajo realizado. Aunque luego se hagan tests y snapshot testing, por un lado, desarrollar viendo los cambios y no depender de ejecutar el simulador es una ventaja que puede marcar la diferencia.

Crearemos una extension en Exercise para construir fácilmente objetos fake para nuestras previews.


```swift
extension Exercise {
	static func fake(type: Exercise.Mode) -> Self {
		Self(
			id: UUID(),
			distance: 15.45,
			title: "First exercise of my life",
			starts: .now,
			finished: .now.addingTimeInterval(3600),
			type: type
		)
	}
}
```

Entonces, nuestra preview, puede quedar de la siguiente manera.

```swift
#Preview {
	NavigationStack {
		List {
			ExerciseRowView(exercise: .fake(type: .cycling))
			ExerciseRowView(exercise: .fake(type: .running))
			ExerciseRowView(exercise: .fake(type: .swimming))
			ExerciseRowView(exercise: .fake(type: .walking))
		}
		.listRowSpacing(8)
	}
}
```


Podemos refactorizar la preview y hacerla un poco más ergonómica. Exercise.Mode es CaseIterable, esto significa que podemos hacer uso de allCases de la siguiente forma. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.list.2)

```swift
#Preview {
  NavigationStack {
    List {
      ForEach(
        Exercise.Mode.allCases.map(Exercise.fake(type:))
      ) { item in
        ExerciseRowView(exercise: item)
      }
    }
  }
}
```

## Creando el listado de ejercicios en SwiftUI

Al igual que hemos hecho en el punto anterior, crearemos otro fichero y lo llamaremos ExercisesView. En este caso, vamos a necesitar un modelo para controlar la lógica del listado de ejercicios. Crearemos otro fichero y lo llamaremos ExercisesModel.

```swift
import Observation

@Observable
class ExercisesModel {
	let exercises: [Exercise]
	
	init(
		exercises: [Exercise] = []
	) {
		self.exercises = exercises
	}
}
```

Una vez tenemos ya el modelo de la vista, crearemos la vista en si.

```swift
import SwiftUI

struct ExercisesView: View {
	let model: ExercisesModel
	
	var body: some View {
		List {
			ForEach(model.exercises) { exercise in
				ExerciseRowView(exercise: exercise)
			}
		}
		.listRowSpacing(8)
	}
}
```

Finalmente, crearemos unas previews, pero esta vez que sean un poco más interesantes para esta pantalla. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.list.3)

Primero, añadiremos otra extension en Exercise.

```swift
extension Exercise {
	enum FakeStrategy {
		case random
	}
	static func fake(_ strategy: FakeStrategy) -> Self {
		let id = UUID()
		let randomDate: Date = Date.randomBetween(start: "2025-04-01", end: "2025-05-01")
		return Self(
			id: id,
			distance: Double.random(in: 0..<50),
			title: "This is exercise \(id.uuidString)",
			starts: randomDate,
			finished: randomDate.addingTimeInterval(TimeInterval.random(in: 0..<3600)),
			type: Mode.allCases.randomElement()!
		)
	}
}
```

Y añade al final de Exercise la siguiente extensión. No te preocupes si no la entiendes. Al final, es una forma de generar una fecha comprendida entre dos que le pasemos, y solamente la vamos a utilizar para crear datos fake, para nuestras previews.

```swift
fileprivate extension Date {
	static func randomBetween(start: String, end: String, format: String = "yyyy-MM-dd") -> Date {
		let date1 = Date.parse(start, format: format)
		let date2 = Date.parse(end, format: format)
		return Date.randomBetween(start: date1, end: date2)
	}

	private static func randomBetween(start: Date, end: Date) -> Date {
		var date1 = start
		var date2 = end
		if date2 < date1 {
			let temp = date1
			date1 = date2
			date2 = temp
		}
		let span = TimeInterval.random(in: date1.timeIntervalSinceNow...date2.timeIntervalSinceNow)
		return Date(timeIntervalSinceNow: span)
	}

	private func dateString(_ format: String = "yyyy-MM-dd") -> String {
		let dateFormatter = DateFormatter()
		dateFormatter.dateFormat = format
		return dateFormatter.string(from: self)
	}

	private static func parse(_ string: String, format: String = "yyyy-MM-dd") -> Date {
		let dateFormatter = DateFormatter()
		dateFormatter.timeZone = NSTimeZone.default
		dateFormatter.dateFormat = format

		let date = dateFormatter.date(from: string)!
		return date
	}
}
```

Ahora ya podemos crear nuestra preview en ExercisesView. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.list.4)

```swift
#Preview {
	ExercisesView(
		model: ExercisesModel(
			exercises: (0..<50).map { _ in 
				Exercise.fake(.random)
			}
		)
	)
}
```

## Borrando un elemento del listado de ejercicios

Nuestro primer caso de uso va a ser poder eliminar un elemento de una lista. Para ello, añadiremos el siguiente código, justo después del ForEach.

```swift
ForEach(model.exercises) { exercise in
	ExerciseRowView(exercise: exercise)
}
.onDelete { indexSet in
// _????
}
```

Ahora lo que tenemos es que asociar el evento de onDelete, con el modelo de la vista. 

```swift
import Foundation
import Observation

@Observable
class ExercisesModel {
	var exercises: [Exercise]

	init(
		exercises: [Exercise] = []
	) {
		self.exercises = exercises
	}

	func onDeleteExercises(_ indexSet: IndexSet) {
		for index in indexSet {
			exercises.remove(at: index)
		}
	}
}
```

Necesitamos hacer que el listado sea mutable. Cambiaremos de let a var. IndexSet es un conjunto de indices. Entonces para eliminar los elementos usaremos un for sencillo.

Por último, actualizaremos la vista añadiendo la llamada onDeleteExercises en la vista.

```swift
ForEach(model.exercises) { exercise in
	ExerciseRowView(exercise: exercise)
}
.onDelete { indexSet in
	model.onDeleteExercises(indexSet)
}
```

Ya tenemos la primera funcionalidad. Ahora vamos a crear un pequeño test para validarla. Crearemos un nuevo archivo de test y lo llamaremos ExercisesModelTests.

```swift
import Testing
@testable import NavigationDoExercise

struct NavigationDoExercisesListTests {
	@Test func givenExercisesWhenOnDeleteExercisesCalledThenExercisesDeleted() async throws {
		let model = ExercisesModel(
			exercises: [
				.fake(.random)
			]
		)

		model.onDeleteExercises([0])

		#expect(model.exercises.isEmpty)
	}
}
```

Ejecutamos los tests, y los dos que tenemos funcionan. [Ver en Github](https://github.com/lamanzanarapida/navigation-do-exercise/tree/navigation.list.5)

## Alerts y Dialogs, para el próximo capítulo.

Lamentablemente, utilizar este patrón es una mala práctica en el desarrollo móvil. Necesitamos un paso más, para avisar de que algo tan importante como eliminar un ejercicio se realice. Y para ello, utilizaremos un alert. Pero eso, será en otra ocasión.