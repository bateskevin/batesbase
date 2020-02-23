---
layout: post
title:  "Set up DockerPester for your Project"
date:   2020-02-19 08:56:01 +0100
categories: Powershell
permalink: Powershell/2020/02/19/Set-up-DockerPester-for-your-Project.md.md/
---

# Set up DockerPester for your Project

Check out the Second part of this Post: [How to set up DockerPester for your Project](https://bateskevin.github.io/batesbase/Powershell/2020/02/19/Set-up-DockerPester-for-your-Project.md.md/)

In the [first Blogpost on DockerPester](https://bateskevin.github.io/batesbase/Powershell/2020/02/19/DockerPester.md/) I talked about how you can run Pester tests in containers with Powershell. Short version of that Post:

1. Install Docker
2. Install DockerPester
3. Run Invoke-DockerPester with a few params.

that was basically it. You can run your Tests in a Container and get your Passthru Object back and look at you results. Now while this is fine, and this is the baisc functionality of this module, it does not really invite you to actually use it in your day to day coding life. 

So at the moment of writing this post there have been 3 Versions of a Prerelease on the gallery. 

#### Prerelease 0.1.0-alpha

In 0.1.0 I published the base functionality described in the first blog post linked above. 

#### Prerelease 0.2.0-apha

In the second Prerelease I added a nice output. In the first Version it was really just about publishing the functionality, that anybody may test and feedback on it. In this Version I wanted to have a nice console output and more verbosity on what is currently happening, so troubleshooting while the code is failing is gonna be easier. So that is what was published here.

#### Prerelease 0.3.0-alpha

Now the content of this Prerelease is what this post is going to be about. Here I added a way (just a few functions) to set up DockerPester for your Project in a persistent way. You set this up once and you can use the Same cmdlet "Invoke-DockerPester", but instead of using all the Parameters (which did not change) you can use the new Parameter "Project", pass your Project name and your tests will be run. 

To be perfectly honest, parts and bits of this functioality where already in the previous prerelease, but it was far from finished.

# Set up a Project

So to set up a Project you will need to define some parameters. I usually define them in a hastable and then splat it to my command:

So first define your Hash:

{% highlight powershell %}
$Param = @{
    InputFolder = "/Users/kevin/code/PSHarmonize”
    Image = "mcr.microsoft.com/powershell:7.0.0-rc.2-alpine-3.10”
    Context = "default”
    Executor = “LNX”
}
{% endhighlight %}

And then Splat it to the 'Add-DockerPesterProject' commandlet:

{% highlight powershell %}
Add-DockerPesterProject @Param
{% endhighlight %}

Now what are we passing here. The first Parameter 'InputFolder' is the path to out project locally. 'Image' is the Image that Docker will use to spin up the container. we define the Context, in case we are in a different Context while starting the run and set the Executor that is Coresponding with the OS of the Image of the Container Machine. 

This would look like this when performed in Powershell:

![AddDockerPesterProject](https://github.com/bateskevin/DockerPester/raw/master/IMG/AddDockerPesterProject.gif)

As you see to retrieve Information about your Project you can use:

{% highlight powershell %}
Get-DockerPesterProject -Name PSHarmonize
{% endhighlight %}

#### Multiple Jobs

You can do the process described above multiple times to add more jobs to your project. Use this for example to create multiple jobs with different Images. So let's say we would want to test our code with alpine 3.10, but also with alpine.3.9. You just add it like this:

{% highlight powershell %}
$Param = @{
    InputFolder = "/Users/kevin/code/PSHarmonize”
    Image = "mcr.microsoft.com/powershell:7.0.0-rc.2-alpine-3.9”
    Context = "default”
    Executor = “LNX”
}
{% endhighlight %}

And when you run 'Get-DockerPesterProject -Name PSHarmonize' again you get both jobs:

{% highlight powershell %}
Image              : mcr.microsoft.com/powershell:7.0.0-rc.2-alpine-3.10
Context            : default
PrerequisiteModule : 
PathToTests        : Tests
ContainerName      : DockerPester
Executor           : LNX
InputFolder        : /Users/kevin/code/PSHarmonize
PathOnContainer    : /var

Image              : mcr.microsoft.com/powershell:7.0.0-rc.2-alpine-3.9
Context            : default
PrerequisiteModule : 
PathToTests        : Tests
ContainerName      : DockerPester
Executor           : LNX
InputFolder        : /Users/kevin/code/PSHarmonize
PathOnContainer    : /var

{% endhighlight %}

# Run DockerPester for your Project.

So as said at the start of this post once you've set up our project like described above you can run the following command to run all your defined jobs for your project:

{% highlight powershell %}
Invoke-DockerPester -Project PSHarmonize
{% endhighlight %}

This will run all of your defined jobs:

![Run](https://media.giphy.com/media/igIq9155M6eSmG0UWC/giphy.gif)

Ok that's cool! Now the last thing left to do is to retrieve the Results of your Pester tests. To get back an Object with the Information of all of your last jobs use:

{% highlight powershell %}
Get-LatestDockerPesterResults -Project PSHarmonize
{% endhighlight %}

This will give you back an Array with Information about your Jobs, including the PassThru Object of each Job:

{% highlight powershell %}
Name                           Value
----                           -----
PassThru                       @{PassedCount=479; TotalCount=479; InconclusiveCount=0; ScriptBlockFilter=; TagFilter=; ExcludeTagF…
DateOfTest                     20200223102733
Image                          mcr.microsoft.compowershell7.0.0-rc.2-alpine-3.9
PassThru                       @{PassedCount=479; TotalCount=479; InconclusiveCount=; ScriptBlockFilter=; TagFilter=; ExcludeTagFilter=;…
DateOfTest                     20200223102637
Image                          mcr.microsoft.compowershell7.0.0-rc.2-alpine-3.10
{% endhighlight %}


And there you have it folks. You can run this over and over again and test your code on different Images. A cool Idea would be to display your results somehow. Maybe html or something in that direction would be cool. But the gist here is that you do this very easy set up once, and you can run as many tests with one very easy commands as you want.

If you have any feedback on the project (which still is in a starting phase) please open Issues on [Github](https://github.com/bateskevin/DockerPester) or reach out to me on Twitter or Slack.

Cheers,
Kevin