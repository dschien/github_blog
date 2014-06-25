Title: STS Agent Based Model Building
date: 2014-06-16 13:00
comments: true
slug: sts-agents

<!-- PELICAN_BEGIN_SUMMARY -->
Step by step instructions to follow during the Agent Based Modelling day in the socio-technical systems unit 
at the University of Bristol. 
<!-- PELICAN_END_SUMMARY -->
 
## Table of Contents
[TOC]

# Preparation 

We'are using repast simphony as modelling environment.
There are two options to install this on your computer:
- A) download the distribution as a binary (Windows/OSX) or install as a plugin to Eclipse.
- B) download a VirtualBox virtual machine pre-configured

Option B) requires you to download a 4GB archive but it guarantees, that you can run all code examples.
Option A) requires that you download the code examples separately 

## Repast Simphony 

## Virtual Machine 
You must have Oracle VirtualBox installed on your computer.
- [VM Download link](https://www.dropbox.com/sh/a69w6cm104o1dpz/AACAs_tCvXgrEyce5-724SiLa)
- [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- You also need the virtual box extensions from the VirtualBox download page.

### Starting the VM
Once you downloaded the VM it should be a matter of double clicking the `ABM.vbox` file to bring focus
to the VirtualBox settings window.

You'll need to amend the location of the hard disk image file in the settings dialog.
Choose the SATA controller "Add Harddisk" > Choose existing and choose the `abm_vm.vdi` file.

![vm storage settings](|filename|/images/201406/virtualbox_settings.png)

You can find additional instructions in this [video](/4t6gk7zve0b5hqa/ABM.mp4)
{% video https://dl.dropboxusercontent.com/s/4t6gk7zve0b5hqa/ABM.mp4 %}

## Direct Install 
 
- Download for Win/Mac/Linux from [here](http://repast.sourceforge.net/download.php)
- For Linux you need Eclipse from [here](http://www.eclipse.org/downloads/)

Plus you need to install **all** of the following plugins:

- Repast from this update site (http://mirror.anl.gov/repastsimphony)

From the default Kepler update site (is preconfigured):

- GMF Tooling 3.1.0
- GMF Tooling Runtime Extensions 3.1.0

From the Groovy Update Site: http://dist.springsource.org/release/GRECLIPSE/e4.3/

- Extra Groovy Compilers
- Groovy-Eclipse

### Installing Plugins
Go to *Help*> *Install New Software*
![eclipse update](|filename|/images/201406/eclipse_update.png)


- click "Add"
- Paste in the url to your plugin as above and a name after your liking (I use the url as a name)
- Wait until the plugin index has finished loading
- Pick your plugins
- Accept licenses if necessary
- Repeat as necessary
 
## Sources from Github
- You can use the command line or the GUI for Windows/ MacOS
- Checkout by commit tag with command line or GUI

- the repository is available [https://github.com/dschien/NormalJobs](https://github.com/dschien/NormalJobs)
- The command to clone is: `git clone https://github.com/dschien/NormalJobs.git`

### Changing to a specific commit
- In the GUI there are buttons to do it
- On the command line do
1. `git log` to find a tag
2. `git checkout <tag>`

# Building the Model

The goal is to develop an alternative modeling exploration to reproduce the results from 
"Early estimation of project performance: The application of a system dynamics rework model"

In particular we're interested in reproduce the project performance measurements.
![work done](|filename|/images/201406/project-performance.png)
  
## Relogo Basic Concepts

- ReLogo is based on Eclipse IDE
- creates Repast Simphony models with **turtles, links, patches, and observers**. 
- Using the Logo domain specific language based on the Groovy (Groovy: A dynamic language for the Java platform) dynamic language. 
- ReLogo freely interoperates with Groovy and Java and run as a regular Repast Simphony model

### Terms 

Agent Terminology | Relogo Equivalent
------------------|------------------
Agent             |Turtle
Environment       |Observer          


## Development Cycle
1. Code your Agents, Environment and Observer
2. Run the simulation inside the Relogo GUI 
3. Log output to files or interface to other tools for analysis

## Relogo GUI
- visualisation of simulation space
- display of simulation results in charts and tables
- define connectors to external analysis applications
- set simulation parameters for simulation run
![work done](|filename|/images/201406/relogo_gui.png)
  

### Sequence Diagram

<!-- contents of sd for https://www.websequencediagrams.com/
title Job Progress

note right of Worker: create new instances
Observer->Worker: initialisation
Observer->Job: initialisation
Observer->OpenJobPool: initialisation
Observer->DoneJobPool: initialisation

loop each time step
note right of Worker: executed in each time step
alt worker has no Job
    Observer->OpenJobPool: get next job
    Observer->Worker: assignJob
else worker has finished Job
    Observer->Worker: removeJob
    Observer->DoneJobPool: move Job to done Jobs
else worker needs more time
    Worker->Job: increment done level
end       

end
-->
![sequence diagram](|filename|/images/201406/sd.png)


## Phase 1 - Project Foundations 
1. We'll create a new project of type 'Relogo'
2. Add turtles representing Agents
3. Add a UserObserver representing the environment
4. Add a `setup` method to initialise the simulation and a `step` method to define the actions taking place in each time step  

### Creating a new project
0. Select the Relogo perspective
1. Right click on workspace
2. Select New > ReLogo Project
3. Title "NormalJobs"

### Creating a new worker turtle
1. Right click src/normaljobs.relogo
2. Select New> Turtle
3. Title "Worker"
4. Define new method 
```groovy    
def step(){}
```
    
### Worker Turtle Complete
```Groovy
package normaljobs.relogo

import static repast.simphony.relogo.Utility.*;
import static repast.simphony.relogo.UtilityG.*;
import repast.simphony.relogo.Plural;
import repast.simphony.relogo.Stop;
import repast.simphony.relogo.Utility;
import repast.simphony.relogo.UtilityG;
import repast.simphony.relogo.schedule.Go;
import repast.simphony.relogo.schedule.Setup;
import normaljobs.ReLogoTurtle;

class Worker extends ReLogoTurtle {
	def step(){

	}
}
```

### Preparing UserObserver
1. Open src/normaljobs.relogo/UserObserver
2. Remove comments around `step` and `go` methods
3. Rename `createTurtles` to `createWorkers`
4. Replace constant number with variable `numWorkers`
5. Rename target of go method to `workers`
6. Delete body of `ask` closure

### UserObserver Complete
```Groovy
package normaljobs.relogo

import static repast.simphony.relogo.Utility.*;
import static repast.simphony.relogo.UtilityG.*;
import repast.simphony.relogo.Stop;
import repast.simphony.relogo.Utility;
import repast.simphony.relogo.UtilityG;
import repast.simphony.relogo.schedule.Go;
import repast.simphony.relogo.schedule.Setup;
import normaljobs.ReLogoObserver;

class UserObserver extends ReLogoObserver{

	@Setup
	def setup(){
		clearAll()
		createWorkers(numWorkers){
		}
	}

	@Go
	def go(){
		ask(workers()){
		}
	}
}
```

### Create `numWorkers` Variable in `UserGlobalsAndPanelFactory`
1. Add new slider to change the number of workers
    `addSliderWL ("numWorkers", "Number of Workers", 1 , 1 , 1000 , 11)`
2. Delete others

### Complete UserGlobalsAndPanelFactory
```Groovy
package normaljobs.relogo

import repast.simphony.relogo.factories.AbstractReLogoGlobalsAndPanelFactory

public class UserGlobalsAndPanelFactory extends AbstractReLogoGlobalsAndPanelFactory{
	public void addGlobalsAndPanelComponents(){
		
		addSliderWL ("numWorkers", "Number of Workers", 1 , 1 , 1000 , 11)
	}
}
```

### Create Job Class
We'll need to track the completion level for each Job. We encapsulate this in a new object type.
1. Create a new class 
2. Name 'Job'

### Start Test Run
1. Click Play > NormalJobs Model
2. In the Relogo GUI Click the power-on button to initialise
3. Click play
4. Click Stop

### Github commit
You can get the github sources via commit [c9fca9d84b652a524ca1b2c478df288eb7b8a8bf](https://github.com/dschien/NormalJobs/commit/c9fca9d84b652a524ca1b2c478df288eb7b8a8bf) 


## Phase 2 - Working on Jobs
- Creating new jobs in job pools
- Workers adds to completion
- Observer maintains job pools

### Add properties to class `Job`

1. Add a property `double numRequirements`  
2. Add a property `double completionLevel = 0` to track the level of completeness of this job


### Make Worker work on Jobs
1. In turtle `Worker` create class property `Job job` and a property `double increment = 1`

2. in `step` method, if has job and the complemetionlevel is smaller than requirements then increment completion level by 1
3. add method `isJobDone` return `true` if job complete, `false` otherwise

### Worker Turtle complete
```Groovy
package normaljobs.relogo

import static repast.simphony.relogo.Utility.*;
import static repast.simphony.relogo.UtilityG.*;
import repast.simphony.relogo.Plural;
import repast.simphony.relogo.Stop;
import repast.simphony.relogo.Utility;
import repast.simphony.relogo.UtilityG;
import repast.simphony.relogo.schedule.Go;
import repast.simphony.relogo.schedule.Setup;
import normaljobs.ReLogoTurtle;

/**
 * Worker Agent
 * @author schien
 *
 */
class Worker extends ReLogoTurtle {
	
	Job job = null
	double increment = 1
	
	/**
	 * Increment completion level by increment if completion level less than requirements
	 * @return
	 */
	def step(){
		if (this.job && this.job.completionLevel < this.job.requirements){
			this.job.completionLevel += increment
		}
	}
	
	/**
	 * Check if job done.
	 * @return true if done, false otherwise
	 */
	def isJobDone(){
		if (this.job && this.job.completionLevel >= this.job.requirements){
			return true
		}
		return  false
	}
}
```

### Control number of Jobs
1. Create a new variable `numJobs` in `UserGlobalsAndPanelFactory` to control the number of Jobs

### Make Observer manage Job Pool
1. Create two new members `openJobs` and `doneJobs` both of type Queue
2. In the setup method create jobs with a requirement level of 100
- you can use a for loop as you know from java or a range expression such as`(0..<numJobs).each{}`
- adding to a queue can be done with the `add` method from the java [Collections API](http://docs.oracle.com/javase/7/docs/api/java/util/Collection.html)
- or you can use the Groovy `<<` operator as in `this.openJobs << new Job(requirements:100)`
3.. In the go method, 
- iterate over each worker, implement as a closure to the ask method
 ```Groovy
 ask(workers()){ worker ->
    // do something with the worker
 }
 ```
- check if a worker has a job, 
- if it doesn't -> assign a job,
- let him work on it
- if it does have a job -> check if the job is complete
- if it is complete -> take the job away, add it to the `doneJob` pool


### UserObserver Complete
```Groovy
package normaljobs.relogo

import static repast.simphony.relogo.Utility.*
import static repast.simphony.relogo.UtilityG.*
import normaljobs.ReLogoObserver
import repast.simphony.engine.environment.RunEnvironment
import repast.simphony.relogo.Stop
import repast.simphony.relogo.schedule.Go
import repast.simphony.relogo.schedule.Setup

class UserObserver extends ReLogoObserver{

	def openJobs = [] as Queue
	def doneJobs = [] as Queue

	@Setup
	def setup(){
		clearAll()
		this.openJobs = [] as Queue
		this.doneJobs = [] as Queue

		(0..<numJobs).each{
			this.openJobs << new Job(requirements:100)
		}

		createWorkers(numWorkers){
		}
	}

	@Go
	def go(){
		ask(workers()){ worker ->
			if (!worker.job) {
				if (this.openJobs){
					worker.job = this.openJobs.poll() 
				} 
			} 
			// let the worker work on it
			worker.step()
			
			if (worker.isJobDone()){
				this.doneJobs << worker.job
				worker.job = null
			}
		}		
	}	
}

```

### Github commit
You can get the github sources via commit [051e412de17cf1bf6fe94199f4380ae0bb51c3e8](https://github.com/dschien/NormalJobs/commit/051e412de17cf1bf6fe94199f4380ae0bb51c3e8)

### Run the project
What do you see?
What do would like to see?

## Phase 3 - Adding Monitoring
We'll add monitors, so that we can track how our workers are completing jobs
We'll also add logging of output to file to plot in excel
And we'll plot output in a chart

### Add a method for open and done job size
Repast Relogo allows us to create data sinks that track the change of variables over time.
These require to access the number of open and done jobs in our queues. Relogo requires that 
they are accessible via methods.
1. Create new methods `getNumDoneJobs` and `getNumOpenJobs` and return the size of the queues
```Groovy
def getNumDoneJobs(){
		return this.doneJobs.size()
	}
```

### Add new Data Sinks for open and closed Jobs
1. Start the Relogo GUI
2. Right-click the Data Sinks Node, title 'Jobs'
3. Select an aggregate Source, keep tick count in 'standard' set tab
4. In the tab "Method data sources" select agent type "UserObserver" and method 'getNumOpenJobs', set 'Open Jobs' as the name
5. Repeat for done jobs
6. Click the Disk symbol to save these settings
It should look as in the image below
![linear Jobs](|filename|/images/201406/datasink.png)


### Add a File Sink
1. On Node 'Text Sink' add a new file sink
2. Select all variables from the data source jobs, including ticks
3. Set a file name
![linear Jobs](|filename|/images/201406/filesink.png)

### Add new Chart
1. Right-click the Charts Node, title 'Jobs', type Time Series
2. Select the 'Jobs' Data source you just created
3. Pick both series
4. Set a title, and y-axis lable
5. Save settings


### Run
- What do you see? -> the graph is pretty much unreadable
- Repeat with the maximum number of jobs
- In eclipse, select the project folder and click 'Refresh'
- A file jobs.{date} should appear, open it with Excel, if you please

![linear Jobs](|filename|/images/201406/chart-linear.png)


### Automatically stop the simulation on Jobs complete
1. Add a new local variable `idleWorkers` to the UserObserver
2. In the branch for empty queue and idle workers, increment the `idleWorkers` variabel
3. At the end of the `go` method, test if the `numWorkers == idleWorkers`, in that case run `RunEnvironment.getInstance().endRun();`
4. Select "Source" > Organize Imports to declare the class RunEnvironment 

### UserObserver complete
```Groovy
package normaljobs.relogo

import static repast.simphony.relogo.Utility.*
import static repast.simphony.relogo.UtilityG.*
import normaljobs.ReLogoObserver
import repast.simphony.engine.environment.RunEnvironment
import repast.simphony.relogo.Stop
import repast.simphony.relogo.schedule.Go
import repast.simphony.relogo.schedule.Setup

class UserObserver extends ReLogoObserver{

	def openJobs = [] as Queue
	def doneJobs = [] as Queue

	@Setup
	def setup(){
		clearAll()
		this.openJobs = [] as Queue
		this.doneJobs = [] as Queue

		(0..<numJobs).each{
			this.openJobs << new Job(requirements:100)
		}

		createWorkers(numWorkers){
		}
	}

	@Go
	def go(){
		int idleWorkers = 0

		ask(workers()){ worker ->
			if (!worker.job) {
				if (this.openJobs){
					worker.job = this.openJobs.poll()
				} else {
					// all jobs done
					idleWorkers ++

				}
			}
			// let the worker work on it
			worker.step()

			if (worker.isJobDone()){
				this.doneJobs << worker.job
				worker.job = null
			}
		}
		// we've completed our work
		if (idleWorkers == numWorkers){
			RunEnvironment.getInstance().endRun();
		}
	}

	def getNumOpenJobs(){
		return this.openJobs.size()
	}

	def getNumDoneJobs(){
		return this.doneJobs.size()
	}

}

```

### Run 
We should now see something like this
![linear Jobs](|filename|/images/201406/linear-jobs.png)

### Github commit
You can get the github sources via commit [66cef351683115a3790e070d49b8e042bf88cd83](https://github.com/dschien/NormalJobs/commit/66cef351683115a3790e070d49b8e042bf88cd83)


## Phase 4 - Normalising
Our model is completely linear. We'd like to simulate an s-shaped curve for the work done.
Because we'd like to be as quick as possible, we plan to achieve this by distributing work in our open work pool as a normal distribution 
horizontally reflected.
![sigma](|filename|/images/201406/sigma.png)

We can do calculate requirements relative to some min requirement level we pick.
![increment](|filename|/images/201406/increment.png)

Idea: We'll assign requirements to a job in proportion to the probability of the normal distribution.



### Refactoring 
Our jobs will have varying number of requirements. It makes sense to control the later, as the number of jobs is a function of the
number of requirements.
- In `UserGlobalsAndPanelFactory` change the variable `numJobs` number of jobs to `numRequirements and change the display lable 
- also increase the value boundary to 10000

### Initial Jobs by requirements, normaly distributed

0. Implement the norm function as
```Groovy
def norm(x, mu, sigma){
		return 1/(sigma * Math.sqrt(2 * Math.PI)) * Math.exp( - (x - mu)**2 / (2 * sigma**2))
	}
```


1. Given our total num of reqs, we define mean as `def mu = T/2`
2. And sigma as `def sigma = T/4` -> thus 95.4% of all samples will be in this interval. The remaining 4.6% will go in the tail
 ![work done](|filename|/images/201406/sigma.png)
- We want each worker to have roughly the same amount of requirements to deal with 
- Change the increment level for our worker on initialisation
1. Create a new property in Observer `def workIncrement = 10`
2. In the worker creation closure add 
```Groovy
it ->
			it.increment = this.workIncrement
```

- Now we'll work from the mean to the outside, create jobs and add those to head and tail of our queue
![work done](|filename|/images/201406/increment.png)

## Optional: Using libraries
- Usually we don't want to implement each method we need, so we use existing libs
- For example, you could replace the norm function with the Apache stats lib
- download the [binary](http://commons.apache.org/proper/commons-math/download_math.cgi)
- extract, copy the `commons-math3-3.3.jar` to the `lib` folder in your eclipse project
- go to the java perspective, browse the `lib` folder, right click the jar file > Build Path > 'Add  to Build Path'
- Return to the Relogo perspective
- Create a new instance of NormalDistribution `def norm = new NormalDistribution()`, then get $p(x)$ from `norm.density(x)`