---
layout: default
title: Lists in a Model
nav_order: 7
---
# Lists as a Model

Like for any other type you can create a `Store` that holds a list:

```kotlin
val listStore = RootStore<List<String>>(listOf("a","b","c"))
```

Of course you can handle this the same way we have seen before: map the `data` to a `Flow` of `Tag`s by iterating over the `List` and bind it to some place in your `render`. This is perfectly valid, but for `List`s that change more often it would not be performing well (your whole list will be rerendered whenever the model changes).

For those cases fritz2 offers the method `each()` on a `Flow<List<?>>` that creates a `Seq` from your `Flow`. `Seq` offer intermediate operations, that work on the elements of your List.

Binding a `Seq` in your `render` context works exactly as for a `Flow` by just calling `bind()` on it. But instead of a `DomMountPoint` a `DomMultiMountPoint` will be created. This kind of `MountPoint` gets a `Flow` of patches as it's upstream from the `Seq` and is therefore able to change only the DOM-elements that need to be changed (when you add a new element to your List for example or remove one):

```kotlin
    val seq = object : RootStore<List<String>>(listOf("one", "two", "three")) {
        var count = 0

        val addItem = handle { list ->
            count++
            list + "yet another item no. $count"
        }
        val deleteItem = handle<String> { list, current ->
            list.minus(current)
        }
    }

    val myComponent = render {
        section {
            ul {
                seq.data.each().map { s ->
                    render {
                        li {
                            button("btn", id = "delete-btn") {
                                text(s)
                                clicks.map { console.log("deleting $s"); s } handledBy seq.deleteItem
                            }
                        }
                    }
                }.bind()
            }
            button("button") {
                text("add an item")
                clicks handledBy seq.addItem
            }
        }
    }
```

So `each()` allows you to use _one-way-databinding_ when working with the elements in your `List`. If you need _two-way-databinding_ (for example to edit the single elements in a form, etc.), just call `eachStore()` on the `Store` instead. This gives you a `Seq` of `SubStore`s, one for each element of you `List`. You can use them just like any other `Store` to build a form and bind your data.
If you have to further process your data-`Flow` before, you can still use `each()` and call `sub()` inside the each body with the element to get the `Store` representing that element in your list.

Like for the single elements in your `List` you will also need to get `Store`s for elements "hidden" deeper in your [Nested Structures](NestedStructures.html). Let's have a look at what we can do for you.
