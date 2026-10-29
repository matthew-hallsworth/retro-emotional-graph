# Retro mood graph

A single-file web page for running an "emotional journey" retrospective. Each person draws a line showing how they felt over a sprint (or any period), locks it, and pins notes to the moments they want to talk about.

There is no build step, no server and nothing to install. Everything lives in `retro-graph.html`.

## Features

- **Draw a mood line:** Press and drag across the chart. The y-axis runs from 😫 to 😄 with a neutral 😐 line in the middle, and the x-axis runs from the start of the period to the end. The area above neutral is tinted green and the area below is tinted red.
- **Redraw part of the line:** Dragging over a section you've already drawn replaces only that section.
- **Lock graph:** A checkbox that stops drawing and disables deletion, so the line can't be wiped by accident.
- **Delete line:** Clears the line (and any notes). It only works while the graph is unlocked.
- **Notes:** While the graph is locked, click the line to add a note at that point. Notes appear as numbered markers along the line and in a "Talking points" list under the chart, in timeline order. Click a marker or list item to edit or delete a note.
- **Editable labels:** The title and the start and end labels can be renamed in place.
- **Saved in the browser:** The line, notes, labels and lock state are kept in `localStorage`, so a refresh doesn't lose anything.
- **Works on touch devices** and follows the system light or dark mode.

## Using it

### Open it locally

Download or clone the repo and open `retro-graph.html` in a browser. That's all.

### Host it with GitHub Pages

1. In the repo, go to **Settings → Pages**.
2. Under **Build and deployment**, set the source to **Deploy from a branch**, choose `main` and the `/ (root)` folder, then save.
3. After a minute or so the page is available at `https://<your-user-or-org>.github.io/<repo-name>/retro-graph.html`.

If you'd like the page at the root URL instead, rename `retro-graph.html` to `index.html`.

### Running a retro with it

A format that works well:

1. Everyone opens the page on their own device and draws their line privately.
2. Each person ticks **Lock graph** and adds notes to the highs, lows and turning points.
3. People take turns sharing their screen and talking through their talking points in order.

Each person's graph is stored only in their own browser. Nothing is sent to a server or shared with anyone else.

## How it works

The page is plain HTML, CSS and JavaScript with no dependencies. The only external request is for the Bricolage Grotesque font from Google Fonts; if it doesn't load, the page falls back to the system font.

### Data model

- **The line** is an array of `N` values (currently 300), one for each evenly spaced point along the x-axis. Each value is either `null` (nothing drawn there) or a number from `-1` (worst) to `1` (best). Storing the line this way means it can only have one value per point in time, which is what makes partial redraws work. When the pointer moves quickly, the points in between are filled in by linear interpolation so the line stays continuous.
- **Notes** are objects of the form `{ id, i, text }`, where `i` is the index into the line array. A note's height is read from the line at render time, so notes follow the line if part of it is redrawn. If there's no line at that point, the note sits on the neutral line.

### Rendering

Everything on the chart is drawn on a single `<canvas>`, which is redrawn in full after every change. The canvas is sized for the device pixel ratio so it stays sharp on high-density screens, and a `ResizeObserver` redraws it when the layout changes. Colours come from CSS custom properties on `:root`, so the light and dark themes are defined in one place in the stylesheet.

### Storage

State is saved under the `localStorage` key `retro-mood-graph-v1` as JSON:

```json
{
  "values": [null, 0.42, ...],
  "notes": [{ "id": "n...", "i": 120, "text": "Release went out" }],
  "locked": true,
  "title": "Sprint retro",
  "start": "Sprint start",
  "end": "Sprint end"
}
```

All storage access is wrapped in `try`/`catch`, so the page still works (without saving) where storage is blocked. If you change the shape of this data, bump the key version (for example to `retro-mood-graph-v2`) or add handling for the old shape, so people with saved graphs don't hit errors.

### Where to find things in the file

The script is one self-contained function, organised into commented sections:

| Section | What it covers |
| --- | --- |
| `persistence` | Saving to and loading from `localStorage` |
| `geometry` | Converting between line indexes, mood values and canvas pixels, plus hit-testing for the line and note markers |
| `rendering` | Drawing the grid, faces, line, shading and note markers |
| `notes list` | Building the "Talking points" list under the chart |
| `note editor` | Opening, positioning, saving and deleting notes |
| `UI state` | Button states, cursor styles and status text |
| `canvas events` | Pointer handling for drawing and for clicking on the line |

Useful constants at the top of the script include `N` (line resolution), `PAD` (chart padding), `FACES` (y-axis faces and their values), `MARKER_R` (note marker size) and `LINE_HIT` (how close a click needs to be to count as clicking the line).

## Contributing

Contributions are welcome. To keep the project easy to pick up and host anywhere:

- **Keep it to a single HTML file** with no build step. If a library is truly needed, load it from a CDN at a pinned version rather than adding tooling.
- **Test your change by opening the file in a browser.** Before opening a pull request, please check:
  - Drawing, partial redraws, locking and deleting still work.
  - Adding, editing and deleting notes still works, and notes stay attached to the line after a redraw.
  - The page still works after a refresh (saved state loads correctly).
  - Both light and dark mode look right.
  - It works on a narrow screen and with touch, if you can test that.
- **Keep the controls keyboard-accessible** with visible focus styles.
- **Describe what you changed and why** in the pull request, with a screenshot or short screen recording for visual changes.

### Branches and pull requests

1. Create a branch from `main` (for example `feature/export-png` or `fix/marker-overlap`).
2. Make your change and test it as above.
3. Open a pull request against `main` and ask a colleague to review it.

### Ideas for future work

- Export the graph as a PNG or share it as a link
- A shared team view with everyone's lines on one chart
- Configurable y-axis labels or faces
- Undo for the last stroke

## License

No license has been chosen yet. Add a `LICENSE` file before sharing this outside your organisation.
