## Intro

_31 March 2021 · #rust · #traits_

Have you ever wondered what's the difference between:

- `Deref<Target = T>`, `AsRef<T>`, and `Borrow<T>`?
- `Clone`, `Copy`, and `ToOwned`?
- `From<T>` and `Into<T>`?
- `TryFrom<&str>` and `FromStr`?
- `FnOnce`, `FnMut`, `Fn`, and `fn`?

Or ever asked yourself the questions:

- _"When do I use associated types vs generic types in my trait?"_
- _"What are generic blanket impls?"_
- _"How do subtraits and supertraits work?"_
- _"Why does this trait not have any methods?"_

Well then this is the article for you! It answers all of the above questions and much much more. Together we'll do a quick flyby tour of all of the most popular and commonly used traits from the Rust standard library!

You can read this article in order section by section or jump around to whichever traits interest you the most because each trait section begins with a list of links to **Prerequisite** sections that you should read to have adequate context to understand the current section's explanations.
