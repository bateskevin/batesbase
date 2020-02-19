---
layout: post
title:  "DockerPester"
date:   2020-02-19 08:56:01 +0100
categories: Powershell
permalink: Powershell/2020/02/19/DockerPester.md/
---

# Docker Pester - Run crossplatform tests locally!

Recently there have been some changes in my Backpack concerning the portable system that I carry with myself to perform my job. 
I have had an HP laptop handed to me on my first day of work at my current company (about two years ago) and I actually had no problems
whatsoever at the start. Now this Laptop lived for about 3/4 years before it decided to stop working. My Laptop would not turn on and I could not work for 1.5 days
cause we had no spare laptops at the time. 

Now that was a bummer.. My company replaced my system with the same model and since I actually liked the Laptop I was handed in the first
place I was excited to have another one of these. Another 3/4 years down the line the exact same thing happens. Again I'm unable to work for 1.5 days before my replacement arrives. Again it is the same model. Then I learned something interessting. My company does BYOD (Bring your own device) and they actually pay for that system. But as, almost, always with cool stuff there is a hook. The laptop my company hands out have a 2 year lease on them before you can have a new one. Very reasonable policy in my eyes and also I thought at the time that my lease would soon expire and I could order a new one.

No...

As I learned a little while after receiving my 3rd laptop (still same modell) I found out that my lease was reset both times I received a replacement and I thought that I would be stuck in an endless HP Laptop loop, where I would get a new one every 3/4 years. 

Now I'm in luck that I have good people above me that allowed me to ditch that HP misery (Not at all implying that's the case for every HP Laptop!) and order myself my own System to be able to work on that. So I started thinking what I would like to use. It would have to perform well since I do developmemt on it and I also am fond of big and nice screens, since I work in a different spot nearly every day and I don't always have the possibility for external screens. Ah and I code Powershell. 

Could I buy myself a Macbook and have a good time working on it mostly writing Powershell code? Powershell 7 is around the edge and crossplatform is available since v6 so that is kind of covered. VSCode is available.. What do I need more? Honestly I tried to inform myself to the best I could, but soon it got clear to me that I was going to get a Macbook.

At the time I write this I have worked on my Macbook for one week now and I have to say I love it! I have noticed no shortcomings up until now and I have done some cool linux stuff on my machine! I have a Linux ansible lab on my Macbook for example. But the main thing I discovered for me this past few weeks was Docker.

I know I'm late to the part, but I started to look at it and play around with it. My main interesst was actually to run Powershell on linux and see how that's working out. But then it hit me. Run Pester tests on docker machines with powershell on them. You can completly destroy your container and there will be a new one up and running in a matter of seconds. 

I know it's a simple idea and I by no means think for a second, that I'm the first one to come up with this, but still. So I started to write a Module.

In this Blogpost you'll learn how to run the Pester tests for your Powershell project on multiple operating systems locally.

[DockerPester](https://github.com/bateskevin/DockerPester) makes this possible.

## What you need to do this

there are a few prerequisites you should install to use 

### Docker ![DockerLogo](https://static1.squarespace.com/static/5bfe78fe9d5abb94164ed3af/5c0f8b3c4fa51a03ef54a96b/5c112c188a922d6c219dcad6/1556553088561/docker-whale.png?format=1500w){:height="50px" width="50px"} 

Docker is a containerization software that let's you run containers on your machine. 

Install it on your platform. It is available for Linux, MacOS and Windows here: [Download Docker](https://hub.docker.com/?overlay=onboarding)

### Docker Context

I'm by no means a specialist when it comes to docker. Actually I only just started with docker myself, but one of
the things I learned about it, is that you can pull docker images only if they fit the architecture of your OS.

So standardwise you will not be able to run a Linux container on your Windows machine of vice versa. On your Mac
you will be able to run Linux containers, but not Windows Containers.

### Switch Context on Windows

On Docker for Windows you can switch your Context once you're running Docker Desktop very easily. 

Click the Arrow up in right bottom corner:

![arrow](https://docs.docker.com/docker-for-windows/images/whale-icon-systray-hidden.png)

And then Switch the Context by Clicking on 'Switch to Winows Containers' or 'Switch to Linux Containers' if Windows is already active:

![arrow](https://docs.docker.com/docker-for-windows/images/docker-menu-settings.png)

This is how you switch between linux and windows containers on Windows.

### Switch Context on Mac

There is a possibility to add a Windows docker Context on MacOS.

I will not describe this in detail in this blog post, since it's already documented very well on Github by Stefan Scherer Here:

[Add Docker Windows Context on MacOS](https://github.com/StefanScherer/windows-docker-machine#working-on-macos)

If you follow this documentation you will have a Windows Context called "2019-box"

# DockerPester

Ok so now you should have installed Docker on your machine, which makes you ready to use DockerPester.

## Installing DockerPester

The Module is currently not available on the Gallery. So at the moment the only option of using it you have is to fork and clone the repo.

## First run of DockerPester

So to start off you just need a Powershell Project of yours with some Pester Tests. Create a Hashtable that 
contains the following Key Value pairs:

{% highlight powershell %}
$ParamSet = @{
  ContainerName = "DockerPester" #Name of the Container used to Test.
  Image = "mcr.microsoft.com/powershell:7.0.0-rc.2-alpine-3.8" #Image used for the Container
  InputFolder = "/Users/kevin/code/PSHarmonize"#Module or Folder you pass with your Tests in them
  PathOnContainer = "/var" #Path you want to copy to in your container
  PathToTests = "Tests" #Path in 'InputFolder' where your Tests are located
  PrerequisiteModule = "PSHTML" #Pass Modules that are prerequisites to your tests to download from gallery in container.
  Executor = "LNX" #The executor is the OS of your Machine - Docker host. If you run on Windows you set "WIN" if you run on MacOS or Linux you set "LNX"
}
{% endhighlight %}

Each Parameter we define in the hashtable is shortly described with a comment on the same line. Make sure you're in the right context for the Image you define here and the Executor matches your OS. Also make sure to change the 'InputFolder' to an actual project with Pestertests in them. 'PathToTests' is the Path in your Powershell Project Folder where your Tests are located.

To run your first DockerPester run use the following code:

{% highlight powershell %}
Invoke-DockerPester @ParamSet
{% endhighlight %}

So here we basically just splat the Parameters we defined in the Hash $ParamSet to 'Invoke-DockerPester'.

So what will that do. First it will spin up a Container with the Image we defined in our Hash. The ContainerName we set will be the name of that container. Then it will copy the Powershell Module Project Folder from your local Computer that you defined under 'InputFolder' to the path you defined in 'PathOnContainer' on the Container. After that it will start downloading the Prerequisite modules that you defined to download from the gallery and it will install the newest version of Pester and yes, the output on that is not sooooooo nice I will admit that, but hey, we all need room to grow right? 

After the installation of the modules it will start executing the Pester Tests. And when that's done it will return the Passthru Object from Pester back to you.

The Output you will see should look like this: 

{% highlight powershell %}
PS /Users/kevin> Invoke-DockerPester @ParamSet                                 
f6b8047075adc5ee491b772b9321e85bf0e628d9add2a1ff9f79ea254894227a
DockerPester
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 Installing package 'PSHTML'                                                        Downloaded 0.00 MB out of 0.64 MB.                                              [                                                                    ]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       Installing package 'Pester'                                                        Downloaded 0.00 MB out of 0.85 MB.                                              [                                                                    ]                                                                                                                                                                       Installing package 'Pester'                                                        Process Package Manifest                                                        [oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo    ]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      Pester v4.10.1
Executing all tests in '/var/PSHarmonize/Tests'

Executing script /var/PSHarmonize/Tests/Functions.Letters.Tests.ps1

  Describing [PSHarmonize][Functions] Testing Letter Functions

    Context [PSHarmonize][Functions][Letter] C Without Parameters.
      [+] The Function for C should not throw without Parameters. 165ms
      [+] C should Contain the Correct Property for 'EnharmonicFlavour' without any Parameters 18ms
      [+] C should Contain the Correct Property for 'IsPause' without any Parameters 3ms
      [+] C should Contain the Correct Property for 'Length' without any Parameters 39ms
      [+] C should Contain the Correct Property for 'Letter' without any Parameters 17ms
      [+] C should Contain the Correct Property for 'NoteMapping' without any Parameters 17ms
      [+] C should Contain the Correct Property for 'Numeral' without any Parameters 12ms
      [+] C should Contain the Correct Property for 'Octave' without any Parameters 11ms
      [+] C should Contain the Correct Property for 'Velocity' without any Parameters 10ms

    Context [PSHarmonize][Functions][Letter] C Wit Parameters.
      [+] [PSHarmonize][Functions][Letter] Octave should be assignable 14ms
      [+] [PSHarmonize][Functions][Letter] Octave should be assignable 11ms

    Context [PSHarmonize][Functions][Letter] Chords of C
{% endhighlight %}

If you want to save the PassThru Object of your Tests to a variable use the following Code:

{% highlight powershell %}
$PassThru = Invoke-DockerPester @ParamSet
{% endhighlight %}

In there you will find all the filled Properties of your standart PassThru Object from Pester. 

# In Conclusion

I had an HP laptop breaking all the time that pushed me to buy a Macbook for my Powershell job. That in exchange pushed me to try out some stuff on Linux which broadens my skillset and I found out I actually really like it! This brought me to docker and I had a lot of fun writing that module, which is far from finished :)

Docker is awesome, Powershell is awesome!

You can run Pester Tests in a docker container and retrieve the Passthru Object with [DockerPester](https://github.com/bateskevin/DockerPester).

Cheers!

Kevin