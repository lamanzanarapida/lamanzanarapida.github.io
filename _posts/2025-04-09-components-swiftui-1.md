---
title: "Componentes en SwiftUI."
excerpt_separator: "<!--more-->"
categories:
  - SwiftUI
tags:
  - Swift
  - SwiftUI
  - iOS
---

SwiftUI nos ofrece la posibilidad de utilizar varios estilos, pero en general, si hacemos una app querremos dar estilo creando nuestros componentes personalizados.

Empecemos por un botón. SwiftUI nos ofrece lo siguiente.

```swift
Button("Un botón") {}
.buttonStyle(.bordered)

Button("Un botón") {}
.buttonStyle(.borderedProminent)

Button("Un botón") {}
.buttonStyle(.borderless)

Button("Un botón") {}
.buttonStyle(.plain)
```

En la mayoría de los casos, por no decir todos, con estos estilos ofrecidos por Apple, no tendremos suficiente para crear nuestros botones para nuesta maravilla app. Con lo cual, necesitaríamos hacer algo así.

```swift
Button {
} label: {
    HStack {
        Spacer()
        Text("Un botón")
        Spacer()
    }
    .padding()
}
.bold()
.foregroundColor(.black)
.background(Capsule().stroke(.black))
```

Esta opción soluciona nuestro problema de crear un botón personalizado, pero no escalaría muy bien. Podríamos hacer lo siguiente.

```swift
struct CustomButton<Label: View>: View {
    private let action: () -> Void
    private let label: Label

init(
    action: @escaping () -> Void,
    @ViewBuilder label: () -> Label
) {
    self.action = action
    self.label = label()
}

    var body: some View {
        Button { 
            action()
        } label: {
            HStack {
            Spacer()
            label
            Spacer()
        }
        .padding()
    }
    .bold()
    .foregroundColor(.black)
    .background(Capsule().stroke(.black))
    }
}
```

Aquí solucionamos el problema de escalado, al menos en parte, pero seguimos dependiendo de crear nuevas vistas para cada uno de los botones que necesitamos. Además, estaríamos limitado a uno de los constructores.

```swift
@preconcurrency
init(
    action: @escaping @MainActor () -> Void,
    @ViewBuilder label: () -> Label
)
```

¿Pero qué ocurre con el resto de los constructores? 

Pues existe una forma muy sencilla que, para algunos objetos de SwiftUI podemos construir diferentes estilos de una forma muy limpia.

> Nota: Podemos construir estilos personalizados de una forma sencilla con Toggle, Date­Picker, Gauge, Progress­View, Label, Labeled­Content, Disclosure­Group, Control­Group, Group­Box, Form y Button.

```swift
struct CustomButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        HStack {
            Spacer()
            configuration.label
            Spacer()
        }
        .padding()
        .bold()
        .padding()
        .foregroundColor(.black)
        .background {
            Capsule().stroke(.black)
        }
    }
}
```

Finalmente, podemos hacer uso de este estilo de la siguiente manera.

```swift
Button("Un botón") {}
    .buttonStyle(CustomButtonStyle())
```

Como se puede observar, de esta forma, podemos tener nuestros estilos de la aplicación perfectamente definidos sin construir nuevas vistas, como por ejemplo, `PrimaryButtonView`, `SecondaryButtonView`. Y podemos crearnos unos estilos como `.primary` o `.secondary`. ¿Pero cómo podríamos hacer eso? De la siguiente forma.

```swift
extension ButtonStyle where Self == CustomButtonStyle {
    static var custom: CustomButtonStyle { CustomButtonStyle() }
}

Button("Un botón") {}
    .buttonStyle(.custom)
```

También se pueden construir estilos personalizados en vistas personalizadas, pero eso será en otra ocasión.