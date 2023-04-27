# Traits en Cairo

Los traits especifican plantillas de funcionalidad que pueden ser implementadas. La especificación de la plantilla incluye un conjunto de firmas de funciones que contienen anotaciones de tipos para los parámetros y el valor de retorno. Esto establece un estándar para implementar la funcionalidad específica.

## Definiendo un Trait

Para definir un trait, se utiliza la palabra clave `trait` seguida del nombre del trait en `PascalCase` y luego las firmas de funciones dentro de un par de llaves.

Por ejemplo, supongamos que tenemos múltiples estructuras que representan formas. Queremos que nuestra aplicación pueda realizar operaciones de geometría en estas formas, por lo que definimos un trait `ShapeGeometry` que contiene una plantilla para implementar operaciones de geometría en una forma de esta manera:

```rust
trait ShapeGeometry {
    fn boundary( self: Rectangle ) -> u64;
    fn area( self: Rectangle ) -> u64;
}
```

Aquí nuestro trait `ShapeGeometry` declara las firmas de dos métodos `boundary` y `area`. Cuando se implementen, ambas funciones deben devolver un `u64` y aceptar parámetros tal como se especifica en el trait.

## Implementando un Trait

Un trait puede ser implementado usando la palabra clave `impl` seguida del nombre de la implementación y la palabra `of`, seguida del nombre del trait que está siendo implementado. Aquí hay un ejemplo de cómo implementar el trait `ShapeGeometry`.

```rust
impl RectangleGeometry of ShapeGeometry {
	fn boundary( self: Rectangle ) -> u64 {
        2_u64 * (self.height + self.width)
    }
	fn area( self: Rectangle ) -> u64 {
		self.height * self.width
	}
}
```

En el código anterior, `RectangleGeometry` implementa el trait `ShapeGeometry` definiendo lo que deben hacer los métodos `boundary` y `area`. Note que los tipos de los parámetros de las funciones y los valores de retorno son idénticos a los especificados en el trait.

## Parámetro `self`

En el ejemplo anterior, `self` es un parámetro especial. Cuando se usa un parámetro con el nombre `self`, las funciones implementadas también [se adjuntan a las instancias del tipo como métodos](ch04-03-method-syntax.md#defining-methods). Aquí hay una ilustración,

Cuando se implementa el trait `ShapeGeometry`, la función `area` del trait `ShapeGeometry` se puede llamar de dos maneras:

```rust
let rect = Rectangle { ... }; // Rectangle instantiation

// First way, as a method on the struct instance
let area1 = rect.area();
// Second way, from the implementation
let area2 = RectangleGeometry::area(rect);
// `area1` has same value as `area2`
area1.print();
area2.print();
```

## Traits con tipos genéricos

Por lo general, queremos escribir un trait cuando queremos que múltiples tipos implementen una funcionalidad de una manera estándar. Sin embargo, en el ejemplo anterior, las firmas son estáticas y no se pueden usar para múltiples tipos. Para hacer esto, usamos tipos genéricos al definir traits.

En el siguiente ejemplo, usamos el tipo genérico `T` y nuestras firmas de métodos pueden usar este alias que se puede proporcionar durante la implementación.

```rust
use debug::PrintTrait;

// Here T is an alias type which will be provided buring implementation
trait ShapeGeometry<T> {
    fn boundary( self: T ) -> u64;
    fn area( self: T ) -> u64;
}

// Implementation RectangleGeometry passes in <Rectangle>
// to implement the trait for that type
impl RectangleGeometry of ShapeGeometry::<Rectangle> {
    fn boundary( self: Rectangle ) -> u64 {
        2_u64 * (self.height + self.width)
    }
    fn area( self: Rectangle ) -> u64 {
        self.height * self.width
    }
}

// We might have another struct Circle
// which can use the same trait spec
impl CircleGeometry of ShapeGeometry::<Circle> {
    fn boundary( self: Circle ) -> u64 {
        (2_u64 * 314_u64 * self.radius) / 100_u64
    }
    fn area( self: Circle ) -> u64 {
       (314_u64 * self.radius * self.radius) / 100_u64
    }
}

fn main() {
    let rect = Rectangle { height: 5_u128, width: 7_u128 };
    rect.area().print(); // 35
    rect.boundary().print(); // 24

    let circ = Circle { radius: 5_u128 };
    circ.area().print(); // 78
    circ.boundary().print(); // 31
}
```

## Administrando y usando implementaciones de traits externos

Para usar los métodos de traits, es necesario asegurarse de que los traits/implementaciones correctos estén importados. En el código anterior, importamos `PrintTrait` desde `debug` con `use debug::PrintTrait;` para usar el método `print()`.

En algunos casos, puede ser necesario importar no solo el trait, sino también la implementación si están declarados en módulos separados. Si `CircleGeometry` estuviera en un módulo/archivo separado `circle`, entonces para usar `boundary` en `circ: Circle`, necesitaríamos importar `CircleGeometry` además de `ShapeGeometry`.

Si el código estuviera organizado en módulos de esta manera,

```rust
use debug::PrintTrait;

// struct Circle { ... } and struct Rectangle { ... }

mod geometry {
    use super::Rectangle;
    trait ShapeGeometry<T> {
        // ...
    }

    impl RectangleGeometry of ShapeGeometry::<Rectangle> {
        // ...
    }
}

// Could be in a different file
mod circle {
    use super::geometry::ShapeGeometry;
    use super::Circle;
    impl CircleGeometry of ShapeGeometry::<Circle> {
        // ...
    }
}

fn main() {
    let rect = Rectangle { height: 5_u64, width: 7_u64 };
    let circ = Circle { radius: 5_u64 };
    // Fails with this error
    // Method `area` not found on... Did you import the correct trait and impl?
    rect.area().print();
    circ.area().print();
}
```

Para hacer que funcione, además de,

```rust
use geometry::ShapeGeometry;
```

para hacerlo funcionar, también tendrías que usar `CircleGeometry`,

```rust
use circle::CircleGeometry
```
