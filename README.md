# Edge autofill crash: hidden native `<select>`

Minimal reproduction of a renderer crash (`STATUS_ACCESS_VIOLATION`) in
Microsoft Edge 153 on Windows. Using browser address autofill on a form that
contains a visually hidden native `<select>` crashes the tab.

Static HTML, no build step, no dependencies to install.

## The cases

| Page | Contents | Result in Edge 153 |
| --- | --- | --- |
| `native.html` | Address form plus one visually hidden `<select>`. No JavaScript. | Crashes |
| `native-control.html` | The same form without the hidden `<select>`. | Fills normally |
| `radix.html` | React and `@radix-ui/react-select` 2.0.0 from a CDN. Radix renders the hidden `<select>` itself. | Crashes |
| `radix.html?no-select=1` | The same page without the Radix dropdown. | Fills normally |

## Steps to reproduce

1. In Edge, save at least one address under `edge://settings/personalinfo`.
2. Open `native.html`.
3. Click the **Full name** field.
4. Hover over an address suggestion in the autofill popup, without clicking.
5. The tab crashes with `STATUS_ACCESS_VIOLATION`. The crash ID is in
   `edge://crashes`.

Then open `native-control.html` and repeat. The form fills normally, so the
hidden `<select>` is the only difference between the two.

## Why this matters

Radix UI's `Select` renders a visually hidden native `<select>` inside forms to
support browser autofill (`BubbleSelect` in 2.0.0, `SelectBubbleInput` in
2.3.x). Any site using that component inside an address form is affected, and
site authors can't opt out. `native.html` shows the same crash without React or
Radix, using only the markup and styles Radix produces.

Edge fills every field in the form from a single suggestion, whatever the
fields' `autocomplete` or `name` attributes say, so the `<select>` can't be
excluded from the fill.

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
