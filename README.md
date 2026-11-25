# Godot boxe

The `4.7` branch is Godot 4.7.2-stable with fixes cherry-picked from upstream. The fork has no versions: each change is a commit of its own and a line here. Changes are grouped by feature; rebases and force pushes are allowed.

## Upstream fixes

- `[macOS] Add SO_ERROR check to NetSocketUnix inet connect.` (cherry-picked from `d4644ff602`).

## Build

- The build is named `boxe`, the builds of the CI too: the version is `4.7.2.stable.mono.boxe.<commit>`.
- Export templates are looked up in a folder of their own, `export_templates/4.7.2.stable.mono.boxe`; the official templates aren't downloaded for this build, and installing templates whose `version.txt` has the official version warns that this build doesn't use them.
- The .NET packages (`GodotSharp`, `Godot.NET.Sdk`, `Godot.SourceGenerators`) are versioned `4.7.2-boxe`, so NuGet doesn't mix them up with the official ones.
- The CI of the fork runs only on demand.
- The math tests ignore the false divisions by zero (C4723) that the MSVC of the GitHub runners reports in `cubic_interpolate_in_time()`.

## Small changes

- The inspector plays any `AudioStream` that makes sound on its own: randomizers and playlists get play and stop buttons; microphone, generator and polyphonic streams are left out.
- `internationalization/locale/remember_selected_locale`: the locale chosen in the game is saved to `user://locale.cfg` and restored at launch; `TranslationServer.forget_selected_locale()`.
- `Control.raycast_padding_left/top/right/bottom`, `get_raycast_rect()`: the input area is larger or smaller than the control without changing the layout; the editor outlines it on the selected control. It also limits the click mask of `TextureButton` and pads the title of a folded `FoldableContainer`.
- `AudioStreamRandomizer.random_pitch_semitones_min/max`, `random_volume_offset_db_min/max`: asymmetric ranges of pitch and volume. The old properties are deprecated and load as a range.
- `Control.get_focus_rect()` / `_get_focus_rect()` and `get_visible_focus_rect()`: the part of a control that has the focus (tab, row, cell, menu item), clipped by its parents; `PopupMenu.get_focused_item_rect()`. For a custom focus cursor.
- `StyleBox._get_style_margin()`: a style box made in a script gives the content margins of its sides when `content_margin_*` is negative. While a control draws it, `get_current_item_drawn()` returns that control, so the controls that share the style box can get margins of their own, e.g. to animate the text of a button.

## Mobile platforms and safe area

- Fixed the inverted Swappy modes (`pipeline_forced_on` enabled automatic FPS, the default mode disabled it).
- `application/config/version_code`: the build number of the project. Exporters use it when the number of the preset is empty or 0: as the version code on Android, the bundle version on iOS, visionOS and macOS, and the fourth number of the version on Windows. The version and build number can be read in the exported game; Android warns about a code lower than the project's and above the Google Play limit.
- `display/window/android/refresh_rate` (`System`/`Maximum`): requests the highest refresh rate of the display on Android, and Swappy paces frames for it.
- The `DisplayServer.safe_area_changed` signal (Android, iOS): the safe area changed, e.g. after a rotation.
- `MarginContainer.safe_area` (`None`/`Inside`/`Outside`): the margins take the safe area into account, keeping the content inside it or extending it under cutouts and system bars up to the screen edge. Android and iOS only.

## Interface, text and localization

- `Control.interactable` (`Inherited`/`Disabled`/`Enabled`), `is_interactable_in_tree()`: blocks the input of a branch of controls without changing how it looks: no clicks, hover, focus, drag and drop, shortcuts or actions of assistive technologies; `Enabled` unblocks a descendant.
- `FontFile/SystemFont.outline_mode` (`Smooth`/`Pixel`), a font import option, `TextServer.font_set_outline_mode()`: `Pixel` grows the rasterized glyph by `outline_size` pixels in every direction, diagonals included, for crisp outlines of pixel fonts, shadow outlines too.
- `FontVariation.spacing_left/right`: space at the start and end of every line; `spacing_glyph` only goes between the elements of a line, inline images included.
- `Control.composite_enabled`, `composite_margin`, `composite_include_self`: a control and its children are drawn as a single image, like `CanvasGroup`, so transparency and the material apply to the whole; `clip_children` has no effect meanwhile. The children of groups no longer get the modulation twice. Nested composites, `CanvasGroup`s and `clip_children` nodes each draw into a buffer of their own (both renderers). In C++, `CanvasItem::set_drawing_canvas_item()` makes a node draw into another canvas item, and `get_own_canvas_item()` returns its own one.
- `BBCodeParser`: the tag parsing of `RichTextLabel` moved to a shared class.
- `bbcode_enabled` on `Label`, `Button`, `LinkButton`, `TabBar`, `PopupMenu`, `MenuBar`, `ItemList`, `Tree` and the `FoldableContainer` title; `tooltip_/title_/dialog_bbcode_enabled`; the `[spacer]` tag. The tags of `RichTextLabel` except tables, links and hints with `[url]` / `[hint]`; bold and italics are synthesized from the regular font when not set.
- `Label` draws the shadows of all lines, then their outlines, then the glyphs, so the outline of a line doesn't cover its neighbors.
- BBCode effects in these controls: `[wave]`, `[shake]`, `[pulse]`, `[rainbow]`, `[tornado]`, `[fade]`, custom ones through `custom_effects`; `CharFXTransform.appear_time` for typing text letter by letter. Redrawn every frame only while effects are on screen.
- `text_arguments` on text nodes (`tooltip_text_arguments`, `title_text_arguments`, arguments of items and tabs): `{name}` and plural forms `{n|form|form|form}` by the rules of the locale; `TranslationServer.format_arguments()`.

## Changes that look different in existing projects

- A `CanvasGroup` and its parents modulate its children once: an alpha of 0.5 used to look like 0.25.
- `spacing_glyph` of fonts is no longer added after the last glyph of a line, so letter-spaced text is narrower by that amount.
- The outlines of a `Label` line are drawn after the shadows of all lines and the glyphs after all outlines, instead of line by line.

## Limitations

- Each level of nested composites is a buffer the size of the render target and an extra pass.
- Text with effects is redrawn every frame while it's visible.
- Pixel outlines don't apply to MSDF fonts, nor to bitmap fonts (BMFont and image fonts), which have no outlines.
