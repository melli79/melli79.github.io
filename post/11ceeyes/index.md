After we have set up a [development environment](../00-setup-ide/), we want to start drawing something simple.

# Our Goal
![(.)(.)](/img/eyes.png "100x50")

# 1. Setting up the Project

With the QtCreator, you create a New Project ... of type C++ Qt Widget Application.  Give a memorizable name to the project, e.g. movingEyes (移動的眼睛), but make sure the Build-System is cmake/minGW11.  Call the main window "EyesWidget" and try to make it derive from `QWidget`. We don't need to choose any translation language (as our app will not contain much text).

After a while, the IDE (Integrated Development Environment) opens the file "main.cpp" with a minimal Qt main program.  You can even reduce this as follows:

```C++
#include "eyewidget.h"

#include <QApplication>

int main(int nArgs, char* args[])
{
    QApplication app(nArgs, args);

    EyeWidget w;
    w.show();
    return app.exec();
}
```

This is all we need here.

## Meaning

What do these lines mean?  We create a standard Qt application and hand over the command line arguments, `nArgs` is the number of command line arguments (at least 1) and `args` are these arguments, e.g. `args[0]` is the executable program name.


# 2. Drawing Eyes

How to draw something into the Window?

Well, we have to write our own class, that extends the class `QWidget`.  As such it will be a class with a Qt enhancement and therefore needs its own declaring header file.  But the QtCreator has already created these 2 files.  First, we should double check the header file.  In its minimal definition, it should look as follows:

```C++
#ifndef EYEWIDGET_H
#define EYEWIDGET_H

#include <QWidget>

class EyeWidget : public QWidget
{
    Q_OBJECT

public:
    explicit EyeWidget(QWidget *parent = nullptr);
    ~EyeWidget() override;

    void paintEvent(QPaintEvent*) override;
    void mouseMoveEvent(QMouseEvent*) override;

private:
    double posXL = 0;
    double posXR = 0;
    double posY = 0;
};
#endif // EYEWIDGET_H
```

2 things will be in red now:  the methods `paintEvent(...)` and `mouseEvent(...)`.  In order to obtain a minimal implementation for then, put the cursor on one of them and press \<Alt\>+\<Enter\>.  In the menu there should be an option "implement ...".  Once you have chosen that, the method declaration should become normal colored (such as `explicit EyeWidget(...)`).[^1]

## 2.1 What does the "\*.h" file mean?

In a header file, we define classes and declare methods that will be provided for these classes.  In the above example, we define a `class EyeWidget` and its 4 methods are written between `public:` and `private:`.  They are written under `public:` that means that all will have public access.  But there are 2 additional things:  First the class is not defined from scratch, instead it derives from `QWidget`.  A `QWidget` is a component of a Qt Application that allows for drawing of elements and receiving user input.  It is the basic class for such interaction.  The additional keyword `public` means that all its publicly accessible methods remain publicly accessible.  This should be your default for class inheritance.

Another odd thing is the identifier `Q_OBJECT`.  This is a Qt Macro and signal word that identifies this class as belonging to the Qt class hierarchy.  It makes the class handleable by the Qt Framework.  We will need that for all classes that allow for Windows user operations.

In addition to all this public stuff, we have also defined 3 private variables: `posXL`, `posXR` and `posY`.  These will be necessary to interact with the user.  We will see their meaning once we implement the methods `paintEvent(...)` and `mouseMoveEvent(...)`.  In short `posXL` will be the horizontal position of the eyes' focus from the left eye, `posXR` the horizontal position from the right eye, and `posY` the vertical position wrt. both eyes.

[^1]: If automatically implementing the methods did not work, then you can manually add them to the file "eyeswidget.cpp" as shown below.


## 2.2 The EyeWidget Implementation

The main work of drawing the eyes is done in the file "eyewidget.cpp".  In the beginning (after you have added minimal implementations of `paintEvent(...)` and `mouseMoveEvent(...)`), it should look as follows:

```C++
#include "eyewidget.h"

EyeWidget::EyeWidget(QWidget* parent)
        : QWidget(parent) {
}

EyeWidget::~EyeWidget() = default;

void EyeWidget::paintEvent(QPaintEvent*) {

}

void EyeWidget::mouseMoveEvent(QMouseEvent* event) {

}
```

Once, this is done, you should be able to compile and start the program, e.g. by pressing \<Ctrl\>+\<R\>.  Please give it a try to make sure everything works so far.

Let us now start with drawing eyes.  Fill in the beginning of the `paintEvent(...)` method as follows:

```C++
#include <QtGui>
#include <algorithm>
#include <cmath>
using namespace std;

...

void EyeWidget::paintEvent(QPaintEvent*) {
    auto p = QPainter(this);
    p.setBrush(QBrush(QColor("white")));
    const double eyeWidth = 0.9*min(width()/2, height());
    p.drawEllipse(width()/2 -int(1.0*eyeWidth), 10, int(0.95*eyeWidth), height() - 20);
    p.drawEllipse(width()/2 +int(0.05*eyeWidth), 10, int(0.95*eyeWidth), height() - 20);
    ...
}
```

## 2.3 What does that mean?

Ok, `EyeWidget::EyeWidget(...)` is the so-called constructor.  It is where the existence of an object starts.  It takes (at most) 1 argument `QWidget* parent`, i.e. the parent widget.  But when you look into its declaration in the header file, then you see that it also has a default argument `nullptr`.  So if we do not specify a parent widget, then that will be `NULL`.  This will imply that the widget will become the main widget of a new Window on the desktop.

We also have to pass this `parent` widget to the constructor of the parent class.  That is what `: QWidget(parent)` means, i.e. at this point the parent class will be initialized.  At the moment the body of the method is empty `{  }`, i.e. we do not have to do anything specific.

The `EyeWidget:~EyeWidget()` is the destructor.  The same way every object is constructed before its usage, it also needs to be destructed before the return of its memory.  This is done in the destructor.  In this case we won't do anything specific and can thus just use the default implementation `= default;`.

The more interesting method is `paintEvent(...)` in which the widget is asked to paint its content.  We start by defining a `QPainter` for the current object called `this` and setting its brush to color "white".  If you see a lot of red marked methods/classes, then you need to import \<QtGui\> where all the Qt graphic painting elements are defined, e.g. `QPainter`, `QBrush` and `QColor`.

As it may suggest, `p.setBrush(QBrush(QColor("white")))` sets the current brush to be of the solid color white.  This will be helpful for the eyeballs.

Then we compute the width of our eyes (in the computer).  The problem with modern Desktop (or browser) drawing is that we don't know how big the actual picture will be, e.g. the user can resize the window that contains our drawing, and we need to make the best out of the available space.  Therefore, we will define the `eyeWidth` as the minimum of either `width()/2` or `height()-20`.  These 2 methods provide the current window sizes (width and height, respectively).  The function `min(...)` is defined in the standard library in header \<algorithm\> and for convenience, we import all of the `namespace std`.  The additional factor of `0.9` is to make sure the eyes fit losely into the window.

Eyeballs are just ellipses, that is why we `drawEllipse` twice.  There are 4 parameters to specify an ellipse.  In our case the top-left corner of the bounding rectangle and its width and height.  Again `width()/2` is in the middle of the window and `-int(eyeWidth)` means that the left boundary of the left eye is one `eyeWidth` to the left of the middle of the window.  The eye starts 10 pixels from the top of the window.  The width is `0.95*eyeWidth` such that it leaves a small gap between the 2 eyes (where the root of your nose is).  The height of the eye is almost the window height (reduced by 20 pixels), i.e. no matter how big the user makes the window, the eye will almost fill it, only 10 pixels short on top and on the bottom.

For the right eyeball, we go $0.05\*$ `eyeWidth`s to the right and draw an ellipse of the same size.

When you now restart the program, you should see a gray window with 2 white eyeballs.  You can check what happens when you resize the window (e.g. press the full-screen button on top right), the eyeballs should grow and shrink with the window.


## What if the Program does not Compile anymore?

This is why we write the program incrementally, i.e. one step at a time.  You will know that the error is likely caused by something you just added.  Try to understand the error message and the place where it occurs.

Some hints:

* all parentheses have to be round '(' and ')';
* the decimal point has to be a point, i.e. `0.95` not `0,95`;
* every invocation of `p.drawEllipse()` needs 4 parameters, i.e. 3 commas and between each 2 commas every opening parenthesis needs to be closed as well;

## 2.4 Iris and Pupils

For that we extend the `paintEvent(...)` method as follows:

```C++
void EyeWidget::paintEvent(QPaintEvent*) {
  ...
    p.setBrush(QBrush(QColor("#630")));
    p.drawEllipse(width()/2 +int(0.35*posXL-0.825*eyeWidth), height()/2 +int(posY -0.3*eyeWidth), int(0.6*eyeWidth), int(0.6*eyeWidth));
    p.drawEllipse(width()/2 +int(0.35*posXR+0.225*eyeWidth), height()/2 +int(posY -0.3*eyeWidth), int(0.6*eyeWidth), int(0.6*eyeWidth));
    p.setBrush(QBrush(QColor("black")));
    p.drawEllipse(width()/2 +int(0.35*posXL-0.725*eyeWidth), height()/2 +int(posY -0.2*eyeWidth), int(0.4*eyeWidth), int(0.4*eyeWidth));
    p.drawEllipse(width()/2 +int(0.35*posXR+0.325*eyeWidth), height()/2 +int(posY -0.2*eyeWidth), int(0.4*eyeWidth), int(0.4*eyeWidth));
    p.end();
}
```

Note that every `drawEllipse` comes in pairs, one for the left eye and one for the right eye.  The iris color is "#630", i.e. 6/16 parts red, 3/16 parts green, and 0/16 parts blue.  That is a dark brown.  If you wish, you can also paint steel blue eyes, e.g. "#03C" where C=12, i.e. the blue component is 12/16.  If I have not messed up with the numbers and you restart the correctly typed in program, you should see 2 dark-brown eyes staring at you.

### What do the numbers mean?

I had to fiddle quite a bit with them and they may be not the unique solution to the problem, but `int(0.35*posXL-...)` means the position of the left eyes is adjusted to 35% of the position of the cursor w.r.t. the center of the left eyeball.  Correspondingly it is fully adjusted to the height of the cursor over the middle of the window.  The second components `int(...-0.825*eyeWidth)` and `int(...-0.3*eyeWidth)` mean that the left iris starts 82.5% into the left eyeball (when the eye stares at you) and 30% to the top of the eyeball.  Correspondingly, the right iris starts 22.5% into the right eyeball and also 30% into the top of the eyeballs.  If you choose different factors for the heights, then you will end up with squinting eyes.

Correspondingly, the pupils move with the `posXL`/`posXR` and `posY` by the same factors.  This is reasonable, because the pupils are always in the center of the irises.  They are, however smaller than the iris, and always black.

Please try to restart the program and see if everything is set up well.


# 3. Moving eyes

What really makes the difference between colorful cicular patterns and eyes is that eyes follow the object they watch.  We will make the computer eyes follow the mouse cursor as follows:

```C++
template <typename F>
F sqr(F x) {
    return x*x;
}

void EyeWidget::mouseMoveEvent(QMouseEvent* event) {
    const double xl = event->position().x() -0.25*width();
    const double xr = event->position().x() -0.75*width();
    const double y = event->position().y() -0.5*height();
    const double eyeRadius = 0.9 * min(width()/4, height()/2-10);
    const double fl = eyeRadius/sqrt(sqr(xl) +sqr(y) +sqr(eyeRadius));
    const double fr = eyeRadius/sqrt(sqr(xr) +sqr(y) +sqr(eyeRadius));
    posXL = xl*fl;  posXR = xr*fr;
    posY = y*(fl+fr)/2;
    update();
}
```

## What does that mean?

1. $x_L = \mathrm{event}\to x - 0.25*\mathrm{width}$ is the x-distance of the mouse position (`event->x(), event->y()`) from the middle of the left eye -- in particular $x_L<0$ whenever the mouse is left of (the middle of) the left eye, $x_L>0$ whenever the mouse is to the right of (the middle of) the left eye;

2. $x_R = \mathrm{event}\to x -0.75*\mathrm{width}$ is the x-distance of the mouse position from the middle of the right eye;

3. $y = \mathrm{event}\to y -0.5*\mathrm{height}$ is the y-distance of the mouse position from the middle of the eyes;

4. `F sqr(F x) { return x*x; }` is the square, we use the generic definition `template <typename F>` i.e. we define at the same time the square for integers as well as for floating point numbers;

5. `sqrt(sqr(xl)+sqr(y))` is the distance of the mouse position from the middle of the left eye (s.a. Pythagorean theorem);

6. `sqrt(...+sqr(eyeWidth)` is a safety minimum distance, i.e. the divisor is at least `eyeWidth` (because next we will divide by this number);

7. $ \mathrm{pos-}x_L = x_L\*f_L$; $ \mathrm{pos-}x_R = x_R\*f_R $ means, that the eye will move only a bit in the direction of the mouse;

8. $f_L = \mathrm{eyeWidth}/\sqrt{...}$ and $f_R = \mathrm{eyeWidth}/\sqrt{...}$ are these small factors, `sqrt` means square root;

9. $ \mathrm{pos-}y = y*(f_L+f_R)/2$ means that for the vertical displacement of the eyes we use the average of the 2 factors, in particular both eyse will always have the same height (otherwise they would be squinted);

10. `update()` means that we request the computer to update the picture of the eyes.  We cannot simply call `paintEvent(nullptr)`, because the computer may not be ready for painting;


## 3.9 One last Issue

If you resize the window, there seem to be no constraints, i.e. you can even make the window as small as only to consist of the top border.  That may not be helpful.  Therefore, we add the following commands to the widget constructor:

```C++
EyeWidget::EyeWidget(QWidget* parent)
    : QWidget(parent) {
  setMinimumSize(160, 120);
  setWindowTitle(tr("Qt Eyes!"));
  setMouseTracking(true);
}
```

This has 2 additional effects, first, we set the window title to "Qt Eyes!", and second, we will be notified whenever the mouse moves.  This is necessary, because otherwise the system will save energy in not calling the method `mouseMotionEvent(...)`.

One last question:  Why didn't we use the parameter of the `paintEvent(QPaintEvent*)`?

The answer is that we will always quickly draw the whole 2 eyes.  If your drawing takes a long time, you can read off the details of the region to update from this `QPaintEvent`.

Ok, if everything is correct, the 2 eyes should follow the mouse pointer.  Maybe, you first have to click into the window with the eyes (or at least move the mouse into this window).


# 9. Try for Yourself

## 9.1 Colorful Eyes

Try to draw green or blue eyes!

If you know that `QColor("#630")` produces a dark brown brush, then `QColor("green")` produces a middle green color.  The numbers for steel blue are "#03C".


## 9.2 Squinted Eyes

There are a couple of methods.  The simplest is to make one eye look a bit higher, e.g. so:
```C++
  ...
  p.drawEllipse(width()/2 +int(0.35*posXR+0.225*eyeWidth), int(0.4*height() +posY -0.3*eyeWidth), int(0.6*eyeWidth), int(0.6*eyeWidth));
  ...
  p.drawEllipse(width()/2 +int(0.35*posXR+0.325*eyeWidth), int(0.4*height() +posY -0.2*eyeWidth), int(0.4*eyeWidth), int(0.4*eyeWidth));

```

The y-coordinate is measured from the top, i.e. $y=0$ is the upper boundary of the window, $y=0.5\*\mathrm{height}$ in the middle and $y=\mathrm{height}$ the lower boundary.  If we use $y=0.4\*h$ instead of $y=0.5\*h$, this will turn out a bit higher than the middle.  And if you do that with only one eye, then this will look higher than the other.


## 9.3 Drawing a Snowman

Can you replace the method (`void EyeWidget::paintEvent(...)`) such that it draws 3 stacked balls?

Hint:  Every ball should have a diameter of `const int d = height()/3`, and the first ball begins on top ($y=0$), the second after 1/3 (`y=height()/3`), and the last one at 2/3 (`y=2*height()/3`).

Please enjoy trying out different things and let me know if there are any problems.
