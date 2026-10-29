# Lesson image organization

Keep supplied photos and infographics in `docs/assets/images/session-NN/`.
Use descriptive lowercase filenames with hyphens, preserving the original format.
For example, a future session-six image belongs in
`docs/assets/images/session-06/descriptive-topic.jpg`.

Keep code-created SVG diagrams in `docs/assets/diagrams/session-NN/`.

## Session 4 supplied images

| Stored filename | Original file | Placement |
| --- | --- | --- |
| `docs/assets/images/session-04/methods-overview.jpg` | Desktop `01.jpg` | Method definition, inputs, return values, and void |
| `docs/assets/images/session-04/value-reference-types.jpg` | Desktop `02.jpg` | Copying values and references, before the string section |

The image files are unchanged copies. Explanations beside the second image
clarify its simplified stack/heap diagram, fictional address, mutable shared
objects, and the difference between changing an object and reassigning a reference.

In lesson Markdown, use a path relative to the source page, for example
`../assets/images/session-04/methods-overview.jpg` from `docs/csharp/session-04.md`.
Zensical rewrites it for the generated page. Use the `lesson-illustration` figure
class, meaningful alt text, the image's original width and height, a caption,
and a link to the original image so students can enlarge detailed text.
