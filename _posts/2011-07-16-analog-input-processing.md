---
layout: post
title:  "Analog Input Processing"
date:   2011-07-16
categories: blog
tags: "#AltDevBlogADay"
mathjax: true
---

> This post originally appeared on #AltDevBlogADay on June 16, 2011

Analog input devices are becoming more and more prevalent in the gaming landscape. We've gone from just using gamepads and joysticks, to accelerometer data, swipe gestures, and pretending we have gamepads by using touch screens. Analog controls are all about conveying a feel to the user in order to connect them to their avatar, and deepen the play-experience. There are several tricks for this that can be employed with fantastic results to deliver controls that feel great.

Let \\(μ\\) be a normalized input value from an analog device. Many analog devices will provide input as an integer. For the purposes of this discussion, it is assumed that all input values have been normalized.

# Deadzone

Analog input is, by its nature, noisy. It is critical to ensure that the values you are getting from the device are the result of intentional user manipulation, and not simply background noise. The 'deadzone' is the range of input values which can be generated without any intentional user input.

Let \\(λ\\) indicate a constant, below which input value for the analog device are indistinguishable from background noise. An input value should be rejected if \\(\|μ\| \gt λ\\)

The next key is what you do after a value exceeds λ. Pick up a console adventure game, and very slowly move the control stick until your character begins moving. Does your character unnaturally lurch into motion once the deadzone has been exceeded? If not, well good for you. Many games suffer from this problem, though, and it is because input values have not been re-scaled after the deadzone. To put it another way, your character goes instantly from having a movement value of \\(0\\), to \\(λ\\) causing them to lurch into movement. In order to smooth this movement, we compute \\(μ′\\).

$$ μ′=sgn(μ)*(|μ| — λ)/(1 — λ) $$

We now have a linear, continuous range of normalized input values, without any 'lurching'.

# Non-Linear Inputs Have More Fun

Having linear input values is good; it puts you well above a good number of released games, but we can do better. Consider the steering for a racing game, controlled by an accelerometer device. When the user wants to make a small turn, they will turn the device only slightly; when they want to make a more dramatic turn, they will drastically turn the device. We want to prevent over-correction, because it makes the user feel as though they do not have good control over their experience.

$$ μ″ = ƒ(|μ′|) * sgn(μ) $$

where \\( ƒ(x) \\) is continuous over the interval \\( [0, 1] \\) and \\( ƒ(0) = 0 \\) and \\( ƒ(1) = 1 \\).

Analog input must be tuned to the purpose of the input. In Marble Blast Ultra I observed that, I wanted very fine control over the input with small stick-movements, but wanted drastic changes when the control stick was near its extents. To do this, I used \\(ƒ(x)=x^{e}\\) [(plot)](https://www.wolframalpha.com/input/?i=plot+%7C+x%5Ee+%7C+x+%3D+0++to++1) on the movement input, and \\(ƒ(x)=x^{3.2}\\) [(plot)](https://www.wolframalpha.com/input/?i=plot+%7C+x%5E3.2+%7C+x+%3D+0++to++1) for the camera. I found these values by playing the game, tweeking, and playing some more. It was easy to conceptualize what I was trying to do by looking at a graph of the function, but as this is about the feel of the input, it is critical that enough time is devoted to playing the game with the altered input.

I strongly suggest that you grab a graphing calculator, and mess around with some functions. You'll probably settle on some variation on \\(e^{x}\\) or \\(x^{C}\\). Your input needs may call for something like

$$ƒ(x)=max(0, ln(x)+1)$$

[(plot)](https://www.wolframalpha.com/input?i=plot+%7C+max%280%2C+ln%28x%29%2B1%29+%7C+x+%3D+0++to++1)

which I have yet to use, but think it could work fantastically for an analog stick input of some sort.
The ultimate goal of processing analog values is to make the input more visceral, than the raw values reported by the analog device.

# Squaring the Circle

Take a look at the analog sticks on a game controller, or better yet: take apart a game controller. What you'll find is a the analog stick seated in a square component, which is made into a nice circle by the plastic shell of the controller. You should also observe that the extents of the square component are only reachable by the control stick in four precise locations. That means that, under normal circumstances, you will only see \\(\|μ\| = 1\\) when the stick is either at the exact top, bottom, left, or right.

Why do you care? Well if the magnitude of movement is based on the analog input, you will not get the full range of values. Further, it means that players can obtain an unfair advantage in your game by taking the shell off their controller, and cutting the circular housing for the control stick into a square, allowing them to reach magnitudes that other players can not. We learned this the hard way, in Marble Blast Ultra.

I had to dig out some old code, but this was our solution:

```cpp
void pushToSquare(Vector2 &dir)
{
  float absX = fabs(dir.x);
  float absY = fabs(dir.y);
  float dirLen = min(dir.len() * 1.25f, 1.0f);
  float scale = max(0.01f, absX > absY ? absX : absY);
  dir *= dirLen / scale;
}
```

# Going Further

Analog input is a representation of actions a player wants to perform in the game. Instead of simply slaving your code to the values sampled by the hardware, I encourage you to use those values to interpret the desires of the user to create a better play-experience.

I recently worked on an Android racing game and did some experimentation with trying to detect when a user was trying to swerve to avoid an obstacle. To do this, I kept a sliding window of input values, μ, on the relevant accelerometer axis. I used this to compute several averages over segments of time, in essence, doing a kind of piecewise differentiation on the input. I then intended to experiment with adjusting the tire-grip to try and assist the user in swerving.

While I ended up running out of time for this experiment, it's something I'd like to mess with a bit more in the future. I think there is a lot of ground to be gained by treating input as an expression of the users desires, instead of a direct input to game values. It is important to not cross over into creating non-deterministic situations for the user; where they feel as though the input they are generating does something different each time they perform an action.

# Consider the Surface Scratched

I haven't seen these issues addressed in any other places, and I don't feel as though this is really a complete treatment. I developed the techniques I shared through trial and error, with a dash of OCD. It is my hope that I have introduced the concept that interpreting and processing analog input is a viable option, as opposed to simply feeding it directly into your kinematics. Analog input processing is an opportunity to make the user feel connected to the game-experience by making their manipulations of game objects feel more natural. Help them perform fine-grained movements when needed, and react more drastically to dramatic inputs.
