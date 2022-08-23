# World SDK Animator As Code

This is an experimental modified version of [av3-animator-as-code](https://github.com/hai-vr/av3-animator-as-code) for VRChat World SDK.
You can use "Animator As Code" notation to create animators on VRChat world creations.

We call the original "Animator As Code" as `AAac` in this document for short.
In the same way, we call this modified version as `WAac`.

# API differences

- Replace AvatarDescriptor with Animator
    - AAac uses `AvatarDescriptor` as a kind of "binder" that holds multiple animators.
    - WAac replaces it with single `Animator`.
    - You have to use `Animator` instead of `AvatarDescriptor` in `struct AacConfiguration` on [the configuration](README-original.md#declare-an-animator-as-code-aac).
- No "Playable Layers"
    - So, `CreateMainFxLayer()` , `CreateMainGestureLayer()` , ... methods are removed.
    - Instead, WAac introduces `CreateMainLayer()`.
    - Same on Supporting Layers, use `CreateSupportingLayer(string suffix)` instead.
- World SDK doesn't support `VRCAvatarParameterDriver`
    - So, [Driver related methods](README-original.md#avatar-parameter-driver-state-behaviour) on `AacFlState` are removed
- World SDK doesn't support `VRCAnimatorTrackingControl`
    - So, [Tracking related methods](README-original.md#other-state-behaviours) on `AacFlState` are removed.
- World SDK doesn't support `VRCAnimatorLocomotionControl`	
    - So, [Locomotion related methods](README-original.md#other-state-behaviours) on `AacFlState` are removed.



# Notes on modifications

- WAac defines `WORLD_AAC` on top of each `.cs` files if needed.
WAac uses this preprocessor symbol to denote modified parts clearly.
- [Avatars 3.0 parameters](README-original.md#avatars-30-aacav3) (for instance `AacFlFloatParameter GestureLeftWeight`) will work on WAac.
But they are just some of animation parameters. Theere are No special handlings in World SDK.
- [AacVrcAssetLibrary](README-original.md#asset-library-aacvrcassetlibrary) is still available on WAac.
But it uses VRChat Avatar SDK's example assets internally. It won't work without them.
- Currently WAac seem to have no dependancies on VRChat SDK while it's not an intented goal. It's just "as a result", 

## Notes on files in Examples

Replaced with new one:
- AacExampleContainer.controller / WorldAacExampleContainer.controller
- AacExampleFX.controller / WorldAacExampleController.controller
- AnimatorAsCodeExampleScene.unity / WorldAnimatorAsCodeExampleScene.unity

Removed:
- AacExampleMenu.asset
- AacExampleParams.asset
- AacExampleCubemap.exr



# LICENSE

MIT License.

Original works by Haï~ (@vr_hai github.com/hai-vr) are under MIT License. See [LICENSE-original](LICENSE-original) file.

