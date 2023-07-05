After we drew a first fractal last week (the Tree of Pythagoras), we want to produce more mathematical art this time.

# The goal
<iframe width="560" height="315" src="https://www.youtube.com/embed/b005iHf8Z3g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# 0. The initial setup

Using the QtCreator start a new Qt/Widget/C++-Project named "Mandelbrot" (derive the main widget from `QtWidget`), don't specify any default language, and reduce the main program to:
```C++
  #include <QtApplication>
  #include "mandelbrot.h"

  int main(int nArgs, char** args) {
    QtApplication app(nArgs, args);
    MandelbrotWidget window;
    window.show();
    return app.exec();
  }
```

The header file looks about the same as last time:

```C++
  #include <QtWidget>

  struct Rect {
    double x0, y0, dx, dy;

    inline double y1() const {
        return y0 +dy;
    }
  };

  class MandelbrotWidget :publc QtWidget {
    public:
      explicit MandelbrotWidget(QtWidget* parent);
      virtual ~MandelbrotWidget();

      void paintEvent(QPaintEvent*) override;

    private:
      Rect range = { -2.0, -4.0/3, 2.5, 8.0/3 };
      Rect scale;
  };
```

`Rect` is abbreviation for rectable.  Such a rectangle consists of one point $(x_0, y_0)$ its width `dx` and its height `dy`.  The given Range is the full Mandelbrot, with which we will start.

In order to map the Mandelbrot to the screen/into our Widget, we will need to compute its `scale` first.


# 1. Defining Mandelbrot
What is a Mandelbrot (杏仁面包)?

Well, literally it means almond bread, i.e. a bread with almond flour (杏仁粉) or split almonds (分裂的杏仁) on the crust (面包皮).  But where it actually came from is the name of the mathematician Bernoît Mandelbrot (贝尔诺特·曼德布罗特), who studied iterating systems and their dynamics.

The core is the following iteration

$$ z_{n+1} = z_n^2+c. $$

It means that you choose a complex (复数) parameter $c=(c_x, c_y)$ and a starting point, like $z_0=(0,0)$, and then you compute in complex coordinates the square of the last $z_n$ and add $c=(c_x,c_y)$.

If you are lucky, your math teacher or your father will tell you in highschool, what are complex numbers (复数).  For our purposes it is sufficient to know that the complex number $z_n=(x,y)$ consists of 2 real (实数, i.e. usual) numbers.  They are added in components as follows
 $$ z+c = (x+c_x, y+c_y). $$

Multiplying complex numbers is a bit more tricky, because we want $zw$ to be 0 if and only if $z$ or $w$ are 0.  Therefore, we have the rule $\mathsf{i}^2=-1$ and $1=(1,0)$ as well as $\mathsf{i}=(0,1)$.  So in total that leads to
 $$ zw = xw_x + \mathsf{i}(xw_y+yw_x) +\mathsf{i}^2yw_y = (xw_x-yw_y, xw_y+yw_x). $$

In particular, the square of the complex number $z$ is
 $$ z^2 = (x^2-y^2, 2xy). $$

When we implement that, we need to make sure that in $z_{n+1}= z_n^2+c$ we use the old values for every place in the equations, maybe like this
```C++
  double x,y, cx,cy = ...;
  double  x1 = x*x -y*y + cx;
  y = 2*x*y + cy;
  x = x1;
```

## 1.2 What is an Iteration (迭代)?
The formulas above tell us to compute $z_1= z_0^2+c$.  Then, we can compute $z_2=z_1^2+c$, $z_3=z_2^2+c$, and so on.  In particular, you obtain a whole sequence (列) of (complex) numbers: $z_0$, $z_1$, $z_2$, $z_3$, ...

Unfortunately, you cannot draw infitintely many numbers (or even compute them).  Instead, Bernoît Mandelbrot found out that you get an interesting pattern when you make a black dot for every number $c=(c_x,c_y)$ whose sequence remains bounded (有界), i.e. the numbers do not grow bigger and bigger beyond every limit.

In 2D ($z=(x,y)$) the easiest bounds are circles around the origin, i.e. $r=\sqrt{x^2+y^2}$ and experience teaches us that you have to go upto $r\le2$.  In order to save computing the square root, you just stick with $r^2=x^2+y^2$ and compare $r^2\le4$.


# 2. How to draw that?
## 2.1 How to decide “remains bounded”?

As said above, experience teaches us that we need to test for $r^2\le4$.  Unfortunately, we cannot compute infinitely many sequence elements.  Instead, we decide for a maximum number `maxColors=256` and compute at most 256 sequence elements and check whether for each $r^2=x^2+y^2\le4$.  If that remains true up to `maxColors`, the point $c=(c_x, c_y)$ belongs to the Mandelbrot, we will draw it in black.

## 2.2 How to Produce Colors?

In the sample animation they did not just draw black-and-white, but they also draw the points at the boundary in colors.  We can determine their color as follows:  If already the first point ($z_1=c=(c_x, c_y)$) violates $r^2\le4$, we will paint it with color 1, (e.g. blue), if the first point remains inside, but the second point $z_2=(x,y)$ is outside, then we draw $(c_x, c_y)$ with color 2 (e.g. green), and so on.

We can do that with the following function:
```C++
  static unsigned maxColors = 256;

  unsigned iterate(double x0, double y0, double cx, double cy) {
    double x=x0, y=y0;
    for (unsigned color=1u; color<maxColors; color++) {
      double x2 = x*x, y2 = y*y;
      if (x2+y2>4.0)
        return color;
      y = 2*x*y +cy;
      x = x2-y2 +cx;
    }
    return 0u;
  }
```

### But that are Numbers not Colors!

Well, you just have to convert them to colors, e.g. as follows

```C++
  QColor colorize(unsgined color) {
    if (color==0u)
      return Qt.black;
    const int c = int(color) % 256;
    return QColor(c, 128, 255 -c);
  }
```

`int(color)%256` means to compute the iteration value modulo 256, i.e. $1\mapsto1$, $2\mapsto2$, ..., $256\mapsto0$, $257\mapsto1$, ... .  `QColor(c,128,255-c)` means that the colors range from $(1,128,254)$ (light blue) over $(128,128,127)$ (gray) until $(255,128,0)$ (light read).


## 2.3 How to draw a whole Picture?

For that we use an old trick from television: instead of computing all points at the same time, we run through the picture from top-left until bottom-right row-by-row and and compute for every point its color and draw it, maybe like this:

```kotlin
  ...

  void MandelbrotWidget::paintEvent(QPaintEvent*) {
    auto p = QPainter(this);
    scale = computeScale(range, width(), height());
    double cy = scale.y0;
    for (int py=0; py<height(); py++) {
      double cx = scale.x0;
      for (int px=0; px<width(); px++) {
        auto c = iterate(0.0, 0.0, cx, cy);
        p.setColor(colorize(c));
        p.fillRect(px, py, px+1, py+1);
        cx += scale.dx;
      }
      cy += scale.dy;
    }
  }
```
`for (int py=0; py<height(); py++)` is a for-loop in the variable `py`.  It starts at 0, then 1, ..., until excluding `height()`, i.e. the last value is `height()-1`.  This is necessary, because the window is `height()` points high, from 0 until including `height()-1`.

Correspondingly for `px`.

At the end of every loop, we have `cx += scale.dx` (or the corresponding for `cy`).  We need this in order to shift $c=(c_x, c_y)$ to the next point.  Before every loop, we define `double cx = scale.x0;`, i.e. the left (upper) boundary of the image.


It remains to compute the current scale.  For that, we query the current window size as `(width(), height())`.  It is important to draw the Mandelbrot always with the 1:1 side ratio, i.e. we need $|dx|=|dy|$ in the end.  Generally speaking, we want to use the scales `range.dx/width` and `range.dy/height`.  However, when they are different, we need to use for both axes the bigger scale (sic!). Therefore, we define
```C++
  #include <algorithm>
  using namespace std;

  Rect computeScale(const Rect& range, unsigned width, unsigned height) {
  double dx = max(range.dx/width, range.dy/height);
  ...
  return { x0, y0, dx, -dx };
  }
```

Indeed, we have to invert the y-scale, because on screen the y-axis points downwards, but in a mathematical coordinate system the y-axis points upwards, therefore `-dx`.

### b) How to obtain the bounds `(x0, y0)`?

In principle, we want to use `range.x0` and `range.y0`, but there are 2 problems:

1. When `range.dy/height > base.dx/width`, we have to add to the left (and to the right) a little extra. Therefore `double x0 = range.x0 -(dx*width-range.dx)/2;`.

2. The y-axis has a negative scale (`dy < 0`). Therefore, we need to use `y1 = range.y0 +range.dy`, and correspondingly, we have to add the offset, i.e. `double y0 = range.y1() +(dx*height-range.dy)/2;`.

The formulas may not be obvious. Therefore, you best check, e.g. by verifying that the Mandelbrot is visible on screen (instead of just showing a blue rectangle).

It remains to fill up the definition of the remaining methods of `MandelbrotWidget`, e.g. as follows:

```C++
  #include "mandelbrot.h"
  #include <QtGui>
  #include <algorithm>
  using namespace std;

  ...
  MandelbrotWidget::MandelbrotWidget(QtWidget* parent) :QtWidget(parent), scale(computeScale(range, 300, 320)) {
    setMinimumSize(300, 320);
    setGeometry(100,0, 600, 640);
    setWindowTitle(tr("Mandelbrot"));
  }

  MandelbrotWidget::~MandelbrotWidget() = default;

  ...
```

So the minimum window size is $300 \times 320$ that respects the side ratio of the range.  We also suggest the window geometry to the window manager as $(100,0)+600\times640$, i.e. if we get the whole thing through, then the window will occur on the top, 100 pixels from the left and of size $600\times640$.  The window manager is not required to respect these coordinates, but with most GUIs it does.  If your laptop has a quite small screen, you may have to adjust those coordinates.  One somewhat ugly point is that we need to initialize the scale (even before knowing the dimensions of the window).  Therefore, we just compute the scale with some assumed width and height.  Note that this scale is overridden as soon as we start the `paintEvent(...)` method, because there, we compute it again with the proper size of the window.


# 9. Try for Yourself
Now it is your turn.  When you typed in the whole program without errors, then it should be startable and draw the Mandelbrot (the beginning of the above video).

## 9.0 Does your Program work?

If it does not compile, have a look at the compiler error message(s).  Can you understand where the error happens?  What does it complain about?  Compare to the program here and try to fix the problem(s).

## 9.1 Can you change the color palette?

Currently, the program draws in shades of purple.  What do you need to change (and how) in order for the colors to change from red via yellow to green?

Hint:  You need to change the function `QColor colorize(unsigned iterations)`.


## 9.2 How to draw only a zoom-in of the Mandelbrot?

Hint: `Rect range = {...};` determines the range you see.

How to zoom to the ball at the left end of the Mandelbrot?

Please enjoy trying around with the program.  If the program does not do what you expected, just restore the old values and start from there again.
