# Edge autofill crash: hidden native `<select>`

Minimal reproduction attempt for a renderer crash (`STATUS_ACCESS_VIOLATION`)
in Microsoft Edge 153 on Windows, seen when browser address autofill fills a
form that contains a visually hidden native `<select>` inside a CSS container
query.

Static HTML, no build step, no dependencies to install.

## The cases

| Page | Contents | Edge 153 |
| --- | --- | --- |
| `native.html` | Address form plus one visually hidden `<select>`. No JavaScript. | No crash |
| `native-control.html` | The same form without the hidden `<select>`. | No crash |
| `radix.html` | React and `@radix-ui/react-select` 2.0.0 from a CDN. Radix renders the hidden `<select>` itself. | No crash |
| `radix.html?no-select=1` | The same page without the Radix dropdown. | No crash |
| `native-container.html` | Case 1 plus `container-type: inline-size` on the form. | Untested |
| `radix.html?container=1` | Case 3 plus the same container query. | Untested |

Cases 1 to 3 were tested in Edge 153.0.4234.32 and none crashed, so the hidden
`<select>` on its own is not enough. Cases 4 and 5 add a CSS container query on
the form, which the affected site has (a Tailwind `@container` class) and these
pages previously lacked. A container query makes the form the containing block
for the absolutely positioned hidden `<select>`.

## Steps to reproduce

1. In Edge, save at least one address under `edge://settings/personalinfo`.
2. Open `native-container.html`.
3. Click the **Full name** field.
4. Hover over an address suggestion in the autofill popup, without clicking.
5. The tab crashes with `STATUS_ACCESS_VIOLATION`. The crash ID is in
   `edge://crashes`.

Then open `native.html`, which is identical except for the container query, and
repeat.

## Why this matters

Radix UI's `Select` renders a visually hidden native `<select>` inside forms to
support browser autofill (`BubbleSelect` in 2.0.0, `SelectBubbleInput` in
2.3.x), and site authors can't opt out before 2.3.x. The `native*.html` pages
reproduce that markup without React or Radix, using only the element and the
styles Radix produces, so any crash can be attributed to the browser rather
than to a library.

Edge fills every field in the form from a single suggestion, whatever the
fields' `autocomplete` or `name` attributes say, so the `<select>` can't be
excluded from the fill.

Radix has open reports about this hidden `<select>` misbehaving in constrained
layouts: [#3875](https://github.com/radix-ui/primitives/issues/3875) and
[PR #4131](https://github.com/radix-ui/primitives/pull/4131).

## Environment

- Microsoft Edge 153.0.4234.32 (Official build) (64-bit), Windows
- Not reproducible in Chrome 153
- `@radix-ui/react-select` 2.0.0 (2.3.7 renders the same hidden `<select>`)

## Run it locally

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000/>.

## Deploy to Vercel

The repo is static, with no framework and no build step.

- **Dashboard:** import the repo, set Framework Preset to **Other**, leave the
  build and output settings empty, and deploy.
- **CLI:** run `npx vercel deploy` in this folder and accept the defaults.
