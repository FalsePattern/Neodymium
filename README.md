# Neodymium Unofficial <img src="NDico.png" align="right" width=150>

<sup>**[CurseForge](https://www.curseforge.com/minecraft/mc-mods/neodymium-unofficial) | [Modrinth](https://modrinth.com/mod/nd1710)**</sup>

## A temporary fork of Neodymium with FalseTweaks compatibility until Makamys comes back

Neodymium is a mod that reimplements chunk rendering in Minecraft 1.7.10 using modern OpenGL. This should improve performance on most hardware.

As a side effect, it has also been reported to fix the [graphical errors](https://www.minecraftforum.net/forums/support/java-edition-support/3148472-weird-bug-with-1-7-10-and-only-1-7-10) that occur when running Minecraft with an integrated Intel GPU. Installing [CoreTweaks](https://github.com/makamys/CoreTweaks) is recommended for a more comprehensive fix.

## Benchmarks

Tests were made staying in one place without moving the camera, on Peaceful difficulty. The FPS was measured once the chunk update number had reached the single digits and the FPS had stabilized.

| Mods                     | GPU          | OS         | Resolution | Baseline | With Neodymium | Change |
|--------------------------|--------------|------------|------------|----------|----------------|--------|
| OF + FC                  | GTX 1050Ti   | Ubuntu 20  | 854x480    | 480 FPS  |        840 FPS |   +75% |
| OF + FC                  | GTX 1050Ti   | Ubuntu 20  | 1920x1080  | 440 FPS  |        560 FPS |   +27% |
| OF + FC                  | GTX 1050Ti   | Windows 10 | 1920x1080  | 230 FPS  |        270 FPS |   +17% |
| OF + FC                  | Intel HD 630 | Windows 10 | 1920x1080  | 90 FPS   |        130 FPS |   +44% |
| GTNH 2.1.2.3qf + OF + FC | GTX 1050Ti   | Ubuntu 20  | 1920x1080  | 300 FPS  |        390 FPS |   +30% |

#### Legend
* OF: OptiFine HD_U_D6
* FC: FastCraft 1.23
* Nd: Neodymium 0.1

## Usage

You need to install [UniMixins](https://github.com/LegacyModdingMC/UniMixins) first, then just drop neodymium in your mods folder. You can confirm that the mod is working by pressing F3 and looking at the right side of the screen. The number of rendered meshes will be shown, along with the amount of used memory.

If an incompatibility is detected, "**(!) Incompatibilities**" will be shown in the overlay, or the mod may disable itself entirely. The cause of the incompatibility will be printed to the log. You can also view them using the `/neodymium status` command.

In the mod's config file you can find various options you can use to fine-tune the mod to suit your hardware. The config is reloaded when the chunks are reloaded (e.g. when you press F3+A), or immediately upon saving the config file if the hot swap feature is enabled.

### Debug

There are some debug key combinations provided. You have to hold down the *debug prefix key* (F4 by default) while pressing them.

* **F**: switch between Neodymium's renderer and the vanilla chunk renderer. Can be used to compare the difference the mod makes.
    * *(Only usable in creative mode or dev.)*
    * `enableVanillaChunkMeshes` has to be enabled for this to work, otherwise you'll just see the void after switching.
* **V**: toggle whether the world is rendered or not. Can be used to see the theoretical maximum FPS that can be achieved via chunk renderer optimization.
    * *(Only usable in creative mode or dev.)*
* **M**: show the VRAM debugger. The positions of the white pixels shown correspond to the offsets of memory sections allocated in the vertex buffer on the GPU.
* **Left**: reload renderers. Provided for convenience because F3+A makes you strafe left.
* **Right**: toggle renderer update speedup. While this is enabled, chunk updates will be sped up 300x. This kills your FPS but reduces the time you have to wait until all the renderers have loaded.

## How it works

The mod injects callbacks that run when world renderers (16x16x16 sections of the world) change. Right after a world renderer has finished rendering, Neodymium captures the tessellator's state and converts the mesh to its own format. The vanilla chunk renderer is disabled, and a different implementation runs instead. The mod doesn't change how meshes are constructed, only the way they get drawn. It should work with any mod that uses the tessellator to draw blocks.

There are also some additional optimizations:

* Face culling: Faces that aren't facing the camera won't be submitted for rendering. This reduces GPU workload, and will increase the framerate if the GPU was choking. Inspired by a similar optimization in Sodium.

The mod increases memory usage, since the chunk meshes have to be stored somewhere. On Normal render distance in a vanilla world, it uses ~70-150 MB, both in RAM and VRAM.

## Incompatibilities
* The mod performs poorly if **Advanced OpenGL** (occlusion culling) is turned on.
* **OptiFine**'s "Smooth" and "Multi-Core" chunk loading settings are not compatible.
* **OptiFine** shaders **are** now compatible, but may have some minor issues. FalseTweaks allegedly fixes this.
* **CoFHTweaks** is incompatible (https://github.com/makamys/Neodymium/issues/11)
* **Gilded Game Utils**: the regular version is not compatible - use [the fork with threaded lighting removed](https://legacy.curseforge.com/minecraft/mc-mods/gilded-game-utils-fix) instead.
* **Various coremods** may be incompatible with Mixin. Use [Mixingasm](https://github.com/makamys/Mixingasm) to fix this.
* Also see: [issues labeled with `mod incompatibility`](https://github.com/makamys/Neodymium/issues?q=is%3Aissue+is%3Aopen+label%3A%22mod+incompatibility%22).

## Suggested mods
For more 1.7.10 performance mods, refer to [this list](https://gist.github.com/makamys/7cb74cd71d93a4332d2891db2624e17c).

## License

This mod is licensed under version 3 of the GNU Lesser General Public License. See [LICENSE](LICENSE) for additional information.

<a href="https://www.flaticon.com/free-icons/neodymium" title="neodymium icons">Neodymium icons created by Freepik - Flaticon</a>

## Contributing

When running the mod in an IDE, add these program arguments:
```
--tweakClass org.spongepowered.asm.launch.MixinTweaker --mixin neodymium.mixin.json
```

Additionally, the following VM arguments may come in handy:
* `-ea`: enable assertions
* `-XX:HeapDumpPath=MojangTricksIntelDriversForPerformance_javaw.exe_minecraft.exe.heapdump`: You will get terrible performance on an integrated Intel GPU if you don't add these magic words.
