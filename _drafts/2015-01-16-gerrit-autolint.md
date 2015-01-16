---
published: false
---

## Automating Pylint via Jenkins and Gerrit

As many programmers have found, static analysis and sticking to a style guide is incredibly helpful, especially when your team is from diverse code backgrounds. Choosing a static analysis and style guide is the easy part, sticking to it is another story altogether. 

To assist with this, I decided to integrate our Jenkins server with Gerrit to automatically run pylint (our choice for static analysis - not perfect, but it helps) on incoming reviews and post results. There's a handful of guides around the web that will teach you how to setup Jenkins to run a job on gerrit events, but not a lot of information on custom messages back to Gerrit, or automating static analysis. 

### Integrating Jenkins and Gerrit
#### Gerrit Trigger
Install plugins: Gerrit Trigger, GIT Plugin

Configure Gerrit Trigger plugin:
	1. Click 'Manage Jenkins' on the left at the Jenkins Dashboard
    2. Click 'Gerrit Trigger'
    3. Click 'Add New Server' on the left
    4. Type in a name for your server, select 'Gerrit Server with Default Config' and click "Ok"
    5. Fill out the fields with your Gerrit host information. 
    6. You'll need to copy over your Jenkins ssh key into Gerrit (and provide proper access rights in gerrit! 'Stream Events: ALLOW for Event Streaming Users')
    7. Major parts to fill out: 'Hostname', 'Frontend URL', 'Username'
    8. Click 'Test Connection'
    9. If connection fails, try connecting via terminal to your gerrit host (using SSH).
   
Configure your Jenkins Job to use Gerrit Trigger:
	1. Create a new Jenkins Job
    2. Under Source Code Management, select 'Git'
    	a. 
   
   
  



