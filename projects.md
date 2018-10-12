---
layout: page
title: Projects
---

### Processing Tetris ###

[ProcessingTetris](https://github.com/dustinroepsch/ProcessingTetris) started off as a challenge for myself, I decided to record a timelapse of me making Tetris, with the goal being to create a working game as quickly as possible. I used `Java` with the `Processing` library, and the implementation took about three hours total.

<iframe width="560" height="315" src="https://www.youtube.com/embed/PCnav51ldYk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Overall this project was a lot of fun, especially with the time pressure. I full inted to do something like this again in the future. 

### Korruct ###

[Korruct](https://github.com/dustinroepsch/Korruct/blob/master/README.md) is a simple Spell Correcting library for Java based off of [this post](http://norvig.com/spell-correct.html) by Peter Norvig.

This library provides a simple interface for performing spell correction in Java. First, create a `Keyboard` that represents all of the valid characters in your language with their relative distances, and a text file that contains all the valid words for your language. These distances will be used as the heuristic by which tiebreakers are resolved when the algorithm attempts to spell correct a word. For convenience I provide an implementation for standard US keyboards called `QWERTYKeboard`.

```java
public class ExampleUsage {
  public static void main(String[] args) throws IOException {
    Keyboard keyboard = new QWERTYKeyboard();

    // Create a spellCorrector with a US English (QWERTY) keyboard and a list of english words.
    SpellCorrector spellCorrector =
        new SpellCorrector(FileSystems.getDefault().getPath("common_english_words.txt"), keyboard);

    Scanner in = new Scanner(System.in);

    while (true) {
      System.out.println("Enter a word:");
      String inputWord = in.next();
      if (inputWord.trim().equals("q")) {
        break;
      }

      // Attempt to correct the input word
      Optional<String> correction = spellCorrector.getCorrection(inputWord);
      System.out.println(correction.map(s -> "Did you mean: " + s).orElse("No typo!"));
    }
  }
}
```

### PPM185 ##

[PPM185](https://github.com/dustinroepsch/PPM185) is simple a `C` library for image manipulation. I designed this library with the intention that it would be simple enough for students taking Cpre185 (intro to c) at Iowa State University would be able to use it without too much struggle. Unfortunately, this code hasn't become an official part of the class, however it has proven to be useful for when a student asks for an easy way to create images. 

# Example #

Here is an example of using the library to generate the famouse barnsley fern.

# Code #

```c
#include "../ppm185.h"

#include <math.h>
#include <time.h>
#include <stdlib.h>

#define IMAGE_WIDTH 500
#define IMAGE_HEIGHT (IMAGE_WIDTH * 2)
#define NUM_PIXELS (IMAGE_HEIGHT * IMAGE_WIDTH)
#define NUM_ITERATIONS 10000000

#define X_WINDOW_MIN -2.1820
#define X_WINDOW_MAX 2.6558
#define Y_WINDOW_MIN 0
#define Y_WINDOW_MAX 9.9983

/*
* Input: double value in range [oldMin, oldMax]
* Output: double in range [newMin, newMax]
*/
double scale(double value, double oldMin, double oldMax, double newMin, double newMax)
{
    return ((newMax - newMin) / (oldMax - oldMin)) * (value - oldMin) + newMin;
}

void f1(double *x, double *y)
{
    *x = 0;
    *y = *y * 0.16;
}

void f2(double *x, double *y)
{
    double old_x = *x;
    double old_y = *y;
    *x = 0.85 * old_x + 0.04 * old_y;
    *y = -0.04 * old_x + 0.85 * old_y + 1.6;
}

void f3(double *x, double *y)
{
    double old_x = *x;
    double old_y = *y;

    *x = 0.2 * old_x - 0.26 * old_y;
    *y = 0.23 * old_x + 0.22 * old_y + 1.6;
}

void f4(double *x, double *y)
{
    double old_x = *x;
    double old_y = *y;

    *x = -0.15 * old_x + 0.28 * old_y;
    *y = 0.26 * old_x + 0.24 * old_y + 0.44;
}

void mutate_point(double *x, double *y)
{
    int roll = rand() % 100;

    if (roll < 1) //1% chance
    {
        f1(x, y);
    }
    else if (roll < 85) //85% chance
    {
        f2(x, y);
    }
    else if (roll < 85 + 7) //7% chance
    {
        f3(x, y);
    }
    else
    {
        f4(x, y);
    }
}

void calc_sd(int *arr, double *mean, double *sd)
{
    double sum = 0;
    double sum_squared = 0;

    for (int i = 0; i < NUM_PIXELS; i++)
    {
        sum += arr[i];
        sum_squared += arr[i] * arr[i];
    }

    *mean = sum / NUM_PIXELS;
    *sd = sqrt(NUM_PIXELS * sum_squared - sum * sum) / NUM_PIXELS;
}

int main()
{
    srand(time(NULL));

    //this array is big so I used calloc to not blow up the stack (with the bonus of initializing every element of the array to zero)
    int *histogram = calloc(NUM_PIXELS, sizeof(int));

    double x = 0;
    double y = 0;

    for (int i = 0; i < NUM_ITERATIONS; i++)
    {
        mutate_point(&x, &y);
        int screen_x = scale(x, X_WINDOW_MIN, X_WINDOW_MAX, 0, IMAGE_WIDTH);
        int screen_y = scale(y, Y_WINDOW_MIN, Y_WINDOW_MAX, 0, IMAGE_HEIGHT);

        histogram[screen_y * IMAGE_WIDTH + screen_x] += 1;
    }

    double mean, sd;
    calc_sd(histogram, &mean, &sd);

    double gradient_min = mean - sd;
    double gradient_max = mean + sd;

    ppm_image_t *output_image = ppm_create(IMAGE_WIDTH, IMAGE_HEIGHT);

    for (int row = 0; row < IMAGE_HEIGHT; row++)
    {
        for (int col = 0; col < IMAGE_WIDTH; col++)
        {
            int green = histogram[row * IMAGE_WIDTH + col];

            if (green == 0)
                continue;

            if (green < gradient_min)
            {
                green = gradient_min;
            }
            if (green > gradient_max)
            {
                green = gradient_max;
            }
            ppm_set_green(output_image, IMAGE_HEIGHT - row - 1, col, scale(green, gradient_min, gradient_max, 0, 255));
        }
    }

    ppm_write_to_file(output_image, "barnsley.ppm");

    //cleanup
    free(histogram);
    ppm_destory(&output_image);
    return 0;
}

```

# Output #

![barnsley fern](/assets/images/barnsley.png){:width="350px"}

