# MiniMax H3 Repair Context

This patch adds a `repair_context` continuity mode to Easy Media MultiTrack Project.

## Behavior

- Middle segment: previous segment = Forward Context, next segment first frame = Backward Guide.
- First segment: no Forward Context; the next segment first frame is the Backward Guide.
- Last segment: previous segment is Forward Context; there is no Backward Guide.
- Passthrough segments keep their existing behavior and are not changed by this mode.

## Important

For a repair that needs the next segment as a Backward Guide, that next segment must already have an active rendered video in the project. Use `project_save: new` when repairing a segment in the middle, because `override` would clear downstream generations before they can be used as the guide.

The existing `shot`, `context`, and `context_swap` paths are kept separate from the new repair path.

## UI

The MiniMax MultiTrack segment controls now expose `Repair Context` and the project video combine view displays the mode.

## Repair Segment Selection / Context Frames

The `MultiTrack Project` node now supports a temporary `repair_segments` selection.
Enter segment numbers as a comma-separated list, for example `2,4,7`.

When this field is non-empty:
- only those segments are queued for this run;
- their continuity mode is temporarily treated as `repair_context`;
- the source MultiTrack Editor data is copied before the temporary mode change, so the original Shot / Swap Context modes are not overwritten;
- non-contiguous repair targets use each target's actual previous saved segment as Forward Context, not the previous selected repair target;
- each selected target can still use the next saved segment as its Backward Guide.

The node also exposes `context_frames` (default `22`) as a free numeric input.
The requested number is allowed to be typed directly. H3's Motion Context implementation
snaps the usable span down to its supported temporal grid when necessary and logs both
the requested and applied values. `reference_frames` is kept separate as the hard video
anchor span and defaults to `5`.
