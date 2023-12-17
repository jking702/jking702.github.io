---
layout: post
title:	"Calculating pi numerically"
date:	"2023-12-17"
categories: project
---

An interesting idea came to me recently when I was watching the film [Pi][pi-imdb]. In the film the reclusive Mathematical genius Max Cohen discovers a 216 digit number that becomes of interest to many groups, as this number has been shown to accurately predict day to day stock prices and apparently has some links to the true name of god.

Regardless of the existance of such a number, the number apparently had links to the mathematical constant we all know about, and consequently the title of the film, pi. So it made me start thinking about how we calculate pi as we are now able to get pi to hundreds of digits long.

So in order to test my mathematical powers and to take my first step into digital hardware I thought I might try and calculate pi to at least 20 decimal places (which is about double the precision of my high school calculator). This also gives an interesting look into how numbers are manipulated in digital for calculations.

## Initial calculations

So the obvious enough thing to do is to find a value for pi, knowing that pi is nothing more than the circumference of a circle divided by it's diameter.
$$
\(\pi\) = \frac{circumference}{diameter}
$$

We can begin finding the circumference by drawing a polygon inside a circle. Then if we can find the perimeter of the polygon we have a estimate for the circumference. If we were feeling motivated enough we might want to draw a polygon inside and outside and find the difference between them, this would allow us to find the margin of error and it might be something I come back to. Right now if we increase the number of sides of the polygon we tend towards the actual circumference of a circle. So what expression do we have for this?

$$
\(\pi\) = \lim_{{x \to \infty}} N \cdot \sin\left(\frac{180}{N}\right)
$$

Full mathematics about that will be added later, but the point is that now we have turned this into a trigonometry problem, as we want to find accurate values of sin(x) with low values of x. 

## Alternative calculations

The other method that I started out with was the taylor series of sine and cosine. As the taylor series converges directly to the trigonometric function, we could instead make a polynomial out of the function and solve the polynomial to give Pi.

## C++ ahoy

## SystemVerilog

##

[pi-imdb]: https://www.imdb.com/title/tt0138704/

