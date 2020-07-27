# Traits, Trait Objects, Enums and Generics in Rust

- What is the point of this article?
	- What behaviour/pattern am I trying to discuss?
	- Discussion of the different possible resulting structures?
- Discussion on enums
	- Look into current discussion and find complaints/benefits, do I agree/disagree?
	- Readability
	- Completely document the options and behaviour
		- Discuss complaint that it creates very ugly methods
	- Object safety (look into this, what does it mean, show examples, e.g. ensuring a trait is clonable)
- Discussion on traits
	- Look into current discussion and find complaints/benefits, do I agree/disagree?
	- Extensibility (outside of module)
	- Generics, capturing some generic property or behaviour of a type
- What approach do I use/recommend?
	- Start off with enums
	- Cases not to use enums
		- The sub-types/variants (what do you call them?) need to have methods specific to them. If this is the case is it better to use an enum that wraps the structs as in the below code?
		- ```rust
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



