---
published: false
---

## Automating Pylint via Jenkins and Gerrit

Static analysis and sticking to a style guide helps everyone stay on the same page, especially when your team comes from diverse code backgrounds. Choosing a static analysis and style guide is the easy part, of course--sticking to it is another story altogether.

To get everyone onboard, I integrated our Jenkins server with Gerrit to automatically run pylint (our choice for static analysis - not perfect, but it helps) on incoming reviews and post results. There're a handful of guides around the web that teach how to setup Jenkins to run a job on Gerrit events, but not a lot of information on custom messages back to Gerrit, or automating static analysis.

### Integrating Jenkins and Gerrit
Install plugins: Gerrit Trigger, GIT Plugin

#### Configure Gerrit Trigger plugin:

1. Click 'Manage Jenkins' on the left at the Jenkins Dashboard
2. Click 'Gerrit Trigger'
3. Click 'Add New Server' on the left
4. Type in a name for your server, select 'Gerrit Server with Default Config' and click "Ok"
5. Fill out the fields with your Gerrit host information. 
6. You'll need to copy over your Jenkins ssh key into Gerrit (and provide proper access rights in gerrit! 'Stream Events: ALLOW for Event Streaming Users')
7. Major parts to fill out: 'Hostname', 'Frontend URL', 'Username'
8. Click 'Test Connection'
9. If connection fails, try connecting via terminal to your gerrit host (using SSH).
![](http://i.imgur.com/QUM1zz0.png)   
   
  
#### Configure your Jenkins Job to use Gerrit Trigger:
1. Create a new Jenkins Job
2. Under Source Code Management, select 'Git'
3. Set 'repository URL' to your Gerrit host, including your project
	(_protocol_://_username_@_hostname_:_port_/_project_/_path_)
       ex: ssh://tester@gerrit_host:29418/Auto/automation_test
4. Select your credentials that you use to connect to gerrit (we use SSH)
5. Click 'Advanced'
6. Set 'Refspec' to '$GERRIT_REFSPEC'
7. Set 'Branch Specifier' to '$GERRIT_BRANCH'
8. Click 'Add', then select 'choosing strategy', and change it to 'Gerrit Trigger'
![](http://i.imgur.com/jmexAle.png)        
9. Select 'Gerrit Event' as a Build Trigger

#### Under 'Gerrit Trigger' Section:
    Still in the same jenkins job configuration
1. Select your server you created earlier under 'Choose a Server'
2. Select 'Silent Mode'
3. Choose you want to trigger on (We use Patchset Created, excluding trivial rebases and 'no code changes'). 
![](http://i.imgur.com/tR08BFM.png)
4. Beneath 'Dynamic Trigger Configuration' --- DO NOT SELECT THE CHECKBOX:
        Choose 'Path' On the left dropdown, in the box put '**'. On the right, select 'Path' again,   and '**' in the text box. 
 ![](http://i.imgur.com/AkzmSCV.png)
 
#### In the Build section:
    Use 'execute shell': 'python gpylinter.py'
        
## [Glint](https://github.com/astraw38/Glint)
Glint is a python package to assist with automatic code reviews. It does the following:

1. Get a list of files changed between the active gerrit branch and the specified gerrit review.
2. Lint the original files in the active gerrit branch.
3. Checkout the current review ID
4. Lint the changed files. 
5. Analyze the results according to specified validators.
6. Post the results of the validation to gerrit via SSH (+1/-1 score assigned, including a message). 

Glint uses the environmental variables set by Gerrit Trigger to do almost all of the configuration. You can still use it via command-line (with options!) for manual testing. 

Glint provides the ability to plugin your own Linter or Validator classes. All you need to do is run LintFactory.register_linter(NewLinter()) or ValidatorFactory.register_validator(NewValidator()). When you run 'run_linters()' or 'run_validators()', it'll pick them up and use them. Your new Linters should derive from BaseLinter, and your new Validators should derive from BaseValidator.

You can also add a checkers to validators, which are simple functions to compare lint data that are passed to the validator.