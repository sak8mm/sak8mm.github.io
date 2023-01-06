---
layout: post
title: Trying to guess your number in a select range!
image: "/posts/guess_number.png"
tags: [Python, Numbers, Guess, Binary Search]
---

In this post I'm going to run through a function that tries to guess your number in a specific number range the user picks with no more than 10 tries. The 'tries' is dependent on the user's number range, but it is no more than 10.

Let's get into it!

---
I am going to first import two python packages for this:

```python
import os
import time as t
```
I will use the **os** python package because it's for the operating system that will allow use to exit out of the program at a certain point. Also, I will use the **time** python package toward the end of the guessing game for a delay feature.

The first two parts to build before we can start working on the guessing is the min and max value function the user will input for the number range to guess. These functions will be similar so we will go over the min_number() function.

The important part when it comes to a user's input is we want it to be usable for our program. So to do this, we put the user's input with a try/except statement and int function. So that way if a user puts '1' vice 1 or 'hello' vice 1 they will be notified their input must be an integer. Obviously, if the user does it correctly then the program will move on to the next step.

The following is what happens if the user types hello vice an integer:

```python
>>> try:
...     min_n = int(input('Pick a min whole number: '))
... except:
...     print('You must enter an integer!')
...
Pick a min whole number: hello
You must enter an integer!
```
Perfect, that is working how we want. Now we just need to get this program to ensure the min value is NOT less than 0 and return the value entered when called in another function. 

To do this we will use an if statement in the following manner:

```python
max_n = -2 
if max_n < 0:
...     print('The number must be >= 0!')
... else:
...     return max_n
...
The number must be >= 0!
```
Ok, awesome, that is working like we wanted. Lets just add some more details to wrap this function up. We are going to add a while loop at the beginning of this function. This will allow the program to keep running for the user's input until everything is correct and then it will stop.

```python
def min_number():
    """User establishes min number a range in the guessing game."""
    while True:
        try:
            min_n = int(input('Pick a min whole number: '))
        except:
            print('You must enter an integer!')
            continue

        if min_n < 0:
            print('The number must be >= 0!')
            continue
        else:
            return min_n
            break
```
The above is the min_number() function that will have the user input their min value. When this function is ran, it will ask for the user's input. Once the input is retained with no issues, our while loop will break and the program will run to the next step which is the max value.

The max value function is similar and is as follows:

```python
def max_number():
    """User establishes max number a range in the guessing game."""
    while True:
        try:
            max_n = int(input('Pick a max whole number: '))
        except:
            print('You must enter an integer!')
            continue                
        if max_n < 0:
            print('The number must be >= 0!')
            continue
        else:
            return max_n
            break
```
The next function we are going to create will take the min_number() and max_number() function we created and start guessing the user's number. This function does not take the user's number they are thinking so it does rely on the user's honesty. Of course we could always change that if we wanted.

Couple of factors that was used to determine this function:


1) Limit number of guesses the program gets based on min and max value, but no more than 10. We don't want to be here ALL DAY!

2) We are going to have the program do a **binary search** to guess the number. This is where user honesty is important. Basically, the program will guess the mid point of the number range and ask if that is the user's number. If not, then the program will ask if its guess is higher or lower than the user's number. From there the program will drill down to the user's number by using mid points if it has enough tries to do so.

3) So with that, we want to ensure the number range the user inputs makes sense so that will be part of our function. Meaning the min value should not be higher than the max value.

4) We will keep track of our guesses to ensure we are not repeating ourselves. We also need to count the number of tries so when we get it correct, we can let the user know how many times it took us.

When we put all those factors into play with a series of while loops and if statements, we get the following:

```python
def guess_num(min_val, max_val):
    """Takes the min and max value given to establish range criteria and starts to guess user's number."""
    tries = (max_val - min_val) // 10
    if tries > 10:
        tries = 10
    elif tries == 0:
        tries = 1
    else:
        pass
    cnt = 0
    print(f'Ok, I will now get {tries} guess(es) in order to guess the right number')
    guess_list = []
    
    while tries > 0:
        guess = (min_val + max_val) // 2
        if guess in guess_list:
            print(f'It looks like I have already guessed {guess}!')
            response = input('Do you want to continue? [type y or n as a response] ')
            if response == 'n':
                os._exit(1)
        else:
            guess_list.append(guess)

        reply = input(f"Is your number {guess} [type y or n as a response]? ")
        cnt += 1
        tries -= 1
        if reply == 'y' and cnt == 1:
            print('That was way too easy partner!')
            break
        elif reply == 'y':
            print('Awesome, I got it correct')
            break
        else:
            if tries != 0:
                h_l = input(f"Is your number higer or lower than {guess} [type l or h as a response]? ")
                if h_l == 'h':
                    min_val = guess + 1
                else:
                    max_val = guess

    if tries == 0 and reply == 'n':
        print(f'I ran out of tries so you won this go around!')
    elif cnt == 1:
        print(f'I got it in {tries - (tries - 1)} try!')
    else:
        print(f'I got it in {cnt} tries!')
    
while True:
    min_val = min_number()
    max_val = max_number()
    
    if max_val <= min_val:
        print('Your range of numbers makes no sense; please try again!')
        continue
    else:
        break

guess_num(min_val, max_val)
t.sleep(5)
```

***Break down by pieces:***

First:

1) Sets the **tries** limit for the program based off the number range, but no more than 10 and the program gets at least 1 guess if number range is small.

2) It also notifies the user how many **tries** the program will get and create a guest_list that is empty to hold guess(es) to ensure it does not repeat itself.

```python
tries = (max_val - min_val) // 10
    if tries > 10:
        tries = 10
    elif tries == 0:
        tries = 1
    else:
        pass
    cnt = 0
    print(f'Ok, I will now get {tries} guess(es) in order to guess the right number')
    guess_list = []
```
Second:

1) This portion checks if the number has already been guessed by looking at our guess list we created. With each guessed number, it will be stored in that list for record keeping. If the program already guesses the number, it will ask the user if they want to continue.

```python
while tries > 0:
        guess = (min_val + max_val) // 2
        if guess in guess_list:
            print(f'It looks like I have already guessed {guess}!')
            response = input('Do you want to continue? [type y or n as a response]')
            if response == 'n':
                os._exit(1)
        else:
            guess_list.append(guess)
```

Third:

1) This part actully renders a guess. Based on the user's response, the progam will either get the correct answer or ask if the number is higher or lower than the guess.

2) Also, this part keeps track on the **tries** and **counts** during this process.

```python
reply = input(f"Is your number {guess} [type y or n as a response]? ")
        cnt += 1
        tries -= 1
        if reply == 'y' and cnt == 1:
            print('That was way too easy partner!')
            break
        elif reply == 'y':
            print('Awesome, I got it correct')
            break
        else:
            if tries != 0:
                h_l = input(f"Is your number higer or lower than {guess} [type l or h as a response]? ")
                if h_l == 'h':
                    min_val = guess + 1
                else:
                    max_val = guess
```
Forth:

1) This portion renders the appropriate response if the program ran out of tries or guessed correct with a response in the number of tries it took the program.

```python
if tries == 0 and reply == 'n':
    print(f'I ran out of tries so you won this go around!')
elif cnt == 1:
    print(f'I got it in {tries - (tries - 1)} try!')
else:
    print(f'I got it in {cnt} tries!')
```

Fifth:

1) This part is not part of the guess_num() function but is part of the program. This part checks if your min and max values truly are correct.

```python
while True:
    min_val = min_number()
    max_val = max_number()
    
    if max_val <= min_val:
        print('Your range of numbers makes no sense; please try again!')
        continue
    else:
        break
```
Sixth:

1) The guess_num() function calls the min_number() and max_number() function and begins the game. Once the program ends, it will wait 5 seconds and then terminate.

```python
guess_num(min_val, max_val)
t.sleep(5)
```

Let's do a couple of test runs!

My number is 13:

```python
Pick a min whole number: 1
Pick a max whole number: 100
Ok, I will now get 9 guesses in order to guess the right number
Is your number 50 [type y or n as a response]? n
Is your number higer or lower than 50 [type l or h as a response]? l
Is your number 25 [type y or n as a response]? n
Is your number higer or lower than 25 [type l or h as a response]? l
Is your number 13 [type y or n as a response]? y
Awesome, I got it correct
I got it in 3 tries!
```
My number is now 100:

```python
Pick a min whole number: 1
Pick a max whole number: 200
Ok, I will now get 10 guesses in order to guess the right number
Is your number 100 [type y or n as a response]? y
That was way too easy partner!
I got it in 1 try!
```
My number is now 1:

```python
Pick a min whole number: 1
Pick a max whole number: 10
Ok, I will now get 1 guess(es) in order to guess the right number
Is your number 5 [type y or n as a response]? n
I ran out of tries so you won this go around!
```

I will not have a number:
```python
Pick a min whole number: 1
Pick a max whole number: 100
Ok, I will now get 9 guess(es) in order to guess the right number
Is your number 50 [type y or n as a response]? n
Is your number higer or lower than 50 [type l or h as a response]? l
Is your number 25 [type y or n as a response]? n
Is your number higer or lower than 25 [type l or h as a response]? l
Is your number 13 [type y or n as a response]? n
Is your number higer or lower than 13 [type l or h as a response]? l
Is your number 7 [type y or n as a response]? n
Is your number higer or lower than 7 [type l or h as a response]? l
Is your number 4 [type y or n as a response]? n
Is your number higer or lower than 4 [type l or h as a response]? l
Is your number 2 [type y or n as a response]? n
Is your number higer or lower than 2 [type l or h as a response]? l
Is your number 1 [type y or n as a response]? n
Is your number higer or lower than 1 [type l or h as a response]? l
It looks like I have already guessed 1!
Do you want to continue? [type y or n as a response]n
```

Lastly, lets check if it can do the min and max value correctly by using 10 as a min value and 5 as a max value:

```python
Pick a min whole number: 10
Pick a max whole number? 5
Your range of numbers makes no sense; please try again!
Pick a min whole number:
```
Perfect, everything looks like it is working as expected! Now we have a fun little guessing program we can play with, how cool is that!

**Last thoughts**
There are several things we can do with this program to improve, make less error prone, etc., but here are some of my thoughts to what I would do:

1) Combine the min_number() and max_number() functions into one for less code clutter.

2) Have the user input thier number in the program to ensure itegrity with a response check.

3) Right now with the user input for higher or lower it depends on the user inputting the correct response. I would make that part less error prone.

With a program like this you could go many different directions your imagination allows, but this is one take to a fun game. I hope you enjoyed it!!!

