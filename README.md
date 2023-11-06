### First_Assignment of Research_Track1
In this report, according to the goal requested in the assignment, it is that the robot must pick up the golden tokens that are in the desired position and place them in a certain environment, which in this program, we use the middle position of the screen as the goal. We consider the final to collect tokens.as you can see in the picture below, the first figure show the initial position of the robot and tokens, and the second figure shows the final position. 

This is the first position of the robot relative of the gold tokens in simulator.

![](/sr/robot1.png)

This is the second position of the robot relative of the gold tokens in simulator.

![](/sr/robot2.png)

Python Robotics Simulator
================================

This is a simple, portable robot simulator developed by [Student Robotics](https://studentrobotics.org).
Some of the arenas and the exercises have been modified for the Research Track I course

Installing and running
----------------------

The simulator requires a Python 2.7 installation, the [pygame](http://pygame.org/) library, [PyPyBox2D](https://pypi.python.org/pypi/pypybox2d/2.1-r331), and [PyYAML](https://pypi.python.org/pypi/PyYAML/).

Pygame, unfortunately, can be tricky (though [not impossible](http://askubuntu.com/q/312767)) to install in virtual environments. If you are using `pip`, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use [Portable Python](http://portablepython.com/). PyPyBox2D and PyYAML are more forgiving, and should install just fine using `pip` or `easy_install`.

## Troubleshooting

When running `python run.py <file>`, you may be presented with an error: `ImportError: No module named 'robot'`. This may be due to a conflict between sr.tools and sr.robot. To resolve, symlink simulator/sr/robot to the location of sr.tools.

On Ubuntu, this can be accomplished by:
* Find the location of srtools: `pip show sr.tools`
* Get the location. In my case this was `/usr/local/lib/python2.7/dist-packages`
* Create symlink: `ln -s path/to/simulator/sr/robot /usr/local/lib/python2.7/dist-packages/sr/`

## Exercise
-----------------------------

To run one or more scripts in the simulator, use `run.py`, passing it the file names. 

I am proposing you three exercises, with an increasing level of difficulty.
The instruction for the three exercises can be found inside the .py files (exercise1.py, exercise2.py, exercise3.py).

When done, you can run the program with:

```bash
$ python run.py exercise1.py
```

You have also the solutions of the exercises (folder solutions)

```bash
$ python run.py solutions/exercise1_solution.py
```

Robot API
---------

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```

The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:
  * `code`: the numeric code of the marker.
  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).
  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.
* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:
  * `length`: the distance from the centre of the robot to the object (in metres).
  * `rot_y`: rotation about the Y axis in degrees.
* `dist`: an alias for `centre.length`
* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.
* `rot_y`: an alias for `centre.rot_y`
* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```

### Parameter definition ###

According to the goal of the robot, in this program, two parameters must be defined as thresholds. Parameter a_th is considered as orientation and parameter d_th as distance threshold. The purpose of defining this parameter is for the robot to know when and at what angle it is close to the target so that it can grab it.
```python
a_th = 2.0
""" float: Threshold for the control of the orientation"""

d_th = 0.4
""" float: Threshold for the control of the linear distance"""
```

### drive() ###

The drive function is a function to define the movement of the robot, which allows the robot to move directly forward or backward.This function takes two parameters (speed, seconds). We can give the robot the desired linear velocity per second. So we write as follows:

```python
def drive(speed, seconds):
    """
    Function for setting a linear velocity

    Args: speed (int): the speed of the wheels
          seconds (int): the time interval
    """
    R.motors[0].m0.power = speed
    R.motors[0].m1.power = speed
    time.sleep(seconds)
    R.motors[0].m0.power = 0
    R.motors[0].m1.power = 0
```

### turn() ###

This function defines the rotation of the robot, which allows the robot to rotate around itself.When we give this function the right value, the rotation is defined in the specified time.ike the previous function, this function has two parameters (speed, seconds) which means will rotate with the amount of speed and time.

```python
def turn(speed, seconds):
    """
    Function for setting an angular velocity

    Args: speed (int): the speed of the wheels
          seconds (int): the time interval
    """
    R.motors[0].m0.power = speed
    R.motors[0].m1.power = -speed
    time.sleep(seconds)
    R.motors[0].m0.power = 0
    R.motors[0].m1.power = 0
```

### find_token_gold() ###

The purpose of this function definition is to actually find the closest golden token. In this program, we presentation the distance with the variable (Distance) and the rotation angle with the variable (Rotation).In this function, we consider the range of the robot's vision to be 100 and define it in the program in such a way that if the distance of the golden token is less than 100, then the robot will find the closest token and if there is no token within 100 distance, Run this program again.

```python
  Distance = 100
    for a in R.see():
        if a.dist < Distance and a.info.marker_type == MARKER_TOKEN_GOLD:
            Distance = a.dist
            Rotation = a.rot_y
    if Distance == 100:
        return -1, -1
    else:
        return Distance, Rotation
```

### main ###

In this part, we have to describe the main code of the program according to the definition of functions. We have to write using the command (while) that the robot will put all the golden tokens in the middle position, in a certain place. As mentioned in the previous function, the bot should find the closest token.So, we should write the program in such a way that if the token is not present in the desired distance, then it rotates a little and finds the golden token again according to the defined function.

```python
  Index = 1

while Index:       
  Distance, Rotation = find_token_gold()
  while Distance == -1:
      print("No tokens found!!")
      turn(5,0.5)
      Distance, Rotation = find_token_gold()
```

First of all we need to consider several modes in the code:
The first case is to grab the token if the golden token distance was less than the defined distance threshold. The second case is that if the robot orientation was between (-2) and (+2), which means that the orientation between these two numbers was the threshold, it would move towards the golden token.the third mode is that if the robot is not at the right angle from the golden token, it will turn a little to the left and right.
The command`R.grab()` , when the robot gets close to the golden token, grab the token.
The command`R.release()` ,when the robot grap it the tokens in the right position, then release it.

```python
    if Distance < d_th:
      print("Found it!")
      R.grab()  # if we are close to the token, we grab it.
      print("Gotcha!")
      Index = 0
  elif -a_th <= Rotation <= a_th:  # if the robot is well aligned with the token, we go forward
      print("Ah, here we are!.")
      drive(10, 0.8)
  elif Rotation < -a_th:  # if the robot is not well aligned with the token, we move it on the left or on the right
      print("Left a bit...")
      turn(-2, 0.5)
  elif Rotation > a_th:
      print("Right a bit...")
      turn(+2, 0.5)
turn(-8,1.5)
drive(30,6)
R.release()
drive(-20,1.5)
turn(-10,4)
drive(20,2) 
```

