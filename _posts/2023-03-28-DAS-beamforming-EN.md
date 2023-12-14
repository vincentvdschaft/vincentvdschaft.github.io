# Ultrasound Imaging with Delay-And-Sum beamforming

Many people have some idea of how an X-ray is made, but how an ultrasound image is created from sound is less well known. In this blog post, I will show how the most common technique to form an ultrasound image works. This technique is called **Delay-And-Sum beamforming** or DAS for short.

First, let's look at what happens physically when making an echo and what we are actually measuring. To create an image with ultrasound, a *transducer* is used. This is a measuring instrument with a row of sensors that can transmit and receive sound waves. To form an image, a sound wave is first emitted. The sound wave moves through the tissue where it reflects on different structures. This reflection process is super complex, but fortunately there is a simple model for it:

**You can think of the tissue as a space in which a large number of points are located that absorb a sound wave and then emit it in all directions.**

These points are called *scatterers* (because they scatter the sound in all directions). This is only a model and cannot explain all aspects of ultrasound, but it is good enough for us now.

The reflections from the scatterers are captured by the elements of the transducer. The signals received by the elements are combined into an image that shows how strong the reflections are from each location in the image. This process of going from individual sound signals to an image is called *beamforming*.

## Scenario 1 - One Sensor, One Scatterer

To understand the beamforming process, let's first look at a highly simplified scenario: we have a transducer with only 1 sensor element $E_0$ and we scan an image with only 1 scatterer $S_0$ in it.

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-0-just-one-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

In the right image, you can see how the sound wave propagates from $E_0$ and is scattered by $S_0$. In the left graph, you can see the sound signal received by $E_0$. Two things can be inferred from the incoming signal:

1. Because there is a peak in the signal, we know that there must be a scatterer present.
2. Because we know how fast a sound wave moves, we can also deduce from the time between transmission and reception **how far away** the scatterer is located.

<details>
<summary>🧮 Calculation (click to expand)</summary>
<p>
The signal is sent at $t=0$ and the reflection is received at $t_r=15\mu s$. The speed of sound in tissue $c$ is about $1540 m/s$. The total distance $d_{total}$ that the wave has traveled in that time is therefore
$$d_{total}=t_r\cdot c=15\cdot 10^{-6}\cdot 1540=23.1 cm$$
This is the distance of the round trip. So, the scatterer is halfway, which means that the distance between the scatterer and the sensor is $11.55cm$.
</p>
</details>

But we are not only interested in **how far away** the scatterer is. We want to know **where** it is. Unfortunately, we will never find out in this way. We can see this in this alternative scenario: if $S_0$ is located elsewhere but is still the same distance away from the sensor, we measure exactly the same signal! Again, the peak is exactly at $15\mu s$.

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-1-rotated-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

This means that the reflections of all scatterers that lie on a circle arrive simultaneously at $E_0$. So, we cannot distinguish between the different scatterers. Here, you can also see that the measured peak is now much higher. The reflections of all scatterers add up here. (We'll come back to that later.)

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-2-circle-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

To enable us to determine **where** the scatterer is located, we need more information. We can get that information by adding extra sensors at other locations.

## Scenario 2 - More Elements

We now take two sensors instead of one. Again, element $E_0$ sends out a sound wave. The sensors pick up the same signal, but with different delays. Based on the delays, we can, just like before, calculate for element $E_0$ how far away the scatterer is located. We can now also find a ring with possible locations for $E_1$. With both, we can find the exact location of the scatterer: *At the intersection of the two rings.*
(Actually, it's not a ring, but an ellipse. See the calculation block for an explanation.)

<details>
<summary>🧮 Calculation (click to expand)</summary>
<p>

For $E_0$, we can calculate the distance as before: the time at which we receive the peak with sensor $E_0$, $\tau_0$, is the time from $E_0$ to $S_0$ and back. The total distance between $E_0$ and $S_0$ is therefore $$d_{round}=c\cdot \tau_0$$
The round trip is of equal length, so all possible locations for the scatterer are those for which the distance to $E_0$ is equal to $\frac{1}{2}\cdot c \cdot \tau_0$.

For $E_1$, it's different because the pulse is not sent from $E_1$. This means that $d_{round}$ and $d_{back}$ no longer have to be equal. If the peak is received at time $t_{r1}$, we know that the sound has moved the distance from $E_0$ to $P$ in that time (i.e., $d_{round}$) and then the distance from $P$ to $E_1$ (i.e., $d_{back}$). The total distance is therefore $$d_{round}+d_{back}=c\cdot \tau_1$$
The possible locations for the scatterer are therefore the locations for which this equation holds. If you solve this, you will find that the scatterer must be somewhere on an ellipse around $E_0$ and $E_1$.

</p>
</details>

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-3-multiple-sensors-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

## Delay-And-Sum Beamforming

We have now seen that with one sensor, we can determine how far away a scatterer is located and with two or more sensors, we can see where a scatterer is located. The problem is that in practice, we cannot assume that there is only one scatterer. That is literally never the case. In practice, there are many scatterers that are all "talking" at the same time. Let's look at an example that is a little closer to reality: we take $9$ scatterers at random locations and more than two sensors:

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-5-many-sensors-scatterers-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

The signals now look much more chaotic, and without the right image, it is no longer clear which peak belongs to which scatterer. This means that we can no longer determine where the scatterers are located so easily.

To solve this problem, we use *Delay-And-Sum beamforming*. In this technique, we actually do the opposite of what we did before: instead of looking where the peak is in the signal and calculating the distance, we choose a **distance to focus on** and calculate the **corresponding delay**. By focussing on a specific time in the signal of sensor $E_0$, we are focusing on the ellipse of all points from which reflections arrive at that time. The value we measure there is therefore the sum of all scatterers on that ellipse.

We now choose a focus point $P$. We want to know how strong the reflections are from point $P$. We can **focus** on point $P$ by looking at each sensor in the received signal for the moment when the reflections from $P$ arrive precisely at that sensor. (This is the *delay* part.) This way, we find for each sensor $E_i$ a value that tells us how strong the reflections from its ellipse are through $P$. If we add up these values, we get an estimate of how strong the reflection from point $P$ is. (This is the *sum* part.)

<details>
<summary>🧮 Calculation (click to expand)</summary>
<p>
If we have chosen a focus point $P$, we can find the corresponding delays and thus the locations in the received signals as follows.
For each element $E_i$, we can calculate the total distance that the wave had to travel from $E_2$ (the sender) to $P$ and then to $E_i$. The total distance consists of a distance forward $d_{forward}$ and a distance back $d_{back}$. The delay $\tau_i$ that belongs to this is
$$\tau_i=\frac{d_{to}+d_{back}}{c}$$
The calculated delays are shown in the figure below.
</p>
</details>

The image below shows what happens when you look at different locations $P$ in this way and add up all the signals. The dashed lines in the left graphs show which part of the received signal is used to focus on $P$. The bar on the right shows how high the total response is. The contribution of the different sensors to the height of the bar is always separated by a line. To make it clearer which sensor receives a strong signal, the ellipse and a line to the corresponding element appear when the signal received by that element becomes stronger. In the background, you can see the image formed when you let $P$ move over all possible locations and assign the pixel a brightness based on the total response at that location.

<details open>
    <summary>(Click to collapse)</summary>
<img src="{{ 'assets/images/scene-6-focussing-en-dark.gif' | relative_url }}" style="border:none;"/>
<p></p>
</details>

In this figure, we can see several things:

- We see that the response is by far the largest when we are actually focused on a scatterer and that the scatterers are recognizable in the image.
- However, we also see that the output is not always zero when we focus on a point where there is nothing. This is because the focus point $P$ then lies on the same ellipse as one or more of the scatterers.
- If the focus point moves away from a scatterer, we see that the summed response does not immediately drop to zero. This is because the pulse we sent out is not infinitely sharp but has a certain width. We first have to 'walk down the mountain' before we reach zero. This is a limit to how sharp the image can ultimately become.
- Scatterers $S_7$ and $S_8$ have a more smeared out dot in the image than, for example, $S_5$. This is because $S_7$ and $S_8$ are located more to the side, causing the ellipses of the different elements to be more similar, which means their approximate overlap covers a larger region. The resolution of an ultrasound image is therefore not the same everywhere!

Although the quality of the image leaves a lot to be desired, it is clear that the principle works and provides us with a way to create an image from a series of seemingly chaotic signals.


<details>
<summary>Sidenote</summary>
<p>
Some may notice that the world is not 2-dimensional but 3-dimensional. This means that all points equidistant from a sensor form not a circle but a spherical surface. Does this still work? Well no! Sensor elements that lie on a line can distinguish between all locations in a plane in 2D, but not in 3D. To suppress reflections from outside the image plane, ultrasound probes in practice have a grid of elements instead of a line. These same beamforming techniques are used to focus on the desired plane and suppress signals from outside that plane.
</p>
</details>

## Interesting links

- The paper [So you think you can DAS](https://www.sciencedirect.com/science/article/abs/pii/S0041624X20302444) provides a comprehensive but relatively accessible explanation of how to implement DAS-beamforming.