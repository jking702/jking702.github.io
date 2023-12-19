---
layout: post
title:	"Calculating pi numerically"
date:	"2023-12-17"
categories: project
---

Changes

I was watching the film [Pi][pi-imdb] the other night, and while it gives an interesting take of what being aggressively obsessed with a problem can do to a person, it felt like it was lacking in any genuine Mathematical substance. This is a bit of a harsh take given that the film was made on a budget of $60,000 and bucked the trend by having a Mathematical protagonist.

So in order to cleanse myself of the Hollywood view of Mathematics, and get back into some digital hardware, I would like to calculate the constant Pi. The exact objective is to calculate Pi to at least 20 decimal places (which is about double the precision of my high school calculator). This also gives an interesting look into how numbers are manipulated in digital for calculations.

## Initial calculations

So the obvious enough thing to do is to find a value for pi, knowing that pi is nothing more than the circumference of a circle divided by it's diameter.
```math
\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
```

We can begin finding the circumference by drawing a polygon inside a circle. Then if we can find the perimeter of the polygon we have a estimate for the circumference. If we were feeling motivated enough we might want to draw a polygon inside and outside and find the difference between them, this would allow us to find the margin of error and it might be something I come back to. Right now if we increase the number of sides of the polygon we tend towards the actual circumference of a circle. So what expression do we have for this?

$$
\(\pi\) = \lim_{{x \to \infty}} N \cdot \sin\left(\frac{180}{N}\right)
$$

Full mathematics about that will be added later, but the point is that now we have turned this into a trigonometry problem, as we want to find accurate values of sin(x) with low values of x. 

## Alternative calculations

The other method that I started out with was the taylor series of sine and cosine. As the taylor series converges directly to the trigonometric function, we could instead make a polynomial out of the function and solve the polynomial to give Pi.

## C++ ahoy

## SystemVerilog

## FPGA implementation

[pi-imdb]: https://www.imdb.com/title/tt0138704/

