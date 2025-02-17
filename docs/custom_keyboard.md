# Adding a new keyboard layout

You can import keyboard layout definitions from different sources:

- A large collection of keyboard layouts can be found in the [CLDR project](https://unicode.org/reports/tr35/tr35-keyboards.html), and we have tools to parse those.
- You can also import keyboard layouts produced by the [Kalamine project](https://github.com/OneDeadKey/kalamine).
- If none of these are available, you can write your own layout definition files.

Whatever path you choose, the entry point to adding new keyboard layouts is the `keyboard-layout-generator` package.
It contains the scripts which take keyboard layout definitions in various formats and generate TypeScript files with the same layouts converted into our own internal representation.
To start the generator, run the following shell commands:

```sh
cd packages/keybr-keyboard-generator
npm run generate
```

The generator will write files to `packages/keybr-keyboard/lib/data/layout`. You should not modify the generated files, as your changes will be lost if the generator is run again.

## Adding a custom keyboard layout

Custom keyboard layout definition files are located in `packages/keybr-keyboard-generator/lib/layout`.

You can add your own keyboard layout by copying and modifying an existing one. The configuration format is straightforward, each physical key location (like `"KeyA"`, `"KeyB"`, etc.) is mapped to a list of up to four code points.
The four code points are given for the following key modifiers:

1. No modifier.
2. `Shift` modifier.
3. `AltGr` modifier.
4. `Shift` + `AltGr` modifier.

The code points can be given as:

- A string of up to four characters long.
- An array of up to four strings or numeric code point values.

```typescript
export default {
  KeyQ: "qQ",
  KeyW: ["w", "W"],
  KeyE: [0x0065, 0x0045],
  // ... more keys ...
};
```

Your keyboard layout can also include [dead keys](https://en.wikipedia.org/wiki/Dead_key).

A dead key can be configured as a [combining diacritical mark](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks) code point:

```typescript
export default {
  KeyQ: [
    /* COMBINING GRAVE ACCENT */ 0x0300,
    /* COMBINING CIRCUMFLEX ACCENT */ 0x005e,
  ],
  // ... more keys ...
};
```

Or as a printable diacritical mark string prefixed with the `"*"` character:

```typescript
export default {
  KeyQ: ["*`", "*^"],
  // ... more keys ...
};
```

Please note that for simplicity we only support dead keys which combine base characters with diacritical marks.
We do not support dead keys which switch between different alphabets, produce non-letter characters, etc.

## Configuring a keyboard layout

The generated layout files contain only mappings to code points.
Each layout must also have an id and a name, and these are configured elsewhere.

To complete a layout configuration, add a new entry to the `Layout` class defined in `packages/keybr-layout/lib/layout.ts`.

```typescript
static readonly EN_CUSTOM = new Layout(
    /* id= */ "en-custom",
    /* xid= */ 0xff,
    /* name= */ "My Custom Layout",
    /* family= */ "qwerty",
    /* language= */ Language.EN,
    /* emulate= */ false,
    /* geometries= */ new Enum(
      Geometry.STANDARD_102,
      Geometry.STANDARD_102_FULL,
      Geometry.STANDARD_101,
      Geometry.STANDARD_101_FULL,
      Geometry.MATRIX,
    ),
);
```

### Layout id, xid and name

Give the layout a name, select a unique `id` and `xid`
(Just increase the largest `xid` of an existing layout by one.)
`id` is a human-readable string that we store in JSON files.
`xid` is a single-byte numeric identifier that we store in binary files.
Once assigned, these identifiers must never change.

### Layout family

Lesson results from different keyboard layouts that belong to the same layout family are combined and show together.
Otherwise, switching from the *United States* to the *United Kingdom* would invalidate your previous results as if you switched to a completely new layout.
However, since these two layouts belong to the same family `"qwerty"`, this is not the case.

The most common families are `"qwerty"`, `"qwertz"` and `"azerty"`.
If your layout does not belong to any of these families, you must select a custom family.

### Layout language

The language option allows filtering all available layouts by a selected language.

### Layout emulation

The emulation option can be enabled if your layout does not include dead keys.

### Layout geometries

The geometries option allows the selection of the enabled layout geometries, such as *ISO*, *ANSI*, etc.
Some layouts are only available with a specific geometry, such as *matrix*.

The first listed geometry is the preferred one.
