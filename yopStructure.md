

# Structure for Yop
Notes on how to structure the environment around Yop.

## Github
* Organization: Yoptimization
  * hosts the repositorys for Yop, yoptimization.com, example files
* Yop repository with the toolbox
  * Needs a README that explains how to install it, can be the same as on the website.
  * Needs to update all versions to the releases tab.
  * **TODO:** Look into how to set upp issues so that we can use them easily.
* Yoptimization repository hosting the website.
  * Readme that that tells what the repo is for and how to submit edits and so on
  * **TODO:** look into issues for github a bit and how they can be used for the website repo


## Website
* Home page: Selling page with no sidebar


* About tab: about sidebar
  * about Yop: also about landing page
  * Citing Yop
  * Navigating Yoptimization.com
  * Navigating GitHub
  * **TODO: Benchmark here? or somewhere else? **


* benchmark pages
* Use cases tab: No sidebar?
* Download tab: 
* Docs tab: has it own sidebar
  * can we use the generated documentation from matlab?
* Examples/tutorials tab: has its own sidebar
* Github link: maybe to the organization?

### Sidebars
* About
* Docs
* Examples


### Topnav
From left to right:
* yoptimization.com: links to homepage
* about: about page
* Docs: docs getting started page
* Examples: Example overview page
* Download: install page
* Github: links to github organization or yop repository? **TODO: decide where to link (probably yop repo)!!!**

### Home page: Selling page with no sidebar
Needs 3-4 reasons to use Yop instead of the competition. Benchmark page is good but the reasons needs to be listen on the front page.
Also needs 3-4 use cases for optimization.
Can end up as 4 combinations of it?

* Easy to use:
  * can use your normal matlab ODE (and DAE?) models
  * Easy to fit into your workflow with little or no modification.
  * Efficiently solves easy tasks such as plotting
  * Intuitive interface
* Flexible:
  * Flexible in the sense that you don't have to choose between ease-of-use and functionality.
  * The problem can be expressed in the right domain.
  * abstracted so that the problem can be expressed as a continuous time optimal control problem instead of a nonlinear optimization problem which usually is the case for CasADi.



#### Items on the home page from top to bottom:
* Image of Yop code in matlab editor: Things inside or next to image:
  * Download link
  * Text with name and slogan: e.g. "Yop - makes optimization easy"
* Use cases: list at least four examples of use cases that has links that jump you down to an explanation of them.
* Use case 1:
* Use case 2:
* Use case 3:
* Use case 4: 

* Use case examples for Yop:
  * 1: Modelling and testing your system (goes into testing)
  * 2: Benchmarking your system which can be used as a motivation to upgrade your controller, or something in your system.
  * 3: optimizing, ofc
  * 4: controlling
  * 5: ??


