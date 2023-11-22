# Gravitationally bound objects in vacuum

## Goals

This project explores how objects with a mass and gravitational bounding behave for different accuracies of time steps and how this influences overall behavior in three-dimensional space. While two object problems can be calculated symbolicaly this remains a gap for 3 or more objects.

Hence, simulation is a major way of exploring these types of problems by approaching possible outcomes, may they be convergent or divergent, through experimenting with modelling parameters. Additionally, it provides a path to visually representing the model, and, thus, helping our intuition to become familiar with these problem types.


### First part
As a starting point two objects with a defined mass shall be simulated, each with a start velocity.
1. No visual output necessary. Command line output with position and velocity at the given time shall be sufficient. One cycle is ok for this part.  
2. Implement a recurring cycle for an undefined number of timesteps that can run automatically.  

### Second part
In the second part the simulation shall be extended:
1. Implement mechanism to run simulation for an arbitrary number of objects
2. Visual representation at runtime shall be implemented. Python libs for efficient representation shall be explored and used.

## Theory

Gravitational force is calculated using the formula:
$$ F = {{G m_1 m_2} \over {r^2}} $$

It calculates the force between two objects that have a mass and are gravitationally bound. The force vector directs toward the other object. This is true for two object problems.

With three or more objects it is necessary to calculate the resulting force vector of the gravitational force that each object is putting on the object in discussion.

This formula however, does not incorporate space dimensions and calculates a scalar value of the unit newtown.
To extend its meaning in the realm of space dimensions we introduce the following vectorized form:

$$ \vec{F_{21}} = -{{{G m_1 m_2} \over {\lvert{\vec{r_{21}^2}}\rvert}} \vec{r_{e21}}} $$

$ {\lvert{\vec{r_{21}}}\rvert} = \lvert r_2 - r_1 \rvert $ is the distance between the two objects.

$ \vec{r_{e21}} = {{r_2 - r_1} \over {\lvert r_2 - r_1 \rvert}} $ is the unit vector from object 1 to object 2; scaled to length 1; is scalar



Source: https://en.wikipedia.org/wiki/Newton%27s_law_of_universal_gravitation


## Implementation

## First part

### Defining the class and update schema
To account for the recurrence and comaprability of each object - that obviously share common properties - I've introduced the class _gravitationalObject_. It defines an object that has a mass, a position vector in 3D, a force vector in 3D that it experiences and a velocity vector in 3D.

Initially, I thought it would make sense to have dedicated properties for position and velocity that account for the step wise calculation of new position and velocity. However, it turned out, that this is not necessary, because every object only needs to know the force it currently experiences due to its gravitational effect with other objects and upon the update trigger can directly update the current values for velocity and position for the next step.

Update cycle for objects:
1. Calculate experienced force for each object and write to property (T = 0)
2. Calculate velocity and position for each object and write to property (T = 1)
3. Calculate experienced force ... (T = 1)
4. Calculate velocity and position ... (T = 2)  
(repeat)

### The math and physics behind
The above sequence already incorporates some key ideas of the math and physics that is involved (see __Theory__). Two masses that are in a vacuum, thus, do not experience friction or any other kind of force are governed only by the interaction between the objects. As these objects do not have an electric or magnetic property, gravitation remains as the only effect that needs consideration.

Hence, it is sufficient to know the force of attraction due to gravitation to calculate the next position and velocity of each object for an indefinitely small step size.

This is where the major constraint of this discrete approach comes from. We must assume that the calculated force remains static between simulation steps as we are obviously not able to calculate indefinitely small slices of time.

To account for this limitation the step size is a parameter of this model to be fine tuned. It should be as small as possible while keeping computational costs affordable.

Here is a thought experiment to demonstrate the limitation of this approach vs. actual physics:
1. A small mass object is standing still in space and there is another significantly larger mass object that is also present. We assume the large mass is fixed and does not move.

2. We first calculate the force that the large mass object excerts on the small mass object. We know that the force is pulling the small mass and would accelerate it, but at T=0 the object is not moving.

3. At T=1 we expect that the small mass object should have moved. To calculate a new position we must know the speed at any given time between T=0 and T=1 and must calculate its integral to derive its new position.

4. In this approach however we assume that the calculated speed at T=1 based on the force from T=0 is static.

5. With this the position at T=1 is calculated using a constant speed between T=0 and T=1, which results in a longer distance traveled than it would in real world.

6. Acceleration is also assumed to be constant, which is not true as shorter distances between the objects results in higher force, and, thus, in higher acceleration.

These shortcomings ideally require a form of compensation, that involves computational cost somewhere between shortening the simulation step size significantly and simple linear correction functions. Hence, it would yield an efficient, yet good enough estimation of what happens between the steps.

In this simulation we just assume that small enough step sizes yield convergent and somewhat realistic results.

### Defining the program structure
For the start, I only focus on two objects. To account for more objects that will be introduced in a later step I register every newly instantiated object of the class _gravitationalObject_ with _objectsList_ that holds all objects under consideration.

The user is prompted in the console to provide the number of simulation steps to be performed as well as the step size in seconds.

The simulation steps define the number of iterations for the loop and the step size is used in calculating the position and velocity at the next step.

### Visualizing the result
To visualize and verify the correctness of the approach _matplotlib_ is used. In this step I employ a 3-dimensional plot that draws calculated positions. Results can be verified by adding annotations based on the respective step that was calculated. This helps to identify into which direction the movement of the objects is taking place.