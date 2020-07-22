# Traits, Trait Objects, Enums and Generics in Rust

### Resources:
https://stackoverflow.com/questions/52240099/should-i-use-enum-to-emulate-the-polymorphism-or-use-trait-with-boxtrait-inste
https://smallcultfollowing.com/babysteps/blog/2015/08/20/virtual-structs-part-3-bringing-enums-and-structs-together/

[Something I disagree with](https://youtu.be/erJdCIti_O8?t=615)
- The method does not have to be long, can extract all of the complex logic into functions, then call these functions in the match. In fact I argue this HELPS readability, all of the possible types are immediately apparent based on the enum and the behaviour of each type can be found easily by going to the functions, effectively acts a a table of contents or list to find all of the logic.



