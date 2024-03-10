---
title: "Powershell And GitHub Actions"
author: Kamil
date: 2024-03-10
resources:
  - name: "featured-image"
    src: "featured-image.jpeg"
image: "featured-image.jpeg"
categories: ["PowerShell", "GitHub", "Product"]
tags: [ "PowerShell", "pwsh", "GitHub", "Actions"]
---

I am happy to announce my latest product - PowerShell and GitHub Actions - in which you will learn how to develop, build, test and release a PowerShell module using GitHub Actions.

The series is divided into three projects, which build upon each other to create the module.

In the first project - Build a Module - you will start by developing a module locally. The module will implement GitHub's API so that it will be able to manage GitHub's repository lifecycle - creation, removal, updating and retrieving the repository. Once the module builds locally, you will develop a GitHub Workflow - or CI/CD pipeline - so that the module will be automatically after each push to the repository.

In the second project - Test and Analyze Code - you will write automation tests for the module. Tests will include unit tests and integration tests using Pester, as well as static code analysis using PSScriptAnaylzer. Once you have developed locally, you will add automated testing to the GitHub Workflow.

In the third, and final project - Package the Module - you will focus on releasing the module. In the first task, you will use the NuGet tool to package up the module - so that it can be installed using standard PowerShell commands like Install-Module. The second part of the project focuses on making sure the module works cross-platform on Linux and Windows, gets versioned with Git Tags and finally released on GitHub.

After completing all three projects, you will learn how to develop a module, test it and release it with the CI/CD pipeline running on GitHub.

The series is meant for intermediate PowerShell developers, who have been writing PowerShell scripts and functions and would like to take their skills to the next level.

The series has been designed to be completable on Linux, Windows and macOS. 

I'm excited to share that this project has been developed in collaboration with Manning Publications and has been published as liveProject series.

If you like to learn more about the series follow the link: http://mng.bz/v89q