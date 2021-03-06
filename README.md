# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---
## PROJECT DESCRIPTION

The controller implemented in this project is a non-linear model-predictive controller that steers a car around the track in [Udacity's simulator](https://github.com/udacity/self-driving-car-sim/releases). The simulator provides the present state of the car i.e the position, speed and heading. It also provides a set of coordinates that act as way-points. All the coordinates are in global coordinate system. The maximum achieved speed in this project is 80mph.

## MPC Model

The controller is implemented as a 6 state - 2 actuator kinematic model. The following equations describe the model:

      X Position --> x(t+1) = x0 + v0 * cos(psi0) * dt;
      Y Position --> y(t+1) = y0 + v0 * sin(psi0) * dt;
      Heading    --> psi(t+1) = psi0 + (v0*delta0*dt/Lf);
      Velocity   --> v(t+1) = v0 + a0*dt ;
      Cross-track-err --> cte(t+1) = f(x0) - y0 + (v0*sin(epsi0)*dt);
      Orientation err --> epsi(t+1)  = psi0 - arctan(f'(x0)) + (v0*delta0*dt/Lf));

The actuators values are the steering angle and the acceleration. Lf is the distance between the center of mass of the car and the front wheels.

For each iteration, the cost of the predicted trajectory is minimized by the actuator values. The cost includes the following:

    -> cte^2 (Penalty on cross-track-error)
    -> epsi^2 (Penalty on total orientation error)
    -> (v-80)^2 (Penalty for speed to be less than 80mph)
    -> Delta^2 (Minimize the steering angle use)
    -> Acceleration^2 (Minimize the use of throttle)
    -> 500*(delta(t+1) - delta(t)) (Penalize sudden changes in steering angle)
    --> a(t+1) - a(t) (Penalty on sudden changes in acceleration)


## Timestep Length and Elapsed Duration (N & dt)

The total time for each horizon is calculated as N times dt. I chose a value of 0.05 to achieve finer actuator control. I tried different values of N to increase or decrease the length of the horizon. I tried N=25 but it resulted it a longer prediction horizon. For a "S" like curve it did a bad job because I have a quadratic fitting curve. Too short of a horizon lead to unstability. I chose a final value of 10 that worked well for me.

## Polynomial Fitting and MPC Preprocessing

Since all coordinates are provided in global coordinate system, I convert all the coordinates to the vehicle system. The waypoints are origin-shifted and 2D rotated to the vehicle frame of reference. I fit the waypoints to a 2nd-degree polynomial to get the fitting coefficients.

The initial vehicle position and heading are always 0 in vehicle frame of reference.

Before the transformation described above, I factor-in the actuator latency of 100ms. It is decribed below.

## Model Predictive Control with Latency

Given a real-world delay in actuations, the model must account for it. I observe oscillations and worsening trajectory if this is not accounted for. In this project we account for 100ms latency. 

Before transforming waypoints, I calculate the expected global position, velocity, heading of the car after the 100ms delay. Also since dt = 0.05 i.e. 2xlatency, I return 2nd actuator values instead of the first ones.

     // To account for latency
     x_new = px + (v*cos(psi)*latency);
     y_new = py + (v*sin(psi)*latency);
     v_new = v + (accel*latency); 
     psi_new = psi + (v*delta*latency/2.67);
     
This approach worked well for me.



## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

