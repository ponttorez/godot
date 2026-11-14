# Godot boxe

The `boxe` branch is Godot 4.7.2-stable. The fork has no versions: each change is a commit of its own and a line here. Changes are grouped by feature; rebases and force pushes are allowed.

## Build

- The build is named `boxe`: the version is `4.7.2.stable.mono.boxe.<commit>`.
- Export templates are looked up in a folder of their own, `export_templates/4.7.2.stable.mono.boxe`; the official templates aren't downloaded for this build.
- The .NET packages (`GodotSharp`, `Godot.NET.Sdk`, `Godot.SourceGenerators`) are versioned `4.7.2-boxe`, so NuGet doesn't mix them up with the official ones.
- The CI of the fork runs only on demand.

## Small changes

- The `EditorExport` singleton is cleared when it's freed (tests create and free it).
- The inspector plays any `AudioStream`: randomizers, playlists and localized streams get play and stop buttons.
- `internationalization/locale/remember_selected_locale`: the locale chosen in the game is saved to `user://locale.cfg` and restored at launch; `TranslationServer.forget_selected_locale()`, the `locale_changed` signal.
- `Control.raycast_padding_left/top/right/bottom`: the input area is larger or smaller than the control without changing the layout; the editor outlines it on the selected control.
- `AudioStreamRandomizer.random_pitch_semitones_min/max`, `random_volume_offset_db_min/max`: asymmetric ranges of pitch and volume. The old properties are deprecated and load as a range.
- `Control.get_focus_rect()` / `_get_focus_rect()` and `get_visible_focus_rect()`: the part of a control that has the focus (tab, row, cell, menu item), clipped by its parents; `PopupMenu.get_focused_item_rect()`. For a custom focus cursor.

## Mobile platforms and safe area

- Fixed the inverted Swappy modes (`pipeline_forced_on` enabled automatic FPS, the default mode disabled it).
- `application/config/version_code`: the build number of the project. Exporters use it when the number of the preset is empty or 0: as the version code on Android, the bundle version on iOS, visionOS and macOS, and the fourth number of the version on Windows. The version and build number can be read in the exported game; Android warns about a code lower than the project's and above the Google Play limit.
- `display/window/android/refresh_rate` (`System`/`Maximum`): requests the highest refresh rate of the display on Android, and Swappy paces frames for it.
- The `DisplayServer.safe_area_changed` signal (Android, iOS): the safe area changed, e.g. after a rotation.
- `MarginContainer.safe_area` (`None`/`Inside`/`Outside`): the margins take the safe area into account, keeping the content inside it or extending it under cutouts and system bars up to the screen edge. Android and iOS only.

## Interface, text and localization

- `Control.interactable` (`Inherited`/`Disabled`/`Enabled`), `is_interactable_in_tree()`: blocks the input of a branch of controls without changing how it looks: no clicks, hover, focus, drag and drop or shortcuts; `Enabled` unblocks a descendant.
- `FontFile/SystemFont.outline_mode` (`Smooth`/`Pixel`), a font import option, `TextServer.font_set_outline_mode()`: `Pixel` grows the rasterized glyph by `outline_size` pixels in every direction, diagonals included, for crisp outlines of pixel fonts, shadow outlines too.
- `FontVariation.spacing_left/right`: space at the start and end of every line; `spacing_glyph` only goes between the elements of a line, inline images included.
- `Control.composite_enabled`, `composite_margin`, `composite_include_self`; `CanvasItem.set_drawing_canvas_item()`, `get_own_canvas_item()`: a control and its children are drawn as a single image, like `CanvasGroup`, so transparency and the material apply to the whole. The children of groups no longer get the modulation twice. Nested composites, `CanvasGroup`s and `clip_children` nodes each draw into a buffer of their own (both renderers).
- `BBCodeParser`: the tag parsing of `RichTextLabel` moved to a shared class.
- `LocalizedTexture2D`, `LocalizedAudioStream`, `LocalizedFont`: a resource per locale, chosen like translations and switched when the locale changes.
- `bbcode_enabled` on `Label`, `Button`, `LinkButton`, `TabBar`, `PopupMenu`, `MenuBar`, `ItemList`, `Tree` and the `FoldableContainer` title; `tooltip_/title_/dialog_bbcode_enabled`; the `[spacer]` tag. The tags of `RichTextLabel` except tables, links and hints with `[url]` / `[hint]`; bold and italics are synthesized from the regular font when not set.
- BBCode effects in these controls: `[wave]`, `[shake]`, `[pulse]`, `[rainbow]`, `[tornado]`, `[fade]`, custom ones through `custom_effects`; `CharFXTransform.appear_time` for typing text letter by letter. Redrawn every frame only while effects are on screen. The outlines and shadows of all lines are drawn before the glyphs, so the outline of a line doesn't cover its neighbors.
- `text_arguments` on text nodes (`tooltip_text_arguments`, `title_text_arguments`, arguments of items and tabs): `{name}` and plural forms `{n|form|form|form}` by the rules of the locale; `TranslationServer.format_arguments()`.

## Limitations

- Each level of nested composites is a buffer the size of the render target and an extra pass.
- Text with effects is redrawn every frame while it's visible.
- Pixel outlines don't apply to MSDF fonts.
