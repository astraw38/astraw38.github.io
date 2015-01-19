---
published: false
---

## Automating Pylint via Jenkins and Gerrit

As many programmers have found, static analysis and sticking to a style guide is incredibly helpful, especially when your team is from diverse code backgrounds. Choosing a static analysis and style guide is the easy part, sticking to it is another story altogether. 

To assist with this, I decided to integrate our Jenkins server with Gerrit to automatically run pylint (our choice for static analysis - not perfect, but it helps) on incoming reviews and post results. There's a handful of guides around the web that will teach you how to setup Jenkins to run a job on gerrit events, but not a lot of information on custom messages back to Gerrit, or automating static analysis. 

### Integrating Jenkins and Gerrit
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
    	a. Set 'repository URL' to your Gerrit host, including your project (<protocol>://<username>@<hostname>:<port>/project/path
        b. Select your credentials that you use to connect to gerrit (we use SSH)
        c. Click 'Advanced'
        d. Set 'Refspec' to '$GERRIT_REFSPEC'
    	e. Set 'Branch Specifier' to '$GERRIT_BRANCH'
        f. Click 'Add', then select 'choosing strategy', and change it to 'Gerrit Trigger'
![](http://i.imgur.com/jmexAle.png)        
    3. Select 'Gerrit Event' as a Build Trigger
    4. Under 'Gerrit Trigger' Section:
    	a. Select your server you created earlier under 'Choose a Server'
        b. Select 'Silent Mode'
        c. Choose you want to trigger on (We use Patchset Created, excluding trivial rebases and 'no code changes'). 
![](http://i.imgur.com/tR08BFM.png)

        d. Beneath 'Dynamic Trigger Configuration' --- DO NOT SELECT THE CHECKBOX:
        Choose 'Path' On the left dropdown, in the box put '**'. On the right, select 'Path' again,   and '**' in the text box. 
 ![](http://i.imgur.com/AkzmSCV.png)
 
    5. In the Build section, use 'Execute Shell':
    	a. 'python gpylinter.py'
        
## [Gerritlinter](https://github.com/astraw38/gerritlinter)
Gerritlinter is a rather simple python script I wrote to do the following:

1. Get a list of files changed between the active gerrit branch and the specified gerrit review.
2. Pylint the original files in the active gerrit branch.
3. Pylint the changed files. 
4. Analyze the results according to specified validators.
5. Post the results of the validation to gerrit via SSH (+1/-1 score assigned, including a message). 

GerritLinter uses the environmental variables set by Gerrit Trigger to do almost all of the configuration. You can still use it via command-line (with options!) for manual testing. 

I'll be continuing to work on it, hopefully even adding the option to include other Validators (for different coding languages) - or at the least creating a structure for that to be easily added.