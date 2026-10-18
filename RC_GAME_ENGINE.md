# RC Game rollback engine

This branch contains the engine changes used by
[RC Game](https://github.com/BREVIC3PS/rc), on top of the stepping-physics fork at
`bf04cdc95391fe84f3f5d9f098e1f3a294e9c0ca` (`albertok/godot`, Godot 4.7.2).
Branch: `rc-game-rollback`.

## Added behavior

- Native Jolt body snapshots (`body_get_rollback_state` and
  `body_set_rollback_state`), using the JBR3 format.
- Native space snapshots (`space_get_rollback_state` and
  `space_set_rollback_state`) that preserve contacts, body state, collision
  filters, area memberships, persistent forces and kinematic targets.
- Explicit uniform geometry changes through `body_get_rollback_scale` and
  `body_set_rollback_scale`. Scale must be between 0.25 and 4.0; bodies with
  attached joints are not supported. Snapshots retain their immutable shapes
  and restore mass properties before native simulation state.
- Topology validation: arbitrary body, joint or ordinary shape changes cannot
  be restored across a snapshot. These APIs do not implement gameplay rollback
  or undo side effects from callbacks that have already run.
- ENet skips disconnected peers and handles failed sends without stale-send
  engine error spam. SceneMultiplayer drops relayed packets from departed peers.

The stepping APIs `space_step` and `space_flush_queries` come from the base fork.
Native snapshots are local engine objects/bytes, not a portable network format.
Use matching engine and game builds on all peers.

## Windows builds

Run in a Visual Studio 2022 developer environment with SCons available:

```powershell
scons platform=windows target=editor arch=x86_64 dev_mode=yes debug_symbols=no module_text_server_fb_enabled=yes windows_subsystem=console d3d12=no accesskit=no angle=no -j8
scons platform=windows target=template_debug arch=x86_64 dev_mode=yes debug_symbols=no module_text_server_fb_enabled=yes windows_subsystem=console d3d12=no accesskit=no angle=no -j8
scons platform=windows target=template_release arch=x86_64 dev_mode=no debug_symbols=no module_text_server_fb_enabled=yes windows_subsystem=console d3d12=no accesskit=no angle=no -j8
```

Keep the standard unsuffixed editor and template outputs in `bin/`; generated
binaries and object caches are not source-controlled. Close applications using
the output binaries before rebuilding them.

## Validation

Engine changes exactly match RC Game's checked-in
`engine/patches/godot-rollback.patch` at game commit `16cf185`.
On 2026-10-18, the existing matching custom engine passed these game-side suites:

```powershell
tools/run_tests.ps1 -Suite run_net_rollback_extras,run_net_battery_scale,run_net_rollback_abort
```

Results: 23 rollback extras checks, 62 scaling checks and 36 recovery-abort checks,
all passing. The same engine/template set also passed the game's isolated Windows
export, 101 packed scenery checks, 34 packed map checks, three elimination/respawn
cycles and a rendered RC-130 release match.

The installed binaries were built before this source commit and still report
`4.7.2.stable.custom_build.bf04cdc95`. Committing this branch does not rebuild them.
After rebuilding, update the editor and both templates together: the game's
compatibility handshake checks the engine revision as well as snapshot format.
