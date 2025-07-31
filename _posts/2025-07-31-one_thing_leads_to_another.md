## One Thing Leads To Another

Back when I was working at 4Point, one of my side projects was [FluentForms](https://github.com/4PointSolutions/FluentFormsAPI). 
It is a Java client library that allows you to invoke [AEM Forms](https://business.adobe.com/products/experience-manager/forms/aem-forms.html)  functionality remotely. 
It was used in most of our projects where we would implement client (i.e. business) functionality in a Spring Boot application 
that ran alongside AEM Forms (either remotely or, more often, on the same machine as AEM).

One of the challenges I had was integration testing the library with AEM.  I would typically need to do this every three months (each time a new service 
pack was released) in order to ensure compatibility. 
It was a tedious and error-prone process to set up a new install of AEM, deploy the samples manually and then run the tests.

In an ideal world, the integration test process would be automated so that each commit would initiate a 
complete integration test, but how to get there?
The obvious solution was [TestContainers](https://testcontainers.com/).  My CI platform (GitHub Actions) supports Docker, so this seemed feasible, 
however there were a number of hurdles standing in the way:
1. How to create an AEM Forms Docker image?
2. How to make the image available to GitHub Actions since AEM is proprietary and can't be publicly published?

I still haven't got a solution for the last item however, I didn't let that stop me as even having a private docker image would be helpful for
local integration testing.  It may not allow me to include the integration tests in the CI process, but it would allow me to automate the testing 
locally, so I set my mind to the task of creating an AEM Forms Docker image.

The primary issue I encountered was that the creation of Docker images must be automated but the installation procedure for AEM Forms from Adobe is 
purely manual (run AEM, install the latest AEM Service Pack, run AEM again, install the AEM Forms Add-on, run AEM again, etc.). It's a process that 
can take anywhere from a half-hour to several hours, depending on how distracted you get while you wait for service packs to install. There's also
a sequence of manual steps that need to be performed after installing AEM Forms in order to enable all the AEM Forms functionality (edit the 
`sling.properties` file to enable Bouncy Castle, turn protected mode off, etc.).

None of this was rocket science, but no-one had bothered to automate this before.  I needed it to be automated in order to create a dockerfile that would
build an AEM docker image.

#### Enter aem_cntrl

So the steps to install AEM Forms aren't difficult however they involve a lot of waiting around while AEM installs things.  Many installations 
have been ruined by impatience when the person doing the install didn't wait for AEM to finish whatever it was doing. 

One of the many great things about computers is that they have infinite patience. They will literally wait forever for something to happen if that's what
they are told to do.   

I therefore decided to create an installation program for AEM Forms. 
It would start AEM, monitor the logs and shutdown AEM when each installation step was complete. 
I didn't know exactly what would be required at the most detailed level, so I decided to write it in Java as that would give me more 
flexibility than a shell script. It is also one of the languages AEM uses, so it should be familiar to anyone who works on AEM.

I initially wrote a prototype program that did the installation and I got a base install of AEM Forms working as a docker image. 
I then needed to create an "integration test" image from that base image which contained all the sample files needed to perform integration tests. 
This surfaced the need to be able to do some of the things that the installation program did outside of the installation process - things like
start AEM and wait for it to be up before proceeding. 
I realized that, that in addition to installation, I needed to do many of the  individual operations that the installation program already did. 
This meant breaking up the installation into smaller pieces that could be called individually, while still maintaining the overall installation
functionality, so I created AEM Control.

#### The Final Result

The final result was actually several projects:
1. [AEM Control](https://github.com/4PointSolutions/aem-utils/tree/main/aem_cntrl) - this is a command line program that allows for starting/stopping/monitoring of AEM and has an install command for 
performing a full installation. 
1. [AEM Container](https://github.com/4PointSolutions/aem-utils/tree/main/aem_container) - 
this project contains the dockerfiles and instructions for creating a base AEM or AEM Forms Docker container image.
1. [FluentForms Integrtion Test Container](https://github.com/4PointSolutions/FluentFormsAPI/tree/master/rest-services/test_containers) - 
this project contains the dockerfiles and instructions for creating an integration testing AEM Forms Docker container image. 
It can be called from the FluentForms integration test suite (using TestContainers to launch a local AEM Forms container).

I am currently in the process of refining the AEM Control program.  While it needs additional commands and additional documentation, the installation
functionality is in a usable state. I hope you find it useful. Feel free to log [project issues](https://github.com/4PointSolutions/aem-utils/issues) with questions, comments or issues.

---
