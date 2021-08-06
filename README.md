> Demo for `props` type inference in `setup()` (Vue 3)

https://stackoverflow.com/q/68670714/6277151

When the `props` declaration contains a `default()` or `validator()` property as a regular function, the type inference for the `props` argument in `setup()` is incorrect. This applies to a project scaffolded with Vue CLI 5.0.0-beta.2, which installs Vue 3.1.5 and TypeScript 4.1.6; but other versions may be affected.

For instance, the following code:

```typescript
import { defineComponent, toRef } from 'vue';

export default defineComponent({
  name: 'App',
  props: {
    msg: String,
    when: {
      type: String,
      default: '',
      // ❌ regular function breaks type inference for `props` arg in `setup()`
      validator(value: string) {
        return ['today', 'tomorrow', 'later'].includes(value)
      },
      required: true
    },
    data: {
      type: Object,
      // ❌ regular function breaks type inference for `props` arg in `setup()`
      default() {
        return {}
      },
      required: true
    },
  },
  setup(props) {
    // ⛔️ Argument of type '"data"' is not assignable to parameter of type 'string & ((number | unique symbol | "length" | "toString" | "toLocaleString" | "concat" | "join" | "slice" | "indexOf" | "lastIndexOf" | "every" | "some" | "forEach" | "map" | ... 33 more ... | (<A, D extends number = 1>(this: A, depth?: D | undefined) => FlatArray<...>[])) & string)'.ts(2345)
    let allEvents = toRef(props, 'data')
                                 ^^^^^^
    // ⛔️ Property 'when' does not exist on type 'Readonly<LooseRequired<Readonly<readonly unknown[] & { [x: number]: string; } & { [iterator]?: IterableIterator<string> | undefined; length?: number | undefined; toString?: string | undefined; toLocaleString?: string | undefined; ... 19 more ...; flat?: unknown[] | undefined; }> | Readonly<...>>>'.ts(2339)
    if (props.when === 'today') {
              ^^^^
    }
  }
})
```

...creates this type of the `props` argument in `setup()`, which seems to be an intersection type of keys of `string`:

```typescript
(parameter) props: Readonly<LooseRequired<Readonly<readonly unknown[] & {
    [x: number]: string;
} & {
    [Symbol.iterator]?: IterableIterator<string> | undefined;
    length?: number | undefined;
    toString?: string | undefined;
    toLocaleString?: string | undefined;
    ... 19 more ...;
    flat?: unknown[] | undefined;
}> | Readonly<...>>>
```

## Workaround

Instead of regular functions, use **arrow functions** for functions in the `props` declaration (`default()` and `validator()`) to resolve the issue.

The `props` argument type becomes:

```typescript
(parameter) props: Readonly<LooseRequired<Readonly<{
    msg?: unknown;
    when?: unknown;
    data?: unknown;
} & {
    when: string;
    data: Record<string, any>;
} & {
    msg?: string | undefined;
}>>>
```
