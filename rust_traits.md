# Polymorphism in Rust: using enums and traits. 

Polymorphism is the technical term for multiple different types/objects implementing the same interface. A common example is shapes. Each shape can be considered its own type, however will likely implement the same functions as other shapes, such as calculating the area.

When I started with Rust Traits seemed like the most natural way of implementing polymorphism. Rust is focused on being a performant, low level language and as such requires knowing how much space to allocate for the value of a type. When using traits this can cause some annoyances that enums do not suffer from, meaning I now tend to favour using enums for polymorphic behaviour.

## How to implement the two forms of polymorphism
For those unfamiliar with Rust a trait is defining an interface. It is a collection of functions. If another type implements a trait, it must implement all the functions defined by the trait. For example,

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Rectangle {
    height: f64,
    width: f64,
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
}

struct Circle {
    radius: f64,
}

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius.powi(2)
    }
}

struct RightAngleTriangle {
    base: f64,
    height: f64,
}

impl Shape for RightAngleTriangle {
    fn area(&self) -> f64 {
        0.5 * self.base * self.height
    }
}
```

Enums in Rust are different from enums in languages you may be familiar with. They are much more similar to algebraic data types from functional programming. One defines an enum as you would in other languages, the difference being that the variants themselves can contain data. (Perhaps word this bit better). The enum equivalent to the above example would be,

```rust
enum Shape {
    Rectangle { width: f64, height: f64 },
    Circle { radius: f64 },
    RightAngleTriangle { base: f64, height: f64 },
}

impl Shape {
	fn area(&self) -> f64 {
    	match self {
            Shape::Rectangle{width, height} => width * height,
            Shape::Circle{radius} => std::f64::consts::PI * radius.powi(2),
            Shape::RightAngleTriangle{base, height} => 0.5 * base * height,
        }
    }
}
```

## Using the two forms of polymorphism
There are a couple of ways of using traits, generics and trait objects. The difference between these is static vs dynamic dispatch. Generics uses static dispatch, the compiler will generate a unique function for each of the types that implement the trait. 

```rust
fn print_area<S: Shape>(shape: S) {
    println!("{}", shape.area());
}

fn print_area(shape: Box<dyn Shape>) {
    println!("{}", shape.area());
}
```

This however has the problem that you can only use the types that the function is defined for, making something like a vector that contains elements that implement the trait impossible. Trait objects use dynamic dispatch, making this vector possible.

```rust
// This only works if every element in the vector is the same shape. Not the behaviour we want!
fn sum_areas<S: Shape>(shapes: Vec<S>) -> f64 {}

fn sum_areas(shapes: Vec<Box<dyn Shape>>) -> f64 {
    shapes.iter().fold(0., |acc, shape| {
        acc + shape.area()
    })
}
```

However, with dynamic dispatch the size of the types cannot be known. This means that the types must be put behind a reference and the trait itself must conform to the object safety constraints. For example, say we wanted to clone the vector of shapes. 

```rust
fn sum_areas(shapes: Vec<Box<dyn Shape>>) -> f64 {
    let cloned_shapes = shapes.clone();
    cloned_shapes.iter().fold(0., |acc, shape| {
        acc + shape.area()
    })
}
```

To be able to use clone we need to guarantee that our implementations derive Clone, this can be done using a subtrait. But if we try to make a trait that has Clone as a subtrait we can no longer use this trait as a trait object because of object safety.

```rust
// This violates object safety constraits and will not work.
trait Shape: Clone {
    ...
}
```

To be clear there are workarounds to be able to clone trait objects. But they are a bit of a pain and should be avoided if possible.
 
Enums can do everything a trait object would but the size of the enum is known and uses static dispatch. Making it simple to do something like a vector of the types, without having to worry about putting the types behind a reference and worrying about object safety constraints. Consider the previous example, one can simply derive clone for the enum and then implement the function, without having to worry about references or object safety!

```rust
#[derive(Clone)]
enum Shape {
    ...
}

fn sum_areas(shapes: Vec<Shape>) -> f64 {
    let cloned_shapes = shapes.clone();
    cloned_shapes.iter().fold(0., |acc, shape| {
        acc + shape.area()
    })
}
```

## Discuss whether to use enums or traits
Enums get all of the benefits of static dispatch without having to deal with object safety issues. This is the main reason I now use enums for polymorphic behaviour for most of my use cases.

There are still a couple of times that I use traits. The first of these is if I want users of the library to be able to add their own types to a trait. This is the main reason for using traits. The second is that if the interface I am defining is not determined by the types the will implement it. This is because, I will not be using these as trait objects and the ability to define it for any type is needed. For example, even if only used internally one would not implement Clone as an enum. Finding such a generic behaviour is rare however.

I have seen some arguments that using enums produces methods that are long and hard to read. I disagree with this. If the methods start to get too long one can extract the logic into separate functions, even grouping these functions into a module if there a large number of methods. In fact, I think this makes the code easier to read. The methods can then effectively be used as a table of contents allowing viewing the logic by jumping to the functions, rather than have to search through the code for the individual types.

One gripe I have is that the variants of enums are not considered types in their own right, meaning one cannot create functions that only work with an individual variant from the enum. This can be solved by creating the type and simply wrapping it in the enum, but this is a bit annoying. Others have had a similar problem and have found the enum dispatch removes this annoyance (https://users.rust-lang.org/t/trait-or-enum-for-sum-types/23310/6).

## Conclusion

In the majority of cases I need polymorphism I now use enums. I tend to only use traits if I want to allow users of the code to be able to add their own types or it is clear that the behaviour is not determined by the type. If I do use traits then I always favour using generics over trait objects. Trait objects should only be used when absolutely necessary due to the complications caused by object safety concerns.



























---------------------------------------------------


## Traits vs Enums
Traits in Rust are used to define a common interface for different types, known formally as polymorphism. It provides a guarantee that if some type implements a trait it will be able to use certain methods. Traits are not the only way to do this in Rust, in many cases Enums can also be used to fulfill the same purpose.

Let's create a trait Shape that has a function to calculate the area and a few shapes that implement this Trait. We now have 3 different shapes that all implement the same method area.

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Rectangle {
    width: f64,
    height: f64,
}

    fn area(&self) -> f64 {
        self.width * self.height
    }
}
struct Circle {
    radius: f64,
}

impl Shape for Circle {
        std::f64::consts::PI * self.radius.powi(2)
}

struct RightAngleTriangle {
    base: f64,
    height: f64,
}

impl Shape for RightAngleTriangle {
    fn area(&self) -> f64 {
        0.5 * self.base * self.height
    }
```
This behaviour can be replicated by creating an enum containing the different shapes and defining the method area for the Shape enum. Then define the behaviour for each shape by matching on the enum.

```rust
enum Shape {
    RightAngleTriangle(f64, f64),

impl Shape {
	fn area(&self) -> f64 {
    	match self {
        	Shape::Rectangle(width, height) => width * height,
            Shape::Circle(radius) => std::f64::consts::PI * radius.powi(2),
            Shape::RightAngleTriangle(base, height) => 0.5 * base * height,
        }
    }
}
```




When I first started with Rust I would always go for the Traits implementation. However now, in most cases, I favour the Enum implementation over the Trait implementation. This is because for most of my uses the Enum implementation can do everything the trait implemenation and more.


*Note: the size of an Enum is always that of it's largest variant. If one of the variants is very large you should consider putting it behind a reference.*

You can derive methods from traits just as you would with Structs and can do so without worrying about object safety at all. For example, if I want to ensure all of the types implement the Clone trait I simply add "#[derive(Clone)]" to the Enum. There is often no easy way to do this for Traits, especially when worrying about object safety.

I have seen some people complain that methods for Enums can get very long and hard to read. And that when using Traits all of the code is grouped much more nicely. However, I actually find using Enums easier to navigate that Traits and Structs. If a method starts to get long I will extract it out into a function. Then I effectively have a table of contents for all of the possible behaviour for the Enum. With the Traits and Structs implementation there would be much more manual searching to find all the different implementations.

In summary, the rule I tend to follow is to use Enums if possible. If I want a user to be able to add their own types then use Traits. I do not always stick to this rule, if I have some behaviour which has very little relation to the types which enact it I still tend to favour traits.
## Trait Objects vs Generics
```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Shape for Rectangle {
        self.width * self.height
    }
}

struct Circle {
    radius: f64,
}

impl Shape for Circle {

struct RightAngleTriangle {
    base: f64,
    height: f64,
}

impl Shape for RightAngleTriangle {
        0.5 * self.base * self.height
    }
}
```

Now define some functions that make use of shapes. The print_area_static function makes use of generics and the print_area_dynamic makes use of trait objects. Both functions work as expected and will print out the area of the provided shape.

```rust
fn print_area_static<S: Shape>(shape: S) {

	println!("{}", shape.area());
}

This has some important implications:
1. For the dynamic version the Rust compiler cannot know the size of the shape that will be provided. Therefore, one must pass the shape as a reference. In this case a & was used, allocation to the heap with a Box is also commonly used.
2. The dynamic implementation adds some overhead leading to worse performance. (REFERENCE).
3. Not all traits can be used as trait objects, they have to be object safe (REFERENCE).

Using trait objects is less flexible and will likely lead to worse performance. 

What is the point of trait objects then? Well as the generics version will produce a unique function for each type of shape a generic implementation cannot make use of multiple shapes at once. Consider a function to find the total area of a number of shapes.
```rust
fn total_area_static<S: Shape>(shapes: Vec<S>) -> f64 {
    shapes.iter().fold(0., |acc, shape| acc + shape.area())

fn total_area_dynamic(shapes: Vec<&dyn Shape>) -> f64 {
```


ERROR

However, the dynamic version works fine:

DYNAMIC RESULT

Note: if the vector contains the same shape there is no problem and it will successfully compile and sum the shapes.




Refer to a post on whether to use enums or traits for polymorphism.
# Resources
[Good summary of dynamic dispatch used by trait objects vs static by generics](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#trait-objects-perform-dynamic-dispatch)
[Rationale for the impl feature](https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md)


Useful resources (traits vs enums):
- https://users.rust-lang.org/t/trait-or-enum-for-sum-types/23310
- https://stackoverflow.com/questions/52240099/should-i-use-enums-or-boxed-trait-objects-to-emulate-polymorphism


# Code
```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Rectangle {
    width: f64,
    height: f64,
}
impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
}

struct Circle {
    radius: f64,
}

    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius.powi(2)
    }
}

struct RightAngleTriangle {
    base: f64,
    height: f64,
}
impl Shape for RightAngleTriangle {
    fn area(&self) -> f64 {
        0.5 * self.base * self.height
    }
}

    shapes.iter().fold(0., |acc, shape| acc + shape.area())
}

fn total_area_static<T: Shape>(shapes: &Vec<&T>) -> f64 {
    shapes.iter().fold(0., |acc, shape| acc + shape.area())
}

fn add_two_areas<S1: Shape, S2: Shape>(shape1: &S1, shape2: &S2) -> f64 {
    shape1.area() + shape2.area()
}

fn describe<T: Shape>(shape: &T) {
    println!("This shape has an area of {}", shape.area());

fn main() {
    let rect = Rectangle { width: 5., height: 10. };
    let circle = Circle { radius: 5. };
    let triangle = RightAngleTriangle { base: 5., height: 10. };
    let total = total_area(&vec![&rect, &circle, &triangle]);
    let total_static = total_area_static(&vec![&rect, &rect]);
    let add = add_two_areas(&rect, &circle);
    describe(&rect);
    println!("Areas:\nRectangle: {}\nCircle: {}\nTriangle: {}\nTotal: {}\nTotal static: {}\nAdd: {}", rect.area(), circle.area(), triangle.area(), total, total_static, add);
}
