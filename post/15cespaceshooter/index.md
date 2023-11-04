After we drew some fractals last week, we want to produce something more dynamic and interactive this time.

# The goal

**Disclaimer!** Playing computer games endlessly, leads to neglegt of the real world, your health, and can harm developing children and teenager.  Please be responsible and limit the gaming time.

**免责声明:** 无休止地玩电脑游戏会导致对现实世界和自身健康的否定，并可能对发育中的儿童和青少年造成伤害。
请负起责任，限制游戏时间。

**Disclaimer!** Weapons and wars can harm and kill people.  Making these part of fictional games does not imply their harmlessness.  Always operate weapons with care and do not hand them to children, teenagers or psychologically fragile people.

**免责声明:** 武器和战争会伤害和杀死人。将其作为虚构游戏的一部分并不意味着它们无害。请务必小心操作武器，不要将武器交给儿童、青少年或心理脆弱者。

<center>
<video width="320" height="240" controls autoplay="true" loop>
  <source src="/movies/spaceShooter.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
</center>

# 0. The initial setup

Using the QtCreator start a new Qt/Widget/C++-Project named "SpaceShooter" (太空射击游戏, derive the main widget from `QtWidget`), don't specify any default language, and reduce the main program to:
```C++
  #include <QtApplication>
  #include "spaceShooter.h"

  int main(int nArgs, char** args) {
    QtApplication app(nArgs, args);
    SpaceShooterWidget window;
    window.show();
    return app.exec();
  }
```

The header file looks similar to last time:

```C++
  #include <QtWidget>
  #include <vector>

  struct Rect {
    double x0, y0, dx, dy;

    inline double y1() const {
        return y0 +dy;
    }

    inline double px(double x) const {
      return int(lround((x-x0)*dx));
    }

    inline double py(double y) const {
      return int(lround((y-y0)*dy));
    }
  };

  struct Point {
    double x, y;
  };

  class Shot {
  public:
    double x, y;
  private:
    double vy;
  public:
    Shot(double x0, double y0, double v0=0.1) :x(x0),y(y0),v(y0) {}
    Short(const Shot& src) = default;
    Shot& operator =(const Shot& src) = default;

    bool isUp() const {
      return vy>0.0;
    }
    ...
  };

  class SpaceShooterWidget :publc QtWidget {
    public:
      explicit SpaceShooterWidget(QtWidget* parent);
      virtual ~SpaceShooterWidget();

      void paintEvent(QPaintEvent*) override;
      void keyPressEvent(QKeyEvent*) override;

    protected:
      void accelerateLeft();
      void accelerateRight();
      void shoot();
      void evolve();
      void reset();
      void fillBackground();

      void loadImages();
      QPixmap* rock = nullptr;
      QPixmap* ship = nullptr;
      QPixmap* shot = nullptr;
      QPixmap* boom = nullptr;
      QPixmap* bomb = nullptr;

    private:
      uint score = 0;
      double posX = 0.0;
      const double posY = 0.0;
      vector<Point>  brackground;
      vector<Shot> shots;
      const Rect range = { -1.0, 0.0, 1.0, 1.0 };
      Rect scale = {};
      ...
  };
```

You should remember `Rect`, a rectangle that keeps the dimensions of our app, `range` is the size in our fixed coordinates and scale is the scaling to window coordinates.

The background consists of rocks (石) that will be represented by `Point`s (平面内的点, that are unmoving).

Later we will add shots (枪声) that will be represented by `Shot` which also has coordinates and in addition a vertical velocity (垂直速度, `vy`).


# 1. The basic Shooter Setting

The file "spaceShooter.cpp" contains the definitions of methods for our widget.  As such it must contain all the methods we have declared in "spaceShooter.h" and did not define there.

You probably remember the `SpaceShooterWidget::SpaceShooterWidget(QWidget* parent)` -- the constructor --, and `SpaceShooterWidget::~SpaceShooterWidget()` -- the destructor.  They are common.

Whatever we wish to display in the window, we need to paint in the `paintEvent(QPaintEvent*)`.

Our background will be a couple of rocks, so we need to first fill the `vector<Point>` with positions of these rocks and then draw them in the `paintEvent(...)`-method.  But we don't want to repeat the instructions for drawing a single rock over and over again.  Therefore, we load a `QPixmap` (像素图) with the image of one rock and draw that multiple times over the background.  This can be done as follows:

```C++
#include "mainwindow.h"
#include <QtGui>
#include <QApplication>
using namespace std;

  SpaceShooterWidget::SpaceShooterWidget(QWidget* parent)
        : QWidget(parent), ... {
    setMinimumSize(300, 200);
    setMaximumSize(1800, 1200);
    setWindowTitle(tr("Space Shooter!"));
    loadImages();
    reset();
  }

  SpaceShooterWidget::~SpaceShooterWidget() {}

  Rect computeScale(const Rect& range, int width, int height) {
    const double dx = 0.9*width/range.dx;
    const double dy = 0.9*height/range.dy;
    return { range.x0-0.05*range.dx+10/dx, range.y1()+0.05*range.dy+10/dy, dx, -dy };
  }

  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    scale = computeScale(range, width(), height());
    ...
    auto p = QPainter(this);
    p.setPen(Qt::black);
    p.setFont(QFont("Arial", 16));
    p.drawText(2,20, tr("Score: %1").arg(score*50));
    for (const auto& rock : background) {
      p.drawPixmap(scale.px(rock.x), scale.py(rock.y), *this->rock);
    }
    for (auto shot : shots) {
      p.drawPixmap(scale.px(shot.x), scale.py(shot.y), *this->shot);
    }
    p.drawPixmap(scale.px(posX), scale.py(posY), *ship);
    p.end();
  }

  ...

  void SpaceShooterWidget::reset() {
    posX = 0.0;  vx = 0.0;
    points = 0;
    fillBackground();
    shots.clear();
  }

  void SpaceShooterWidget::fillBackground() {
    background.clear();
    for (int x=-9; x<=9; x++) {
        background.push_back({ 0.1*x, 1.0 });
    }
    for (int x=-4; x<=4; x++) {
        background.push_back({ 0.2*x, 0.9 });
    }
    for (int x=-2; x<2; x++) {
        background.push_back({ 0.4*x+0.2, 0.8 });
    }
  }

  QPixmap* loadRock(const QColor& background) {
    auto* rock = new QPixmap(20, 20);
    auto p = QPainter(rock);
    p.fillRect(rock->rect(), background);
    p.setFont(QFont("Arial", 28, QFont::Bold));
    p.drawText(4,28,"*");
    p.end();
    return rock;
  }

  QPixmap* loadShip(const QColor& background) {
    auto* ship = new QPixmap(20, 20);
    auto p = QPainter(ship);
    p.fillRect(ship->rect(), background);
    p.setFont(QFont("Arial", 24));
    p.drawText(2,18,"A");
    p.end();
    return ship;
  }

  ...

  SpaceShooter::loadImages() {
    const auto background = QColor(232, 232, 232);
    rock = loadRcok(background);
    ship = loadShip(background);
    ...
  }

```

If you start the program now, you will see a static image (静态图像) of some rocks '\*' and an 'A' at the bottom of the window.  That is our spaceship (太空船).  (If you get compile errors, that is because we did not yet define all methods we promised.  You can place method definitions with empty bodies for all required methods.)


## 1.2 Make the spaceship move (让飞船移动)
For that we need to interact with the keyboard (反应键盘).  In order to react fast, we just listen for the `keyPressEvent(QKeyEvent* event)`.  Within this method, we have `event->key()` that tells us which key (按键) was pressed.  I have decided for the following steering:  \< \<-- \> (the left key) for accelerating left, \< --\> \> (the right key) for accelerating right, \<Space\> for shooting, \<Backspace\> for reset, and \<Esc\> in order to abort the program.  We have already declared methods for most of that.  Now the `keyPressEvent()` method looks as follows:

```C++
  void SpaceShooterWidget::keyPressEvent(QKeyEvent* event) {
    switch (event->key()) {
      case Qt::Key_Space:  shoot();
        break;
      case Qt::Key_Left:  accelerateLeft();
        break;
      case Qt:Key_Right:  accelerateRight();
        break;
      case Qt:Key_Backspace:  reset();
        break;
      case Qt::Key_Escape:  QtApplication::exit(0);
    }
    event->accept();
  }
```

You probably know the `exit(int)` function.  It exists the program and gives a return value.  0 is the default value and means that everything went smoothly.  For Qt applications it is nicer to call `QtApplication::exit(0)`, because that gives the Qt framework the time to close resources that were allocated in between (like when the program opened a file, 打开文件).  In our case you won't notice much difference, because all the resources are just allocated memory that is freed at the end of the program anyway.

The first approach to moving the spaceship (移动飞船) is to just change the velocity and the position once when the according key is pressed:

```C++
  void SpaceShooterWidget::accelerateLeft() {
    vx -= 0.02;
    posX += vx;
    if (posX<=-1.0) {
      posX = -1.0;
      vx = 0.0;
    }
    update();
  }

  void SpaceShooterWidget::accelerateRight() {
    vx += 0.02;
    posX += vx;
    if (posX>=1.0) {
      posX = 1.0;
      vx = 0.0;
    }
    update();
  }
```

We have also encoded that if the ship hits the bounds (出界) of the window, it will stop moving (停).  (If you don't do that, the ship will move invisibly beyond the world. -- 将隐身于世界之外.)

If you start the program again, you will notice that the spaceship (at the bottom) will move left/right, whenever you press the left-/right-key.  It will move faster and faster (越来越快) if you press multiple times (反反复复).  Note that it will keep moving left if you pressed the left-key many times and then press the right-key few times.  This is called inertia (惯性) and is one effect when flying.


# 2. Shoot at the Rocks (向岩石射击)

Let's keep adding to our program.  Next, we want to shoot at the rocks when the user hits \<Space\> (the long key that always rattles at bit).  A fitst approach is to just create a shot when the user hits space, maybe as follows:

```C++
  void shoot() {
    shots.push_back(Shot(posX, posY+0.1));
    update();
  }

  ...

  QPixmap* loadShot(const QColor& background) {
    auto shot = QPixmap(20, 20);
    auto p = QPainter(result);
    p.fillRect(shot->rect(), background);
    p.setFont(QFont("Arial"), 20);
    p.drawText(2,22, "^");
    p.end();
    return shot;
  }

  ...

  void SpaceShooterWidget::loadImages() {
    ...
    shot = loadShot(background);
    ...
  }
```

If you try the program now, you will notice that the shot gets stuck (子弹不要飞) one unit over the spaceship.  That is not good.  What we need is evolution of the shot position.

## 2.1 Making the Game Dynamic (让子弹飞)

What we need is to update the positions of the shots from time to time (and update the whole scene).  This can be reached with a `QTimer` as follows.  We need to declare such a QTimer in the `SpaceShooterWidget` as follows:

```C++
  ...
  class SpaceShooterWidget :public QWidget {
    ...

  private:
    ...
    QTimer* timer = nullptr;
  }
```

And then we need to start such a timer when we first draw the window, e.g. as follows:

```C++
  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    scale = computeScale(range, width(), height());
    if (timer==nullptr) {
      timer = new QTimer(this);
      timer->callOnTimeout(this, QOverload<>::of(&SpaceShooterWidget::evolve));
      timer->setInterval(100); // every 100ms
      timer->start(100);  // with 100ms delay it starts
    }
    ...
    for (auto& shot : shots) {
      p.drawPixmap(scale.px(shot.x), scale.py(shot.y), *this->shot);
    }
    p.drawPixmap(... *ship);
    p.end();
  }

  void SpaceShooterWidget::evolve() {
    posX += vx;
    if (posX<=-1.0) {
      posX = -1.0;
      vx = 0.0;
    }
    if (posX>=1.0) {
      posX = 1.0;
      vx = 0.0;
    }
    for (auto shot = shots.begin(); shot!=shots.end();) {
      if (! shot->move())
        shots.erase(shot);
      else
        ++shot;
    }
    ...
  }
```

Additionally, we need to define the method `bool move()` in the struct `Shot` (in the header file):

```C++
  class Shot {
    ...
  public:
    bool move() {
      y += vy;
      return -1<=y && y<=1;
    }
  }
```

If you try now, you will notice that the shots fly through the window.  The only odd thing is that they seem never to hit and destroy anything.

## 2.2 Making Shots Hit their Target (让子弹爆炸)

So how do we detect if any shot has hit any rock?

Well we need to compare them pairwise (比对).  The simplest idea is to do that when you paint all of them in the widget.  Therefore we modify the `void paintEvent(...)` method once more:

```C++
  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    ...
    for (auto shot = shots.begin(); shot!=shots.end();) {
      p.drawPixmap(scale.px(shot->x), scale.py(shot->y), *this->shot);
      bool passes = true;
      for (auto rock = background.begin(); rock!=background.end();) {
        if (shot->hits(*rock)) {
          p.drawPixmap(scale.px(shot->x), scale.py(shot->y), boom);
          background.erase(rock);
          score++;
          shots.erase(shot);
          passes = false;
          break;
        } else
          ++rock;
      }
      if (passes)
        ++shot;
    }
    p.drawPixmap(..., *ship);
  }

  ...

  QPixmap* loadBoom(const QColor& background) {
    auto boom = new QPixmap(20, 20);
    auto p = QPainter(boom);
    p.fillRect(boom->rect(), background);
    p.setFont(QFont("Arial", 36, QFont::Bold));
    p.setPen(Qt::red);
    p.drawText(0,28, "*");
    p.end();
    return boom;
  }

  void SpaceShooterWidget::loadImages() {
    ...
    boom = loadBoom(background);
    ...
  }
```

We also need to decide when a `Shot` hits a rock:
```C++
  class Shot {
    ...
    bool hits(const Point& obs) const {
      return abs(obs.x-x)+abs(obs.y-y) < 0.1;
    }
  };
```

Now the shots should explode when they hit a rock (and then the rock as well as the shot are gone).  Give it a try.  Can you eliminate all the rocks?  What happens then?

## 2.3 Game states (游戏的状态)

Well, so far nothing happens when you are finished with shooting all rocks (maybe except that your score is somewhere like 1600).  So our game needs a state (状态).

What states are possible?

Well, `CONTINUE` where we just continue playing, `EXPLODING` if our ship is exploding, `WINNING` when we are winning, and `GAME_OVER` after our ship exploded.  That's only 4 states.  Let us use an `enum` for that.  We need to define it in the header file, because we also need to introduce the property there:

```C++
  class SpaceShooter :public QWidget {
    ...
  public:
    enum State { CONTINUE, EXPLODING, WINNING, GAME_OVER };

  private:
    ...
    uint countDown = 8;
    State state = CONTINUE;
    ...
  }
```

And then we add the handling in the `paintEvent(...)`-method, because that is when it becomes apparent that there are no more rocks left.

```C++
  void paintWinning(QPainter& p, int width, int height) {
    p.setFont(QFont("arial", 64, QFont::Bold));
    p.setPen(Qt::darkGreen);
    p.drawText(width/2 -120, height/2, "You Won!!!");
  }

  void paintGameOver(QPainter& p, int width, int height) {
    p.setFont(QFont("arial", 64, QFont::Bold));
    p.setPen(Qt::red);
    p.drawText(width/2 -180, height/2, "Game Over!");
  }

  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    ...
    for (auto shot = shots.begin(); shot!=shots.end();) {
      ...
      if (passes)
        shot++;
    }
    if (state!=GAME_OVER && background.empty())
      state = WINNING;

    switch (state) {
      case CONTINUE: p.drawPixmap(scale.px(posX), scale.py(posY), ship);
        break;

      case WINNING:
        p.drawPixmap(scale.px(posX), scale.py(posY), ship);
        paintWinning(p, width(), height());
        break;
      case EXPLODING: p.drawPixmap(..., boom);
        if (countDown>0) {
          countDown--;
          break;
        } else
          state = GAME_OVER;
      case GAME_OVER:
        p.fillRect(scale.px(posX), scale.py(posY), 20,20, bgColor);
        paintGameOver(p, width(), height());
    }
  }

  ...

  void SpaceShooterWidget::reset() {
    ...
    countDown = 8;
    state = CONTINUE;
  }
```

If you now swipe off all rocks, then you will see the green letters "You Won!!!", give it a try.


# 3. Introducing a Dangerous Enemy (引入危险的敌人)
Well, so fat it is super easy to win the game.  We should either play against the time, i.e. hit the `GAME_OVER` once the time runs out (超时), or we introduce some defense (防守) for the rocks.  I have decided for the latter:

```C++
  uniform_real_distribution<> u01(0.0, 1.0);

  template <class RNG>
  bool flipCoin(RNG& random, double bias =0.5) {
    return u01(random) < bias;
  }

  void SpaceShooterWidget::evolve() {
    posX += vx;
    ...
    if (state==GAME_OVER) {
      update();
      return;
    }

    const double bias = 0.007/sqrt(background.size()+1);
    for (const auto& enemy :background) {
      if (flipCoin(random, bias))
        shots.push_back({ enemy.x, 0.65, -0.01 });
    }
    ...
    update();
  }
```

We can find the (template) class `uniform_real_distribution` in the header \<random\>.  The object `u01` can generate a uniformly distributed real number between 0 and 1 (介于0和1之间的均匀分布实数), but for that it needs a random value (随机值).  Once this is clear, it becomes also clear what the method `flipCoin(RNG random, ...)` does, it basically flips a coin (掷硬币), so the output is Heads (正面) or Tails (反面), I mean `true` or `false`.  If we have generated a random number between 0 and 1, we would just compare to 0.5 in order to obtain a fair coin (不偏不倚的硬币).  But in the program we will need a a bent coin (偏倚的硬币), i.e. one with a bias (偏倚).  The smaller the bias the less likely it is to reach `true`.  That is the meaning of the second parameter.

The problem is from where do we get `random` (and what type does it have)?

The answer is that we will use a pseudo random number generator (伪随机数发生器, namely a Mersenne twister).  But for that, we need to keep its state.  So let us put that inside the `SpaceShooterWidget`:

```C++
  ...
  #include <random>

  strcut Point {
    ...
  };
  ...

  class SpaceShooterWidget :public QWidget {
    ...

  private:
    std::mt19937_64 random;
    ...
  }
```

An in addition, we should initialize the random number generator.  This is also called seeding and needs to be done.  (Some random number generators show odd behavior if you do not seed.  In the better case they always produce the same random number sequence.)  We will do that in the constructor.  Generally, we have 2 options:  Either we seed with the current time (say milliseconds since 1970), or with the cryptographic random source (加密随机源) of the computer.  I have decided for the latter:

```C++
  SpaceShooterWidget::SpaceShooterWidget(QWidget* parent)
    :QWidget(parent), random(random_device()()) {
      ...
  }
```

Why didn't I just write `random_device random;` in the class definition?

Well the random device is supposed to generate random numbers to the best of the computer's capabilities.  But on the other hand it is a bit slow in that.  The pseudo random number generators are much faster in generating many (almost) random numbers.

When you start the program again, and don't move or shoot, you will notice that suddenly bullets occur that move downwards.  This is because we chose the same picture regardless of whether they are shots (`isUp()==true`) or bombs (`!isUp()`).  We can correct that in the paint method as follows:

```C++
  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    ...
    for (auto shot = shots.begin(); shot!=shots.end();) {
      if (shot->isUp())
        p.drawPixmap(scale.px(shot->x), scale.py(shot->y), *this->shot);
      else
        p.drawPixmap(scale.px(shot->x), scale.py(shot->y), *this->bomb);
      bool passes = true;
      ...
    }
    ...
  }

  ...

  QPixmap* loadBomb(QColor& background) {
    auto bomb = new QPixmap(20, 20);
    auto p = QPainter(bomb);
    p.fillRect(bomb->rect(), background);
    p.setFont(QFont("arial", 20));
    p.drawText(0,20, "v");
    p.end();
    return bomb;
  }

  void SpaceShooterWidget::loadImages() {
    ...
    bomb = loadBomb(background);
  }
```

## 3.2 Regenerating Enemy (更生的仇敌)

I can make the game even harder:  Let the enemy regenerate, i.e. rocks re-appear after they have been shot.  For that we need to split the `fillBackground()` into 2 parts:  The first will `generateBackground()` and the original method will just stuff the generated background in this variable.  Once we have done that, we can in every `evolve()`-step check whether any `gaps` occurred (有差距).

```C++
  void SpaceShooterWidget::evolve() {
    ...
    if (state==GAME_OVER) {
      update();
      return;
    }
    ...
        shots.push_back({ enemy.x, 0.65, -0.01 });

    if (flipCoin(random, 0.03)) {
      auto gaps = gernateBackground();
      for (const auto& rock : background)
        gaps.erase(rock);
      if (!gaps.empty()) {
        uint index = random() % gaps.size();
        auto rock = gaps.begin();
        for (uint i=0; i<index; i++)
          ++rock;
        background.push_back(*rock);
      }
    }
    update();
  }

  void MainWindow::fillBackground() {
    background.clear();
    for (auto& rock : generateBackground())
        background.push_back(rock);
  }

  set<Point> MainWindow::generateBackground() const {
    set<Point> background;
    for (int x=-9; x<=9; x++) {
        background.insert({ 0.1*x, 1.0 });
    }
    for (int x=-4; x<=4; x++) {
        background.insert({ 0.2*x, 0.9 });
    }
    for (int x=-2; x<2; x++) {
        background.insert({ 0.4*x+0.2, 0.8 });
    }
    return background;
  }
```

But in order to implement this method, we need to declare it in the `SpaceShooterWidget`:

```C++
  ...
  #include <set>
  ...
  struct Point {
    double x, y;

    bool operator <(const Point& p) const {
      return y<p.y || y==p.y && x<p.x;
    }
  }
  ...

  class SpaceShooterWidget :public QWidget {
    ...
  protected:
    ...
    std::set<Point> generateBackground() const;
    void fillBackground();
    ...
  };
```

The reason why I made it a method (返法，and not a simple static function -- 静态函数) is that in the final part where you are asked to generate the background differently for different levels.  But the `level` (级) will be a property of the `SpaceShooterWidget`.  (Alternatively, we could pass all required variables to this function.)

Note that in C++ sets are always ordered (有序集).  But for that you need to define the `operator <`.  If you make this operator a method (inside the class/struct defintion), then you only pass one additional argument.  The other argument is the current object (`this`).  Of course, a comparison does not change any object, so it receives a const reference to the other point and also treats the current object a const.

Now, if you try it again, you will notice that it is pretty hard to win.


# 9. Try for Yourself
Now it is your turn.  When you typed in the whole program without errors, then it should be startable and you should be able to navigate the space ship, shoot, hit the rocks.  But also the rocks will drop bombs on you.

## 9.0 Does your Program work?

If it does not compile, have a look at the compiler error message(s).  Can you understand where the error happens?  What does it complain about?  Compare to the program here and try to fix the problem(s).

## 9.1 Design a Second Level

As we noted, it is now pretty hard to win against the computer.  You could design an easier first level and then make the game harder and harder with every higher level.  In order to adapt to the level, you need to make it a (private) member variable of the `SpaceShooterWidget`.  Initialize it to `1` and reset it to `1`.

You may wish to adjust some things depending on this level, e.g. you could skip generating some of the background in the first level.  Or you could adjust the `bias`es for flipping the coins, e.g. multiply by `sqrt(level)`.  That means that the second level is 41% harder, the 3rd level 68% and the 4th level double as hard as the 1st level.

Of course, you will also need to increase the level, once the current level is won.  You can do that similar to the switch from `EXPLODING` to `GAME_OVER`:

```C++
  void SpaceShooterWidget::paintEvent(QPaintEvent*) {
    ...
    switch (state) {
      ...
      case WINNING:
        if (countDown>0)
          countDown--;
        else
          increaseLevel();
    }
  }

  void SpaceShooterWidget::increaseLevel() {
    level++;
    ...
  }
```


## 9.2 Design better Images

Currently, we are just using a couple of letters and some special characters in order to represent the elements of the picture (ship, rock, shot, bomb, explosion).  You could design 20x20 jpg pictures in your favorite graphics program, store them together with the written program, and then load the jpg-files instead of generating a QPixmap.  A useful sample will be:

```C++
  ship = new QPixmap("./ship.jpg");
``` 

Basically, you can replace the corresponding line in the `loadImages()`-method with this statement.  (You need to put the "ship.jpg" next to the "\*.exe" file.)


Please enjoy trying around with the program.  If the program does not do what you expected, just restore the old values and start from there again.
