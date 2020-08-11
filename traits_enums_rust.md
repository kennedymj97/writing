# Best practices for polymorphism in Rust: Enums vs Traits and Structs

When learning Rust I had some issues with how I implement polymorphism. It felt most natural to me to use traits, however there are some issues with traits (for good reasons) that make them a bit painful to work with. This post explains my current thinking on polymorphism, hopefully it will help you avoid making the mistakes I made.

## What is polymorphism?
Polymorphism is multiple different types implementing the same interface. It is different types that have common behaviour.

There are two ways we can implement polymorphism in Rust, enums and traits.

Structure:
1. Simple example of how to do use both enums and traits for polymorphism, with explanation.
2. Readability
3. Extensibility
4. Object safety
5. Summary of approach, considering the above points what approach do I use?
6. Traits with generics vs trait objects

## Enums
### How to do it.
```rust
#[derive(Debug)]
pub enum Unit {
	Metres(u64),
	Centimetres(u64),
	Millimetres(u64),
}

pub enum Shape {
	Rectangle { width: Unit, height: Unit },
	Circle { radius: Unit },
	RightAngleTriangle { base: Unit, height: Unit },
}

impl Shape {
	pub fn area(&self) -> Unit {
		// should I try to come up with an example that has longer methods?
		match self {
			Shape::Rectangle { width, height } => rectangle::area(width, height),
			Shape::Circle { radius } => ...,
			Shape::RightAngleTriangle { base, height } => ...,
		}
	}
}

mod rectangle {
	use super::*;

	pub fn new(width: Unit, height: Unit) -> Shape {
		Shape::Rectangle { width, height }
	}
	
	pub fn area(width: Unit, height: Unit) -> Unit {
		match width {
			Unit::Metres(x) => match height {
				Unit::Metres(y) => Unit::Metres(x * y),
				Unit::Centimetres(y) => Unit::Centimetres((x * 100) * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 1000) * y),
			}
			Unit::Centimetres(x) => match height {
				Unit::Metres(y) => Unit::Centimetres(x * (y * 100)),
				Unit::Centimetres(y) => Unit::Centimetres(x * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 10) * y),
			}
			Unit::Millimetres(x) => match height {
				Unit::Metres(y) => Unit::Millimetres(x * (y * 1000)),
				Unit::Centimetres(y) => Unit::Millimetres(x * (y * 100)),
				Unit::Millimetres(y) => Unit::Millimetres(x * y),
			}
		}
	}
}

mod circle {
	...
}

mod right_angle_triangle {
	...
}

fn main() {
    let area = rectangle::new(Unit::Metres(3), Unit::Centimetres(3)).area();
    println!("{:?}", area);
}
```
## Traits

```rust
trait Shape {
	fn area(&self) -> Unit;
}

enum Unit {
	Metres(u64),
	Centimetres(u64),
	Millimetres(u64),
}

struct Rectangle {
	width: Unit,
	height: Unit,
}

impl Rectangle {
	fn new(width: Unit, height: Unit) -> Self {
		Rectangle { width, height }
	}
}

impl Shape for Rectangle {
	fn area(&self) -> Unit {
		match width {
			Unit::Metres(x) => match height {
				Unit::Metres(y) => Unit::Metres(x * y),
				Unit::Centimetres(y) => Unit::Centimetres((x * 100) * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 1000) * y),
			}
			Unit::Centimetres(x) => match height {
				Unit::Metres(y) => Unit::Centimetres(x * (y * 100)),
				Unit::Centimetres(y) => Unit::Centimetres(x * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 10) * y),
			}
			Unit::Millimetres(x) => match height {
				Unit::Metres(y) => Unit::Millimetres(x * (y * 1000)),
				Unit::Centimetres(y) => Unit::Millimetres(x * (y * 100)),
				Unit::Millimetres(y) => Unit::Millimetres(x * y),
			}
		}
	}
}

struct Circle {
	radius: Unit,
}

impl Circle {
	...
}

impl Shape for Circle {
	...
}

struct RightAngleTriangle {
	base: Unit,
	height: Unit,
}

impl RightAngleTriangle {
	...
}

impl Shape for RightAngleTriangle {
	...
}


fn main() {
	let area = Rectangle::new(Unit::Metres(3), Unit::Centimetres(3)).area();
    println!("{:?}", area);
}
```

## Readability
## Extensibility
## Object safety



For enums:
- Readability/navigability

For traits:
- Extensibility

Against traits:
- Object safety, this is really the key reason to favour enums over traits, the fact that the size of the traits cannot be known by the compiler.

---------------------------------------------------------------------------------------------------

- What is the point of this article?
	- What behaviour/pattern am I trying to discuss?
	- Discussion of the different possible resulting structures?
- Discussion on enums
	- Look into current discussion and find complaints/benefits, do I agree/disagree?
	- Readability
	- Completely document the options and behaviour
		- Discuss complaint that it creates very ugly methods
	- Object safety (look into this, what does it mean, show examples, e.g. ensuring a trait is clonable)
		- [Good post on object safety](https://huonw.github.io/blog/2015/01/object-safety/)
- Discussion on traits
	- Look into current discussion and find complaints/benefits, do I agree/disagree?
	- Extensibility (outside of module)
	- Generics, capturing some generic property or behaviour of a type
- What approach do I use/recommend?
	- Start off with enums
	- Cases not to use enums
		- The sub-types/variants (what do you call them?) need to have methods specific to them. If this is the case is it better to use an enum that wraps the structs as in the below code?
```rust
		struct Apple;
		struct Banana;
		
		enum Fruit {
			Apple(Apple),
			Banana(Banana),
		}
		```
		- Want to enable users of the module to create their own types, the users should not have to go into the source code and change an enum.
		


### Resources:
https://stackoverflow.com/questions/52240099/should-i-use-enum-to-emulate-the-polymorphism-or-use-trait-with-boxtrait-inste
https://smallcultfollowing.com/babysteps/blog/2015/08/20/virtual-structs-part-3-bringing-enums-and-structs-together/

[Something I disagree with](https://youtu.be/erJdCIti_O8?t=615)
- The method does not have to be long, can extract all of the complex logic into functions, then call these functions in the match. In fact I argue this HELPS readability, all of the possible types are immediately apparent based on the enum and the behaviour of each type can be found easily by going to the functions, effectively acts a a table of contents or list to find all of the logic.

**Thoughts:**
- Below are some example to work from, can show how enums are good. Need to do the traits implementation. Use the operator overloading examples to show what traits are good for! Think I need to do some discussion on trait objects here too as another potential reason to use traits.
- Can you add clone to all the structs that implement a trait and then clone a vec of said types (ensuring different boxed ones?). I don't think so.

```rust
use std::f64::consts::PI;
use std::ops::Mul;

#[derive(Debug, Clone, Copy)]
enum Unit {
	Metres(u64),
	Centimetres(u64),
	Millimetres(u64),
}

impl Mul for Unit {
	type Output = Self;

	fn mul(self, rhs: Self) -> Self {
		match self {
			Unit::Metres(x) => match rhs {
				Unit::Metres(y) => Unit::Metres(x * y),
				Unit::Centimetres(y) => Unit::Centimetres((x * 100) * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 1000) * y),
			}
			Unit::Centimetres(x) => match rhs {
				Unit::Metres(y) => Unit::Centimetres(x * (y * 100)),
				Unit::Centimetres(y) => Unit::Centimetres(x * y),
				Unit::Millimetres(y) => Unit::Millimetres((x * 10) * y),
			}
			Unit::Millimetres(x) => match rhs {
				Unit::Metres(y) => Unit::Millimetres(x * (y * 1000)),
				Unit::Centimetres(y) => Unit::Millimetres(x * (y * 100)),
				Unit::Millimetres(y) => Unit::Millimetres(x * y),
			}
		}
	}
}


enum Shape {
	Rectangle { width: Unit, height: Unit },
	Circle { radius: Unit },
	RightAngleTriangle { base: Unit, height: Unit },
}

impl Shape {
	fn area(&self) -> Unit {
		// should I try to come up with an example that has longer methods?
		match self {
			Shape::Rectangle { width, height } => get_area_of_rectangle(width, height),
			//Shape::Circle { radius } => PI * radius.powi(2),
			//Shape::RightAngleTriangle { base, height } => 0.5 * base * height,
		}
	}
}

fn get_area_of_rectangle(width: Unit, height: Unit) -> Unit {
	width * height
}

fn main() {
    let area = new_rectangle(Unit::Metres(3), Unit::Centimetres(3)).area();
    println!("{:?}", area);
}

// Implement the display trait for the enum. How would you do this for a shape trait, would it be using a super trait???
// What implications does that have due to object safety and having to box or whatever, example is the clone trait.

// Can use a seperate constructor for each variant
// One constructor will not work because of different fields
fn new_rectangle(width: Unit, height: Unit) -> Shape {
	Shape::Rectangle { width, height }
}
```
https://web.archive.org/web/20180121110408/http://keepcalmandlearnrust.com/2017/03/polymorphism-in-rust-enum-vs-trait-struct/
