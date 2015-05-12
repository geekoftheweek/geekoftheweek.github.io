---
layout: post
title:  "Unity 5 Roadshow"
date:   2015-05-04 20:28:00
---
Last Saturday I attended the Columbus, Ohio session of the Unity 5 Roadshow,
hosted by Microsoft and presented by Mike Geig.  I had a great time, and
decided to write down some of the things I learned while the memories are
fresh.

## About Me

To give you some context, I'm a programmer (and a "wannabe" artist) who's used
Unity almost daily for about ten months now.  I'm no pro, but I can get stuff
done.  I've made a couple of small 2D games, some pretty complicated 3D
visualization/simulation apps, and a few VR experiences all using Unity.  Going
into the Roadshow, I figured most stuff wasn't going to be new to me, but I
hoped to fill in some knowledge gaps, pick up a some new tricks, and observe
some best practices.  I definitely learned some new things, mostly centered
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

Here's the painfully simple code:

{% highlight csharp %}
using UnityEngine;
using System.Collections;

public class LightSpin : MonoBehaviour {

  public Vector3 axis;
  public float speed;

  void Update () {
    transform.Rotate(axis, speed * Time.deltaTime);
  }
}
{% endhighlight %}

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
realistic glow from the panels, without adding any traditional lights to the
scene.  In Unity 4, I had quickly learned not to fiddle with the
lights unless absolutely necessary -- baking took too long and was too fickle
to get right.  Going from that to Unity 5's system was a refreshing
improvement.  Somewhere in here we added a FPS character controller so we could
more easily navigate the scene and check things out.

At this point, I decided to script some random flicker into my lighting panels,
to put the realtime GI system through its paces.  I wasn't sure how the
emission value was exposed by the Unity 5 standard shader, so I searched on the
web.  Turns out, there are two separate values to set, one that controls the
color of the object as you look directly at it, and another one that controls
its contributions to the global illumination of the scene.  The final code was:

{% highlight csharp %}
using UnityEngine;
using System.Collections;

public class EmissionFlicker : MonoBehaviour {

  public Color emissionColor;
  public float minChangeTime;
  public float maxChangeTime;
  public float maxEmission;

  private float _nextChange;

  void Update () {
    if (Time.time > _nextChange) {
      _nextChange = Time.time + Random.Range(minChangeTime, maxChangeTime);
      SetEmission(Random.Range(0f, 1f));
    }
  }

  void SetEmission (float intensity) {
    if (intensity < 0.5f) {
      intensity = .25f;
    } else {
      intensity = 1f;
    }
    Renderer renderer = GetComponent<Renderer>();
    Material mat = renderer.material;

    // Set the material's emission
    Color newDiffuseColor = emissionColor * intensity;
    mat.SetColor("_EmissionColor", newDiffuseColor);

    // Set the material's GI contribution
    Color newEmissiveColor = emissionColor * intensity * maxEmission;
    DynamicGI.SetEmissive(renderer, newEmissiveColor);
  }
}
{% endhighlight %}

I was super happy with how it looked!

![Flickering light panel animated gif](/images/unity-5-roadshow-flicker1.gif)

## Mecanim

Next we learned how to control animations.  First, we added a bazooka model as
a child of our FPS character controller.  Next, we created an
AnimatorController asset, added an Animator component to the bazooka, and
dragged the AnimatorController into the Animator.  Once these were connected,
we opened the AnimatorController and started wiring up Mecanim states.  This
was as easy as dragging animation clips into the controller and then
right-clicking to create transitions between them.  The default behavior is for
the state machine to transition to the next state when the current animation
finishes playing, but we set a "Shoot" trigger to force the transition instead.

Mike had us iterate on the state machine a few times, starting simple and then
demonstrating some new features of Unity 5.  Unfortunately I can't remember all
of those in-between iterations, but the final base state machine looked like
this:

![Shooting animation state machine](/images/unity-5-roadshow-state-machine1.png)

Looking at this diagram, there are some important things that aren't visible.
For one, there's a `Shoot` parameter of type `Trigger`, which serves as the
condition for transitioning from `Idle` to `Shoot` in the state machine.  Also,
the `Shoot` sub-state machine itself has a StateMachineBehavior attached,
called RandomShootingAnimation.  This script sets the `ShootIndex` parameter of
the Animator Controller to 0 or 1, randomly.  The sub-state machine then uses
that value, as we'll see shortly.

Descending into the `Shoot` sub-state machine, we see this:

![Shooting animation state machine](/images/unity-5-roadshow-state-machine2.png)

The only hidden information here is that the condition for transitioning from
`Entry` to `Shoot2` is that the `ShootIndex` parameter is 1.  In other words,
which shooting animation plays is controlled by the value of `ShootIndex`, and
the value of `ShootIndex` is controlled by the StateMachineBehavior that
randomly sets its value when the `Shoot` sub-state machine is entered.  I
suspect I haven't done a great job of explaining this, and that this will all
either make sense or it won't.  Frankly, that's kind of how it was presented at
the Roadshow too, since there wasn't exactly time for a course on state
machines.  If this didn't make sense and you're somehow still reading, suffice
it to say that Unity 5's new support for adding behaviors to states and to
sub-state machines allows some fun scripting possibilities that were tricky or
tedious previously.

## Reflection Probes

Once we had an animated bazooka, we made it shiny and then played with
reflection probes.  This is probably the topic where I experimented the most
and learned the most.  As per instructions, we positioned probes through the
hallway at about eye level.  After positioning each probe, we adjusted its
bounding box to *just barely* encompass all of the hallway geometry, plus
extend a little farther down the hallway to overlap with the next probe's
bounding box.  Mike warned that Unity can blend between two reflection probes,
but not three, so try to avoid areas where more than two probes' regions
overlap.  After checking `Box Projection`, reflections looked accurate while
walking down the hallway.

Here's where my clever flickering light script suddenly started looking a _lot_
less clever.  By default, the reflection probes' images are baked before the
game starts playing.  So, even with both light panels shut off, there were
still bright green and red reflections plainly visible on reflective surfaces.
Intrigued, I started trying to fix the problem via trial-and-error fiddling
with the reflection probes.  I ended up working through our lunch break,
switching the `Type` from `Baked` to `Realtime`, and tried all sorts of
combinations of `Refresh Mode` and `Time Slicing`.

It turns out that refreshing the reflection probes' cube maps is a really
expensive operation, and Unity provides the refresh modes and time slicing
options to manage that expense.  One typical approach is to amortize the cost
of updating the maps over many frames, by updating as many of the cubemap faces
as possible in one frame, then picking up where you left off in the next frame.
That's what the `Time Slicing` choices do.  `Individual faces` spreads the
cubemap update over fourteen frames.  `All faces at once` does the update over
nine frames.  `No time slicing`, on the other hand, means that the entire
cubemap will be recalculated each frame, and the frame drawing will just have
to sit and wait until the entire cube map is good and ready.  This affects
performance as badly as you might imagine, and demolished performance on my
scene, knocking it from 300+ frames per second down to ~44.

There's also the option to detach the reflection probe update process from
frame updates completely, and just control it via script.  This was what I
needed -- but I still never got it working properly.  The idea was to trigger a
render probe update after every change in the emissive intensity of a light
panel.  It worked mostly, and performance was still really good, but it looked
like the cubemap was always one frame behind.  My best guess is that it's
something to do with the timing of when the script asks the reflection probes
to refresh, versus when the necessary rendering data is available, or
something.  I'm still not satisfied, but a one-frame delay was much better than
any of the time-sliced options (all of which left the scene looking like some
kind of trippy yuletide nightclub, as illustrated below).

![Out-of-sync reflection map animated gif](/images/unity-5-roadshow-flicker2.gif)

## Navigation and Explosions

From this point on, things were kind of rushed.  It's understandable, since the
topics were scripting-heavy and the Roadshow didn't require any prior scripting
knowledge from attendees.  It was a lot of wiring together pre-made stuff.

Step 1: Make a robot that seeks & shoots the player.  To do this, we dragged
the Drone model into the scene, and added a bunch of Components to it: a Sphere
Collider, a Nav Mesh Agent, and two custom scripts (FindTarget and
BulletImpact).  Then, we opened the Navigation window, adjusted the agent
radius and height to match the model, and clicked Bake.

The BulletImpact script had an Explosion GameObject variable, which segued into
Step 2: Make the robot blow up when shot.  Mike provided a custom explosion
prefab, but I just imported the Particle Systems standard asset package and
used the Explosion effect in there because it's prettier.  Anyway, that was
pretty much it for exploding robots.  I clicked play, and the scripts did their
thing -- the robot started seeking and shooting the player.  When I shot the
robot, it exploded in a satisfying manner, and there was much rejoicing in the
land.

Of course, what's the fun of an explosion if you only ever see one?  I put the
following code on an empty GameObject, to spawn an infinite stream of drones at
regular intervals:

{% highlight csharp %}
using UnityEngine;
using System.Collections;

public class SpawnDrones : MonoBehaviour {

  public GameObject dronePrefab;
  public GameObject navTarget;
  public float spawnTime;
  private float _nextSpawn;

  void Update () {
    if (Time.time > _nextSpawn) {
      _nextSpawn = Time.time + spawnTime;
      GameObject drone = (GameObject)Instantiate(dronePrefab, transform.position, transform.rotation);
      drone.GetComponent<FindTarget>().navTarget = navTarget;
    }
  }
}
{% endhighlight %}

## Audio

The last stop on our whirlwind tour was the completely overhauled audio system.
We added an Audio Source component to the explosion prefab (with a boom noise),
another one to the bazooka (with a *pew-pew* noise), and a third one to an
empty GameObject called "Background Music," with a music loop.  We opened up
the Audio Mixer window, created some Groups (Explosion, Weapon, and
Background), and then set each Audio Source component's Output variable to
point to the appropriate Group (*pew-pew* sound --> weapon group, etc).

Then the fun began.  We selected the Background group, and added an Effect
called "Ducking."  Then we selected the Weapon group and added an Effect called
"Send," with the target being the Background group's ducking effect.  After
fiddling with the parameters a bit, this meant that the background music would
briefly get muted or "ducked" when the bazooka fired.  Super cool, and not
something I would have ever thought to add to a game, let alone figured out
*how* to add.  Mike then led the rest of the group through the steps to make an
explosion duck all audio for a long time, while simultaneously playing an "ear
ringing" sound as if a flashbang had temporarily deafened the player.  I
skipped out on this, since the "ear ringing" sound was completely out of my
hearing range, making it pretty impossible for me to tweak everything properly.

Anyway, the major takeaway in this area is that the new Audio system is
incredibly powerful, and very straightforward to use.  I definitely plan to
invest some time in learning how to use it.  I've already got some interesting
gameplay ideas that would have previously required some quality time in audio
editing software, but will now require less than five minutes of clicking
around in Unity.  I am super impressed.

## Whew

Again, I really enjoyed the Unity 5 Roadshow.  Mike especially was a class act.
He was thoroughly prepared and did a great job of keeping things technically
interesting, entertaining, and moving along at an appropriate pace.  The
material was aimed at someone less experienced in Unity than myself, but I
learned some new things and definitely left more motivated to make games.
