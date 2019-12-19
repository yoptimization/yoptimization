

# Structure for Yop
Notes on how to structure the environment around Yop.

# Github
* Organization: Yoptimization
  * hosts the repositorys for Yop, yoptimization.com, example files
* Yop repository with the toolbox
  * Needs a README that explains how to install it, can be the same as on the website.
  * Should have all the releases of yop in the release tab.
  * **TODO:** Look into how to set upp issues so that we can use them easily.
* Yoptimization repository hosting the website.
  * Readme that that tells what the repo is for and how to submit edits and so on
  * **TODO:** look into issues for github and how they can be used for the website repo
* Examples repo???
  * Repo with all the available examples from the website, so that they can be downloaded and such
  * could be annoying if its not updated since then it can diff from the website..



# Website
The structure for the website.

## Topnav The main navigation bar for the website, located at the top
The website will have different environments that is navigated between with the top navigation bar. The environments have their own side bar that makes it easy to navigate one topic at a time, e.g. navigating the docs without having to see examples and tutorials in the sidebar.

From left to right:
* yoptimization.com: links to homepage
* about: about page
* Docs: docs getting started page
* Examples: Example overview page
* Download: install page
* Github: links to github organization or yop repository? **TODO: decide where to link (probably yop repo)!!!**

## Sidebars
The different sidebars that separates the different environments from each other.

The different sidebars that should be available are:
* About sidebar
  * about Yop: also about landing page
  * Citing Yop
  * Navigating Yoptimization.com
  * Navigating GitHub
  * **TODO: Benchmark here? or somewhere else? **

* Docs sidebar
  * the different docs

* Examples sidebar
  * optimal control examples
  * simulation examples
  * tutorials/guides


Unclear things
* benchmark pages
* Use cases tab: No sidebar?
* Download tab: 
* Docs tab: has it own sidebar
  * can we use the generated documentation from matlab?
* Examples/tutorials tab: has its own sidebar
* Github link: maybe to the organization?


---------------------------------------------------------------------------------------------
## Main pages that are available from the topnav.


### Home page (no sidebar)
Page to tell what Yop is and why you should use it.
Inspiration from https://www.tensorflow.org/ and https://pytorch.org/

#### Items on the home page from top to bottom:
* Image of Yop code in matlab editor: Things inside or next to image:
  * Download link
  * Text with name and slogan: e.g. "Yop - optimal control made simple"
* A why yop section
* Needs 2-4 reasons to use Yop instead of the competition.
  * Show examples or link to the reasons Yop is good.
  * Also link to benchmark page.
* Needs 2-4 use cases for Yop.
  * Show examples or link to examples of the use cases.
* Should have 2-4 reasons to use optimal control.


#### 2 Reasons why Yop is good.
* Easy to use:
  * can use your normal matlab ODE (and DAE?) models
  * Easy to fit into your workflow with little or no modification.
  * Efficiently solves easy tasks such as plotting
  * Intuitive interface
* Flexible:
  * Flexible in the sense that you don't have to choose between ease-of-use and functionality.
  * The problem can be expressed in the right domain.
  * abstracted so that the problem can be expressed as a continuous time optimal control problem instead of a nonlinear optimization problem which usually is the case for CasADi.

#### 2 Reasons why you should use optimal control.
* Optimal control as a benchmark
---------------------------------------------------------------------------------------------
### About page
* What is yop in more detail
* People of yop
* How to get involved? (Maybe in the future)
* Citing Yop
* 


---------------------------------------------------------------------------------------------
### Docs landing page
* Landing page explaining the different elements and lists available info
* Should explain how to use the help commands in matlab
* **TODO: ** can you use the generated docs from the matlab help command?



---------------------------------------------------------------------------------------------
### Examples landing page
Page that tells you what examples and tutorials are available. Needs to tell you where to 
start if you're a beginner. Also needs to show different things that can be done with yop and 
how to do them. For example pareto front tutorial so the user knows that it can be done and 
how to do it.

https://www.tensorflow.org/tutorials/ 




---------------------------------------------------------------------------------------------
### Download page

* Installation guide
  * List dependencies
  * List tested configurations?






---------------------------------------------------------------------------------------------

#### 2 Use cases for Yop
* Modelling
  * Yop lets you use your existing ODE models, so that you can just jump in and start optimizing them.
* Optimizing
  * Easy to setup and test different optimal control problems so you get the solution that you're after.




# Tutorials

* Pareto front tutorial
* Parameterize the control signal
  * piecewise constant
  * piecewise polynomial
* simulating the system
  * Adding noise to the optimal solution
  * simulating piece-wise constant, polynomial control signals
* Initial guess
  * Without simulation
  * with simulation
  * creating a controller for the initial guess
* Modelling your system tutorial
  * yopIfElse, yopInterpolant, logistic-function
  * Adding states 
  * Simulating sensors
  * implementing a controller
* Debugging your results
  * Have a suboptimal solution and show the importance of an initial guess
    * e.g. using single pendulum sidestep.
  * Is the solution unique and what does that entail?
    * How can we make it unique?
* Numerical sensitivity analysis of your solution

Different tutorials that should be included:

* Getting started
  * How to download and install
    * Maybe a quick test to see if its working?
  * How to make a optimization problem
  * How to make an optimal control problem
  * How to plot
    * Normal state and control plotting
      * have a Zermelo or Brachistochrone curve problem to show plotting states against eachother? 
    * Show how to plot more complex functions
      * E.g. plot a simulated sensor
      * Can you plot functions such as min(max(u,-1),1)?
      * plot integral(u)
* Adding states 
  * For example adding an extra state to control the derivative instead.
  * To simulate sensors
* Adding a controller
  * optimizing the controller
    * with the optimal trajectory
    * with several/random trajectories over several phases
      * plotting the diff from trajectories for each phase
  * optimizing with the controller
* Initial guess
  * by simulation
  * by simulation with a controller
* Simulating your system with your controller and comparing it to the optimal solution
  * See what performance of the controller is necessary to get close to the optimal solution.
    * Is the controller fast enough, can we use different sampling times on the feedback and so on.



## Bryson Denham basic tutorial
Have a https://idratherbewriting.com/documentation-theme-jekyll/p2_sample5.html style tutorial with the following steps:

* basics
* Different ways of solving Bryson Denham
* Initial guess
* Pareto front





# Ideas for the website
* have a resources tab where you can download different models, inspiration https://pytorch.org/hub/
  * Maybe better when we have a more diverse set of tools to offer?
  
* Have a learning page (maybe under resources) where we link to educational sources about optimal control
  * Inspiration: https://www.tensorflow.org/resources/learn-ml 
  * Use this in combination with minphase.com, so that minphase can have a collection of optimal control resources




<!-- ## Optimization as the reason for improvement.
If you have a system that works it can be hard to motivate an improvement. You have to weight the potential gain versus the cost of implementation, this can be very hard since the magnitude of the potential gain can be unknown. If you then optimize the system you can get a benchmark on the potential gain and with that motive the change. TODO: fix sentences!
When creating a system or updating an old one you have to motivate why your solution i 
When updating an old system you have to weight the potential gain of the update against the cost of implementing it. It can be hard to know what th potential gain could be without getting the time to work on a solution. 
potential gain versus cost of implementation 
Optimal control does not have to be the solution but it can be the benchmark for your solution. Weighting the costs is not only for optimal control but one must do it when deciding on the 
The benchmark can be used to show how close your solution is to the optimal one. Or it can be used to motivate an improvement of a system.
If you have a system and a regulator it is hard to know how good the system is. With the help of  -->

<!-- # Yop is easy to use
Yop has an intuitive syntax, can easily fit into the workflow of an engineer and makes tasks such as plotting easier.
# Intuitive syntax
Express your problems as continuous time optimal control problems without losing functionality.
# Use your existing models
Yop lets you use your existing ODE models, so that you can just jump in and start optimizing them.
# Express and solve your optimization problems
Easy to setup and test different optimal control problems so you get the solution that you're after.  -->


