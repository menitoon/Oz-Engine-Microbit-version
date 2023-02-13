![](https://github.com/menitoon/Oz-Engine-Microbit-version/blob/main/Logo%20x%20microbit.png?raw=true)
A version of Oz-Engine made for the Microbit card.
You can use it like the regular version [here](https://github.com/menitoon/Oz-Engine-Rebooted)
all you have to do is copy-paste the code in the [Python-Microbit-Editor](https://python.microbit.org/)

but please note some changes:

## -No **prefix**  and **import** needed
while ``import math`` and ``from microbit import *`` are needed
``ìmport OzEngine as oz`` is not possible since the Microbit card can only work with one file so the prefix **oz** is not needed.


## -**Rendering** on the Microbit card
Instead of using `` print(camera.render()) ``
you will need to do so ``camera.send_light_update()``

## -**String** being invalid

Giving a string as parameter of ``char`` for the sprite isn't possible instead, you will have to use **integers** that are between **0** and **9**.



## Making a game : Snake 🐍

With the microbit card, you can detect button input without blocking(pausing) which is useful if we want to make
games like **Snake**

## Making the **Gameloop** 🔄

I will explain how the gameloop works and then make all function that are used in it.
```python

def game():

  global is_win
  global alive
  global camera
  global canvas
  global snake_body
  global direction
  global score
  global latency

  #amount of time a frame lasts
  latency = 600.0

  # Instance canvas
  canvas = Canvas(0)
  # Instance camera
  camera = Camera(canvas, [5, 5], [0, 0], "camera")
  snake_body = []

  
  # Instance Head snake
  head = Sprite(canvas, 5, [2, 3], "head", group="snake")
  snake_body.append(head)

  #Instance Body snake
  body = Sprite(canvas, 3, [1, 3], "body", group="snake")
  snake_body.append(body)


  #set direction
  direction = [1, 0]
  spawn_apple(4, 4) # spawns a apple in a range of 4x4


  time_passed = running_time()
  score = 0
  has_pressed = False         # whetever the button was released or not
  is_win = False
  alive = True

  #Direction of one frame before
  direction_input = direction

  camera.send_light_update()

  #Block until button "a" or "b" is pressed
  while not (button_a.is_pressed() == True or button_b.is_pressed() == True):
      pass
      
  has_pressed = True # say that we are pressing something
  
  while alive:

    #if hasn't pressed yet
    if has_pressed == False:

      # if button a is pressed
      if button_a.is_pressed():
        #direction equals to the direction one frame earlier
        direction = direction_input
        #change direction
        direction = change_direction(-1)
        #say we pressed
        has_pressed = True

      elif button_b.is_pressed():
        #direction equals to the direction one frame earlier
        direction = direction_input
        #change direction
        direction = change_direction(1)
        #say we pressed
        has_pressed = True

    #if no button is pressed and just released 
    elif button_a.is_pressed() == False and button_b.is_pressed() == False:
      # say we haven't pressed something
      has_pressed = False

    if (running_time() - time_passed) > latency: #if the time is higher then "latency" we 
                                                 #update the game

      direction_input = direction #direction of 1 frame before set to current direction
      time_passed = running_time() #update time_passed to "running_time()"
      move(direction[0], direction[1]) #move the snake by the direction 
      check_death()                    # check if we are off-screen or hit ourself
      
      APPLE = canvas.get_sprite("apple") #get reference of "apple"

      #Apple animation
      if APPLE.char == 7:
        APPLE.char = 5

      else:
        APPLE.char = 7

      #update screen
      camera.send_light_update()

  #death animation
  for i in range(4):
    display.off()
    del snake_body[0]
    camera.send_light_update()
    time.sleep(0.15)
    display.on()
    time.sleep(0.15)

  #classic score message 
  display.scroll("Score: " +
                 str(score)) if not is_win else display.scroll(
                   "You won, you have the perfect score of " + str(score))
  game() #restarts the game


```
At the top we are making all of these variable ``global`` so we can access them through other functions.


## Move 🏃

Let's make that little snake move :
````python
def move(x, y):
  #to be able to acces these
  global score
  global latency

  #say whetever we collided with an apple
  is_point = False
  #gets "head" sprite reference
  HEAD = canvas.get_sprite("head")

  #gets the groups that are colliding with "head"
  colliding_groups = HEAD.get_colliding_groups()

  #if any collision
  if len(colliding_groups) > 0:

    #if one of them was a collectable(an apple)
    if "collectable" in colliding_groups:
      #destroy apple and spawn a new one
      canvas.get_sprite("apple").destroy()
      #reduce the amount of time a frame last, in other therms makes the game faster
      latency *= 0.95 
      is_point = True
      score += 1

  else:
    #destroy tail 
    TAIL = snake_body[0]
    snake_body.remove(TAIL)
    TAIL.destroy()
  
  #rename head as "body"
  OLD_HEAD = snake_body[-1]
  OLD_HEAD.char = 3
  OLD_HEAD.rename("body")

  #create new head at desired position
  head = Sprite(canvas,
                5, [OLD_HEAD.position[0] + x, OLD_HEAD.position[1] + y],
                "head",
                group="snake")
  snake_body.append(head)
  if is_point: #if we collided with an apple
    if score == 23: #if perfect score then win
      is_win = True
      is_alive = False #we kill the gameloop process
    else:
      spawn_apple(4, 4) #if no perfect score then we keep spawning apple
````


## Changing Direction ↩️

Now let's make the code to change the direction of the snake:

````python
def change_direction(increment):

  avaible_directions = [[1, 0], [0, 1], [-1, 0], [0, -1]]

  index: list = avaible_directions.index(direction)
  index += increment

  if index > 3: #if index is bigger than 3 set it back to the first one (0)

    index = 0

  elif index < 0: #if index is smaller than 0 set it back to the last element (3)
    index = 3

  return avaible_directions[index]
````

## Spawning Apples 🍎

Before even spawning apples we need to define where theses can spawn
````python
def get_possible_spawn_points(range_x, range_y):
  
  taken_pos = canvas.sprite_position_dict.values()
  positions = []
  for y in range(range_y):
    for x in range(range_x):
      if not [x, y] in taken_pos: # if position doesn't exist then append to list  ``positions``
        positions.append([x, y])
  return positions
````
Now we can make the spawning function but first make sure you ```random``` is imported at the top like so ``import random as rng``:
````python
def spawn_apple(range_x, range_y):

  #gets possble spawn position that aren't taken yet.
  print(get_possible_spawn_points(range_x, range_y))
  spawn_position = rng.choice(get_possible_spawn_points(range_x, range_y))
  
  apple = Sprite(canvas, "7", spawn_position, "apple", group="collectable")
````


## Detecting **death** 💀

Now we need a way to check if we collided in a part of our snake or in the border so let's do that:
````python
def check_death():

  global alive 

  position = canvas.get_sprite("head").position #gets the reference of "head" from that gets it's position

  object_colliding = canvas.get_sprite("head").get_colliding_groups() #get groups that are colliding with "head"
  if "snake" in object_colliding:
    alive = False 

  #if off-screen die
  if position[0] == -1 or position[0] == 5 or position[1] == -1 or position[
      1] == 5:
    alive = False
````

## Executing the **game** 🕹️

The final step is very simple just execute the ``game()`` function:

````python
game()
````

And you're done !
In case you have an Issue [here]() is the full code





_Oz-Engine is not affialted with Microbit_
