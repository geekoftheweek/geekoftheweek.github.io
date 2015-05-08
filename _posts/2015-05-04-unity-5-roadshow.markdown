---
layout: post
title:  "Unity 5 Roadshow"
date:   2015-05-04 20:28:00
categories: unity games
---
On Saturday I attended the Columbus, Ohio session of the Unity 5 Roadshow,
hosted by Microsoft and presented by Mike Geig.  I had a great time, and wanted
to capture some of the things I learned before the memories fade.

## About Me

To give you some context, I'm a programmer (and a "wannabe" artist) who's used
Unity almost daily for about ten months now.  I'm no pro, but I can get stuff
done.  I've made a couple of small 2D games, some pretty complicated 3D
visualization/simulation apps, and a few VR experiences all using Unity.  Going
into it, I figured most of the information wasn't going to be new to me, but
that I'd fill in some gaps in my knowledge and hopefully pick up a few tricks
and best practices.  I definitely learned some new things, mostly centered
around realtime global illumination and audio.  More on that in a bit.

## Arranging a Scene

We were all given some example sci-fi themed First-Person Shooter assets: a
straight hallway segment, a corner hallway segment, and a balcony.  Mike
quickly walked us through the basics of the Unity GUI, and then showed us how
to snap objects together by vertices, by holding the `V` key, then clicking and
dragging a vertex on one object up to a vertex on another object.  Somehow I
had never stumbled across this shortcut on my own, so it was fun to learn a new
trick right off the bat.

## Lighting and Global Illumination

After we had assembled our own hallway with a corner or two, we learned about
lighting.  Mike walked us through disabling the default directional light, to
showcase how the skybox contributes to the ambient light in the scene.  Then,
we set the hallway segments to be static, and watched Unity calculate global
illumination for the scene.  Mike took a few minutes to highlight the
differences between the light baking systems in Unity 4 versus Unity 5.  He
explained that Unity 4 baked the light into textures, so shadows could be
overlaid on top of the color information for the objects in the scene.  On the
other hand, Unity 5 baked information about the objects that would allow it to
quickly figure out the lighting later.  This is what enables the 'real-time'
aspect of Unity 5's real-time global illumination.

I wanted to watch the real-time changing lighting in action, so I made a quick
day/night cycle.  First I opened up the Lighting window and dragged the scene's
default Directional Light into the slot for the procedural skybox's Sun.  Then
I made a script that slowly spun the Directional Light on the x-axis.  Voila.
Day/night cycle, with beautiful global illumination.

![Day/night cycle animated gif](/images/unity-5-roadshow-daynight.gif)

## Materials and Emission

Moving on, Mike demonstrated physically-based rendering by showing us how to
tweak the parameters on the already-defined materials in the scene -- namely
the walls, ceiling, and floor of the hallway segments.  He explained that if
you're going for realism the `Metallic` slider should always be set to either 0
or 1 -- real materials are typically metallic or non-metallic, not somewhere
in-between.  For smoothness, however, he suggested *never* setting the value to
0 or 1, but always some value in the middle.  

We then experimented with light-emitting materials, by creating two ceiling
panels in the corner hallway segment and setting their materials to emissive --
one red, one green.  We cranked their emission values up to around 3, and when
the global illumination calculations finished, we could see an impressively
realistic glow from the panels, with out adding any traditional lights to the
scene.  In Unity 4, I had quickly learned not to fiddle with the
lights unless absolutely necessary -- baking took too long and was too fickle
to get right.  Going from that to Unity 5's system was a refreshing
improvement.

At this point, I decided to script some random flicker into my lighting panels,
to put the realtime GI system through its paces.  I wasn't sure how the
emission value was exposed by the Unity 5 standard shader, so I searched on the
web.  Turns out, there are two separate values to set, one that controls the
color of the object as you look directly at it, and another one that controls
its contributions to the global illumination of the scene.  The final code was:

{% highlight csharp %}
// TODO: Add code!
void Update() {
  if (Time.time > _nextChange) {
    _nextChange = Time.time + Random.Range(minChangeTime, maxChangeTime);
    SetEmission(Random.Range(0, 1f));
  }
}

void SetEmission (float intensity) {
  if (intensity < 0.5f) {
    intensity = 0f;
  } else {
    intensity = 1f;
  }
  Renderer renderer = GetComponent<Renderer>();
  Material mat = renderer.material;

  // Set the material's emissive color
  Color newMaterialColor = emissionColor * intensity;
  mat.SetColor("_EmissionColor", newMaterialColor);

  // Set the material's GI contribution
  Color newGIColor = emissionColor * intensity * maxEmission;
  DynamicGI.SetEmissive(renderer, newGIColor);
}
{% endhighlight %}

I was super happy with how it looked!

![Flickering light panel animated gif](/images/unity-5-roadshow-flicker1.gif)

## Mecanim and Reflection Probes

TODO: Bazooka animations

Next, Mike taught us about reflection probes.  We positioned probes at about
eye level, with bounding boxes that *just* encompassed all of the hallway
geometry, plus a little extra to overlap with the next probe's bounding box.
Mike warned that Unity can blend between two reflection probes, but not
three, so try to avoid areas where more than two probes' regions overlap.

TODO: more about reflection probes

## Navigation and AI

TODO: Enemy drone and navigation

## Explosions and Audio

TODO: Audio groups and ducking
