---
layout: default
usemathjax: true
permalink: /cg/bressenhamcircle
---

# Bressenham's Circle Algorithm

$\require{color}$

![Decision-points for the plotting of neighbouring pixels](/notes-blog/assets/img/cg/decision-points.jpg)

We use the radius error $RE(x_i,y_i) = x_i^2+y_i^2-r^2$​​. 

- The $\textcolor{blue}{blue}$​ pixel $RE(x_{i+1}, y_{i+1}) = RE(x_i + 1, y_i)$
- The $\textcolor{red}{red}$​​ pixel $RE(x_{i+1}, y_{i+1}) = RE(x_i + 1, y_i - 1)$​​

The decision parameters are the sum of the two $RE$s. 

- $D_{i+1} = RE(x_i + 1, y_i) + RE(x_i + 1, y_i-1) = 2(x+1)^2 + y^2 + (y-1)^2 - 2r^2$​​​
- $D_i = RE(x_i, y_i) + RE(x_i, y_i+1) = 2(x)^2 + y^2 + (y+1)^2 - 2r^2$
- $D_{i+1} - D_i = 4x + 2 + 4y$

We can simplify this even further:

- $D_i < 0 \Rightarrow D_{i+1} = D_i + 4x_i + 6$​ and draw pixel$_i$ on the **outside**.
- $D_i < 0 \Rightarrow D_{i+1} = D_i + 4(x_i-y_i) + 10$​ and draw pixel$_i$​​ on the **inside**.



Using the **Bresenham Circle Algorithm**, we have the following:

```C
d = 3 - 2 * radius;

for (x = 0; x < y; x++) {
	write_pixel(cx + x, cy + y, color); // top right
	write_pixel(cx + x, cy - y, color); // bottom right
	write_pixel(cx - x, cy + y, color); // top left
	write_pixel(cx - x, cy - y, color); // bottom left
	
	write_pixel(cx + y, cy + x, color); // right
	write_pixel(cx + y, cy - x, color); // right
	write_pixel(cx - y, cy + x, color); // left
	write_pixel(cx - y, cy - x, color); // left
	if (d <= 0)
		d = d + 4*x + 6;
	else {
		d = d + 4*(x-y) + 10
		y--;
	}
}
```
