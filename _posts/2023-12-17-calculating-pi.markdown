---
layout: post
title:	"Calculating Pi numerically: an ode to accuracy"
date:	"2023-12-17"
categories: project
---

A few days ago I was playing around with my old A-Level CASIO classwiz fx-991ex calculator, in particular I was using it for a past paper in preparation for one of my exams. Towards the end of that session I looked over the mark scheme having felt good about that past paper and then saw the precision of my answers vs the mark scheme were completely wrong. Turns out in engineering the standard practise is to round at every stage of the working out and not just when you get a final answer. 

This was certainly frustrating as, in a fit of paranoia, I checked all my answers several times over. Then afterwards I couldn't stop thinking of the need for balance between accuracy and simplicity in engineering, as you would when you've worked all day and caffine has you in hyperfocus. After all we don't want our heads to explode when trying to solve calculations that take into account every small detail when we could just focus on the handful of dominant processes. However at the same time reading about the invention of the junction transistor and seeing that it was only understood by taking into account the ignored minority carrier concentrations, makes me think twice about the line between simplicity and accuracy.

Surely, there does lie a balance between accuracy and simplicity in scientifc modeling and engineering. Well maybe there is, but that is not what this post is about, I just want to throw simplicity to the winds and calculate Pi with as much accuracy as I can handle, so by the end I hope to have done the following.

1. Refreshing my Maths to understand what functions and terms we may need to use.
- History of the computations of Pi
- Using a taylor series for finding trigonometric functions
2. Use C++ to compute Pi numerically
- Convergent arctan(x)
3. Implement the algorithm in SystemVerilog and simulate using Verilator
4. Implement the algorithm on an FPGA

It may well be that the outcome of this project is only that I over simplifying all my calculations after I decide that I hate accuracy, but then again we only need 40 digits of pi to calculate accurately the circumference of the universe.

## Initial calculations

### Archimedes' Method

So the obvious place to start is to ask what Pi is, which is nothing but a scaling constant going from the diameter of a circle to the circumference. This makes intuitive sense since all circles are mathematically similar and so dividing the circumference by the diameter will give you a constant.

$$
\pi = \frac{circumference}{diameter}
$$

So now all we need is a value of the circumference of a circle as a function of the diameter, we can find an expression for Pi. So we begin by drawing a polygon inside a circle, ensuring that polygon was inscribed. So the perimeter of the polygon works as an estimate for the circumference. This is a nice idea that does give a surprising amount of accuracy given that you're physically drawing and measuring a shape. If you decided to draw it with a rule and compass as Archimedes decided, then you might be able to find Pi to 2 decimal places. Not a bad start

If you wanted to use trigonometry to turn this exercise from a drawing and measuring task to something more arithmetic we can find the perimeter as a function of the number of sides on the polygon and a trigonometric function. If you had a polygon inscribed in the circle and a polygon on the outside of that circle you can derive an underestimate and an overestimate for the value of Pi. Doing that and then calculating a value for Pi as a trigonometric value where we take the limit of the number of sides of the polygon, we get the two expressions for the inside and the outside.

$$
\pi = \lim_{N\to\infty} N \cdot \sin\left(\frac{\pi}{N}\right)
$$

$$
\pi = \lim_{N\to\infty} N \cdot \tan\left(\frac{\pi}{N}\right)
$$

So now the problem turns from a basic geometry problem into an evaluating limits problem. However we can't continue with this as we would need to calculate small values of $$sin(x)$$ and $$tan(x)$$, then multiply it by a large value N. But we have no idea what $$\pi$$ is and so we cannot even begin to calculate what $$sin(\frac{\pi}{N})$$ is as the argument could never be found accurately. Kind of a Mathematical chicken and egg story. I am telling you this mostly to show that this approach is only useful for drawing and measuring, but it does point to another method of calculating Pi.

### Taylor series polynomial

The fundamental relation between trigonometric functions and radians is really what brings us closer to calculating Pi numerically and let's us move away from the drawing circles. So I started out with was the taylor series of sine and cosine. As the taylor series converges directly to the trigonometric function, we could instead make a polynomial out of the function and solve the polynomial to give Pi.

$$
\sin(x) = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \frac{x^7}{7!} + \frac{x^9}{9!} - \ldots = \sum_{n=0}^{\infty} \frac{(-1)^n \cdot x^{2n+1}}{(2n+1)!}
$$

$$
\cos(x) = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \frac{x^6}{6!} + \frac{x^8}{8!} - \ldots = \sum_{n=0}^{\infty} \frac{(-1)^n \cdot x^{2n}}{(2n)!}

$$

So we can make a polynomial out of this to solve, by setting x to be $$\frac{\pi}{2}$$ in the case of sine and either $$\frac{\pi}{2}$$ or $$\pi$$ in the case of cosine, we can then rearrange to get the following polynomials.

$$
\cos(\frac{\pi}{2}) = 0
$$

$$
1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \frac{x^6}{6!} + \frac{x^8}{8!} - \ldots = 0
$$

$$
\sin(\frac{\pi}{2}) = 1
$$

$$
-1 + x - \frac{x^3}{3!} + \frac{x^5}{5!} - \frac{x^7}{7!} + \frac{x^9}{9!} - \ldots = 0
$$

So then solving for x provides us with an estimate for $$\frac{\pi}{2}$$, but again this is not the most sensible value as we have to solve a polynomial that gets increasingly complicated. As before, this method does point towards a slightly better method that has been used since the time of Newton to calculate Pi.

### Direct sum using Taylor series

If instead we try and find the Taylor series for the inverse trigonometric functions, we can then find a value of $$\pi$$ through a direct sum of terms. For example, we can take the series for $$\arctan(x)$$ as an example.

$$
\arctan(x) = \int_{0}^{x} \frac{1}{t^2 + 1} \, dt = x - \frac{x^3}{3} + \frac{x^5}{5} - \frac{x^7}{7} + \frac{x^9}{9} - \ldots = \sum_{n=0}^{\infty} \frac{(-1)^n \cdot x^{2n+1}}{2n+1}
$$

And we get all of this from some implicit differentiation of $$\arctan(x)$$, then we can sub in $$x=1$$ which will give us a realtively simple series that converges to $$\frac{\pi}{4}$$. This formula is only bad in that it takes hundreds of terms to even get Pi to 2 digit accuracy (In reality the following code was used to test the rate of convergence). Instead we can (and will use) the identity 

$$
\frac{\pi}{4} =  4\arctan(\frac{1}{5}) - \arctan(\frac{1}{239})
$$ 

to force the series to converge quickly. The only issue now is memory, as this is an exercise I would like to take right down to the hardware level and not just good C++ but we'll relax that for now.

### Conclusion

## C++

### Machin

## SystemVerilog

### Machin

### BBP Formula

## FPGA implementation
