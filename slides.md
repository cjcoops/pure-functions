---
# try also 'default' to start simple
theme: apple-basic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
---

# Pure Functions


<a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
  class="abs-br m-6 text-xl icon-btn opacity-50 !border-none !hover:text-white">
  <carbon-logo-github />
</a>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# An example

Often we do something like this:

```ts {all|9-16|3-4|12|9-16|all}
export class CartService {
    public totalPrice$ = combineLatest([
        this.cart$,
        this.products$
    ]).pipe(map([cart, products]) => {
        return this.calculateTotalPrice(cart, products);
    })

    private calculateTotalPrice(cart: Cart, products: Product[]): number {
        const totalPriceWithoutDiscount = // do all the logic!
        
        const percentageDiscount = this.userService.getUserDiscount();
        const totalPriceWithDiscount = totalPriceWithoutDiscount * (100 - percentageDiscount)/100;
        
        return totalPriceWithDiscount;
    } 
}
```
---
layout: center
class: text-center
---

# This was a simple example

---

# How can we fix this?

The code above can be divided into 2 parts based on the role it is performing:

| **Role**                                    | **Responsibilities**                                                                                                                                               | **How best to test** | **Pure or Impure?** |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- | ------------------- |
| Calculation / business logic                | Calculating total price given cart, products and discount                                                                                                          | Unit tests           | Pure                |
| Sending / receiving messages i.e. data flow | <ul><li>Receiving data from `cart$` and `products$`</li><li>calling `UserService.getUserDiscount` </li><li>pushing data to subscribers of `totalPrice$`.</li></ul> | Integration / E2E    | Impure              |

Splitting up our code by these responsibilities can help solve our problems for us.

---

# What is a pure function?

- Given the same input, will always return the same output.
- Produces no side effects.
- A dead giveaway that a function is impure is if it makes sense to call it without using its return value i.e. it returns "void", or it makes reference to "this" (not strictly true).

---

# Pure function example

Compare this

```js
const add = (x, y) => x + y;

add(2, 4); // 6
```

to this:
```js
let x = 2;

const add = (y) => {
  x += y;
};

add(4); // x === 6 (the first time)
```

---

# Favour pure functions over impure functions

Pure functions are:

- simpler
- reusable since they're independent of state
- easy to move around when re-organizing code
- independent of outside state so are immune to bugs to do with shared mutable state
- easy to test
  
---

# Extracting the business logic into a pure function

```ts {all|1-7|18-19|13-20|all}
export function calculateTotalPrice(cart: Cart, products: Product[], percentageDiscount: number): number {
    const totalPriceWithoutDiscount = // do all the logic!
    
    const totalPriceWithDiscount = totalPriceWithoutDiscount * (100 - percentageDiscount)/100;
    
    return totalPriceWithDiscount;
} 

@Injectable({
  providedIn: 'root',
})
export class CartService {
    public totalPrice$ = combineLatest([
        this.cart$,
        this.products$
    ]).pipe(map([cart, products]) => {
        // could also get this via a stream either using withLatestFrom or add to combineLatest
        const percentageDiscount = this.userService.getUserDiscount(); 
        return calculateTotalPrice(cart, products, percentageDiscount);
    })
}
```
---

# Options for extracting pure functions

- Extract to a function outside of the class (same file)
- Extract to a function in a separate file
- Create a stateless Class with only static methods e.g. `CartCalculationsService`
- Create a "utils" Class with static methods e.g. `MathUtils`
- Create a Class and instantiate e.g. `new Order(cart, products)`

