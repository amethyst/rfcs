# Unified 2D Render

# Table of Contents

- [Tracking Issues](#tracking-issue)
  - [Amethyst Community Forum Discussion](#forum-discussion)
- [Motivation]
- [Guide Level Explanation](#guide-level-explanation)
- [Reference Level Explanation](#reference-level-explanation)
- [Drawbacks]
- [Rationale and Alternatives](#rationale-and-alternatives)
- [Prior Art](#prior-art)
- [Unresolved Questions](#unresolved-questions)

# Basic Info
[basic]: #basic-info

- Feature Name: unified_2d_render
- Start Date: 2018-11-13
- RFC PR: -
- [Tracking Issue](#tracking-issue): -
- [Forum Thread](#forum-discussion): -


# Summary
[summary]: #summary

This feature allows rendering the following 2D elements using shared code and features:
* Ui (Text + Image)
* Sprite
* 2D Mesh with Texture

The big goal is to move as much code as possible out of the rendering `Pass`es and put them as `Component`s and `System`s in Amethyst's ECS context.

Keyword: Consistency


# Motivation
[motivation]: #motivation
Why are we doing this? What use cases does it support? What is the expected outcome?

Currently, only `SpritePass` supports batching. Each feature we add must be duplicated between the DrawSprite, DrawFlat and DrawUi passes. When you think about it, all we are doing in the end is drawing a 2D texture on screen.

This RFC is about combining the three approaches to draw 2D textures on screen into a single one `Pass`, where possible.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation
<details>

<summary>Non-technical overview and reasoning for it.</summary>
Explain the proposal as if it was already included in the language and you were teaching it to another Amethyst programmer. That generally means:

### Introducing new named concepts.

This RFC introduces a lot of new components. Let's go over each of them quickly, and then we will look at some usage examples.

![Concept Schema](https://i.imgur.com/y2iq2YG.png)

#### Transform2D
This component is the same as its 3D counterpart. It is used to indicate the location and pose of an entity. Here's the notable differences.

`Transform` will be renamed to `Transform3D` to avoid confusion.

`Transform3D` has a rotation stored as a `Quaternion`. `Transform2D` has a simple f32 (float), since it only makes sense to rotate on the Z axis in 2D.

`Transform2D` has an added `dimension` `Vector2` field which indicates the size of the entity.

`Transform3D` has a `scale` `Vector3` field while `Transform2D` has a `Vector2` for it. It is used to multiply the `dimension` field. (taking into account the parent's scale).


#### ScreenSpace

Indicates that the entity's `Transform2D` should be calculated as a position on the screen instead of as a position inside of the world.

#### Overflow

Indicates whether to show the overflowed content if the child's dimensions exceeds the parent's dimensions.

Mostly used with UI.

#### MaterialPbm

Rename the `Material` Component to `MaterialPbm`.

The albedo field will be moved to a `Handle<Texture>` for 3d entities.

#### UV2D

Contains the uv data for 2d sprites. Contains only 4 `Vector2` positions (one for each corner of a square).

This will be used mostly with `SpriteRender`.

#### Material Effects

For each effect that can be applied to a 2d texture, there will be one associated `Component`.

Example effects:
* Blur
* Glow
* Tint
* Pixelate


### Explaining the feature largely in terms of examples.

This isn't really a feature, but more of a refactor.

If you are using Prefabs for the UI, and `SpriteRender` for your sprites, the only thing you will need to change is to use `Transform2D` instead of `Transform`.

If you are manually creating UI entities without using the builders however, the way you do it will change a bit.

Here's how to manually create a UI text after this change:
```rs
world.create_entity()
    .with(Transform2D::default())
    .with(UiText::new(normal parameters....))
    .with(ScreenSpace) // Optional. If not present, your text will be positioned in the world.
    .with(Overflow::none()) // Hide overflow.
    .build();
```

Concerning the sprites, the only change to your existing code is that you will change `Transform` for `Transform2D`.

### Explaining how Amethyst developers should *think* about the feature, and how it should impact the way they use Amethyst. It should explain the impact as concretely as possible.

This change won't really impact your existing code.

What will change for you is that you will now be able to make text that is following a sprite in the world.

You can do that by having the text entity be a child of the sprite entity, and by not adding the `ScreenSpace` component to the text entity.

You will also be able to apply any effect that would normally be reserved for sprites on the ui entity. (tint, blur, animations, rotations, etc...)

Here's an example of a text following a sprite:
```rs
let player = world.create_entity()
    .with(Transform2D::default())
    .with(SpriteRender::new(your data))
    .build();
    
let player_name = world.create_entity()
    .with(Transform2D::from_translation(Vector3::new(0.0, 10.0, 1.0))
    .with(UiText::new("player name", font, font_size, etc...))
    .with(Parent::new(player))
    .build();
    
```

### If applicable, provide migration guidance.

You will need to change all the `Transform` components that are placed on 2d entities for `Transform2D` components.
The same is true with `Transform3D` for 3d entities.

You will need to add `ScreenSpace` on manually created UI entities.

</details>

# Reference-Level Explanation
[reference-level-explanation]: #reference-level-explanation
<details>
<summary>The technical details and design of the RFC.</summary>
This is the technical portion of the RFC. Explain the design in sufficient detail that:

## Data flow

This RFC divides the data flow in three sections:
* User Data (Components)
* Intermediate Processing (Systems)
* Rendering (Pass)

The User Data section is meant to be user-friendly data. For example, the `SpriteRender` type which makes is easy to change which sprite should be rendered.

The Intermediate Processing section converts the user-friendly data into a reusable data format (referred to as 'Reusable Data'.

The Rendering passes then take this reusable data and render it to the screen or to a texture.

We will take a forward approach to discuss those sections. That is, we will start with the user data and go all the way into the `Pass`' logic.

### User Data

The user data closely resembles what is currently available.

I will present it in the three ways it can be used.

In all those cases, assume that `Transform2D` is used instead of `Transform` and/or `UiTransform`.
Here is the type of `Transform2D` for reference:

```rs
pub struct Transform2D {
    translation: Vector3,
    rotation: f32,
    dimension: Vector2,
    scale: Vector2, // Used to multiply your own dimension AND the ones of the child entities.
}
```

#### Ui

You have builders like UiButton that create all the reusable data for you.
Those already exist and will only be tweaked to accommodate for the changes.

#### Sprite

Using sprites isn't different at all from what is already integrated in the engine.

#### Mesh + Texture

Previously, you would use a 3d plane rotated towards the screen to render.
You would slap a texture on it with uv coordinates and that would effectively be your sprite.

Now you will simply need to attach a `Handle<Texture>`, a `Transform2D` and optionally a `UV2D` component on your entity to have the same result.

If you don't attach a `UV2D` component, the `Pass` will assume the default uv coordinates.

This is effectively the 'Reusable data'. It is visible to the user, but they will only rarely touch it manually because of the widespread use of Sprite.

### Intermediate Processing

Converts user data into reusable data.

#### Ui

UiImage will use FlagStorage.
When a change is detected by a System, it will add or change a `Handle<Texture>` component on the entity that is pointing to the same location UiImage is pointing to.

When the UiImage component is removed, the `Handle<Texture>` is also removed.

UiImage will eventually be removed to use directly `Handle<Texture>` instead after the UI refactor.

UiText works the same way, but will not be removed in the UI refactor.
It will be rasterized to a `Texture` by a `System`. See [Drawbacks] for more information.

#### Sprite

The same way as in the `Ui` section, a System checks changes on the `SpriteRender` component and associates the correct `Handle<Texture>` on the entity.

The same System uses the sprite_id field of `SpriteRender` and the `AssetStorage<SpriteSheet>` to associate the proper uv coordinates from the `SpriteSheet` onto the `UV2D` component.

#### Mesh + Texture

No transformation has to be done. The data is already stored in the reusable format.

### Rendering

Now, if you followed through all of this, congratulations! This is here that you will finally see why those changes are useful!

Currently, there are three passes for the three use cases I mentioned earlier.
* DrawSprite
* DrawUi
* DrawFlat (when using 2D meshes)

What do they all have in common?
* Draw rectangle meshes
* Slaps a `Texture` over the mesh using `uv coordinates`
* Batches draw call according to current `Texture`. (only DrawSprite is currently, but they should all be doing that)
* Apply various rendering `Effect`s.
* 

And what Reusable data do we have right now?
* Transform2D (position, rotation, dimension (size of the mesh!)) + ScreenSpace
* Handle<Texture>
* UV2D

Hey, we got everything we need to draw 2d textures!

Note: This RFC assumes that only rectangular meshes are sufficient to cover all use cases, considering transparency is enabled.

You probably noticed that I didn't mention the `Handle<Mesh>` Component. Since all 2d elements will be drawn using a rectangular mesh, we can have a single square mesh integrated directly into the pass. This way, we just need to apply the `Transform2D` position, rotation and dimension to scale it inside of the shader (on the gpu).

Also of note: ScreenSpace elements are always drawn **after** the world space elements.
That way, ScreenSpace elements are always on top.

But what about effects?
We can make one `Component` for each effect.

For example:
```rs
#[derive(Component)]
pub struct Tint {
    pub tint: Color::Rgba(1.0, 1.0, 1.0, 0.5),
}
```

Each of those `Effect`s could then be processed by the pipeline.

</details>

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

This limits the quantity of "hacky" things we can do to improve performance.

For example, if you would want to draw 100_000 circles, it may be possible that the performance would be better when using a circular mesh than when using a square mesh and having transparency on the texture.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions
<details>
<summary>Additional questions to consider</summary>

* How much more performant is SVG rasterization on the gpu vs on the cpu? If its a lot, we need a dedicated Pass for Svg rendering. If its not, we can rasterize it on the cpu, and use the shared `Pass` described by this RFC.
* The same question is true with text rasterization.

</details>

<br/>
<br/>
<br/>

Copyright 2018 Amethyst Developers