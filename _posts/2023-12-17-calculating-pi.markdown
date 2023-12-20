---
layout: post
title:	"Calculating pi numerically: an ode to accuracy"
date:	"2023-12-17"
categories: project
---

A few days ago I was playing around with my old A-Level CASIO classwiz fx-991ex calculator, in particular I was using it for a past paper for one of my upcoming exams. Towards the end of that session I looked over the mark scheme having felt good about that past paper and then saw the precision of my answers vs the mark scheme were completely wrong. Turns out in engineering the standard practise is to round at every stage of the working out and not just when you get a final answer. 

Certainly frustrating as I felt paranoid enough to check my answers several times over, and then afterwards I couldn't stop thinking of the need for balance between accuracy and simplicity in engineering. After all we don't want our heads to explode when trying to solve calculations that take into account every small detail when we could just focus on the handful of dominant terms. However at the same time reading about the invention of the junction transistor and seeing that it was only completely understood by taking into account the ignored minority carrier concentrations, makes me think twice of that.

Surely, there does lie a balance but that is not what this projects is about, I just want to throw simplicity to the winds and calculate Pi to as much accuracy as I can handle. Firstly, doing some basics Maths to understand what functions and terms we may need to use, then using C++ to compute Pi numerically and then trying to stretch this even further and see if more accuracy or speed can be squeezed out of implementing this into SystemVerilog.

## Initial calculations

### The naive approach

So the obvious enough thing to do is to find a value for pi, knowing that pi is nothing more than the circumference of a circle divided by it's diameter.

$$
\pi = \frac{circumference}{diameter}
$$

So we find the circumference as a function of the diameter, this involves drawing a polygon inside a circle (in this case. So the perimeter of the polygon works as an estimate for the circumference, which we find to be. 

If we were feeling motivated enough we might want to draw a polygon inside and outside and find the difference between them, this would allow us to find the margin of error and it might be something I come back to. Right now if we increase the number of sides of the polygon we tend towards the actual circumference of a circle. So what expression do we have for this?

$$
\pi = \lim_{N\to\infty} N \cdot \sin\left(\frac{180}{N}\right)
$$

So now the problem turns from a basic geometry problem into an evaluating limits problem. So to compute Pi accurately now we need to accurately calculate small values of $$ \sin(x) $$.  

### Taylor series

The other method that I started out with was the taylor series of sine and cosine. As the taylor series converges directly to the trigonometric function, we could instead make a polynomial out of the function and solve the polynomial to give Pi.

### CORDIC approach

### Conclusion

## C++ ahoy

## SystemVerilog

## FPGA implementation
