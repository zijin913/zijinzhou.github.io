# Genesis Maestro project-page sources

Page: `content/en/projects/genesis_maestro/index.md`. Reviewed 2026-10-10.
These are editorial provenance notes, outside Hugo's published content directory.

Server: `kaitianchao@Exx_Levine`; repository root: `/home/kaitianchao/Projects/`.

## Written evidence

- `Genesis_Maestro/README.md`, `maestro_sim/README.md`, `docs/PHASE1_REPORT.md`: platform, migration, historical validation and limitations.
- `assets_astra_real2sim/README.md`: separate visual/collision assets, provenance, provisional reconstruction status, cloth limitations.
- `Genesis_Maestro_s0_ik_guard-01/docs/BARE_PUZZLE_V2_20261010.md`: puzzle instruction, chronology, 27 responses, 0.925715 mm peak penetration, single completed trial.
- `Genesis_Maestro_s1s2-01/docs/S0_S3_TABLE_20261008.md`: use the dated amendments through October 10, not the superseded initial tables. Includes tool-retrieval conditions, prior failures, and grasp-chain repair analysis.
- `Genesis_Maestro_s1s2-01/docs/S1_S2_STAGES.md`: stage definitions, IK versus cuRobo, input boundaries.
- `Genesis_Maestro_harness_repeat-01/docs/STEREO_DEPTH_20261007.md`: selected tabletop trial outcomes and stereo/input audit.
- `Genesis_Maestro_harness_repeat-01/maestro_sim/stage_scoring.py`: staged progress is not a success percentage; knife contact does not imply material severing.

## Media

`puzzle.mp4` and `tool-retrieval.mp4` are copied from
`Genesis_Maestro_s1s2-01/outputs/s1s2_analysis/dashboard/videos/`:

- `mechanical_puzzle_box_bare_astra.mp4`
- `multistage_tool_retrieval_bare_astra.mp4`

The four other videos originate from `Genesis_Maestro_harness_repeat-01/outputs/harness_run/`, each at `<run>/<task>/trial_1/interaction_4view.mp4`:

| Website file | Run | Task |
| --- | --- | --- |
| cabinet.mp4 | open_cabinet_astra_fs_01 | open_cabinet |
| mug.mp4 | hang_mug_astra_stereo_noguide_01 | hang_mug |
| whiteboard.mp4 | erase_whiteboard_astra_stereo_01 | erase_whiteboard |
| cube.mp4 | rotate_cube_astra_fs_01 | rotate_cube_purple_up |

Tabletop videos: entire source sequence compressed at 4× source playback speed,
960-pixel width, 20 fps, H.264 CRF 29, yuv420p, faststart, no audio.
Original recordings already omit API waiting time.
`preview.mp4`: puzzle seconds 100–135, 2× source speed, 20 fps, H.264 CRF 29.
Posters: frame at second 5 of each website video; puzzle poster named `featured.jpg`.

No raw model inputs, internal reports, credentials, or server paths are included in the public page.
No claim is made that this page establishes sim-to-real transfer or a controlled causal stage ranking.
