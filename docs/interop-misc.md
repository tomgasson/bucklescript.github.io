---
title: Miscellaneous
---

## Composing `bs` Attributes

As you might have guessed, most `bs.*` attributes can be used together. Here's an extreme example:

```ocaml
external draw: (_ [@bs.as "dog"]) -> int array -> unit = "draw" [@@bs.val] [@@bs.scope "global"] [@@bs.splice]

let _ = draw [|1;2|]
```

```reason
[@bs.val] [@bs.scope "global"] [@bs.splice]
external draw : ([@bs.as "dog"] _, array(int)) => unit = "draw";

draw([|1, 2|]);
```

Output:

```js
global.draw("dog", 1, 2);
```

## Safe External Data Handling

In some cases, the data could either come from JS or BS; it's very hard to give precise type information because of this. For example, for an external promise whose creation could come from the JS API, its failed value caused by `Promise.reject` could be of any shape.

BuckleScript provides a solution, `bs.open`, to filter out OCaml structured exception data from the mixed data source. It preserves type safety while allowing you to deal with mixed source. It makes use of OCaml’s extensible variant, so that users can mix values of type `exn` with JS data.

```ocaml
let handleData = function [@bs.open]
 | Invalid_argument _ -> 0
 | Not_found -> 1
 | Sys_error _ -> 2

(* handleData is 'a -> int option *)
```

```reason
let handleData = [@bs.open] (
  fun
  | Invalid_argument(_) => 0
  | Not_found => 1
  | Sys_error(_) => 2
);

/* handleData is 'a => option(int) */
```

For any input source, as long as it matches the exception pattern (nested pattern match supported), the matched value is returned, otherwise return `None`.
