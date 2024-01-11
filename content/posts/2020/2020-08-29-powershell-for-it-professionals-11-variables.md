---
title: 'PowerShell for IT Professionals [#11] – Variables'
author: Kamil

date: 2020-08-29T10:39:05+00:00
url: /powershell-for-it-professionals-11-variables/
featuredImage: /wp-content/uploads/2020/08/Variables.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - array
  - collection
  - course
  - foreach
  - IT
  - powershell
  - sysadmin
  - variable

---
{{<youtube CfCySkaJDJ0>}}

In this lesson we&#8217;re going to learn how what Variables are, why it&#8217;s good to use them and how to actually create them. We will then assign various values to variables to and use them solely, and in parameters.

We will also have a look at foreach loop so that we can e.g. ping multiple computers using single variable with multiple objects in it.

## Exercises

  * Use Get-Command to find all commands that allow you to manage variables. Then find the command that let&#8217;s to display all currently available variables.

  * In notepad, store a few hostnames/ip addresses each in separate line
      * Use Get-Content commandlet to read the content of the file just created
      * Store the content of the file into a variable
      * Display all the objects of variable &#8211; it should be exactly the same as content of the file
      * Use foreach loop to ping all the servers from your variable

## Common variable types

  * [string] – Text string 
  * [char] – a single character 
  * [bool] – Boolean (can be either $True or $False) 
  * [int] or [int32] – 32-bit Integer 
  * [long] or [int64] – 64 – bit Integer 
  * [DateTime] – Date and time

## Commands used in the lesson

```powershell
# Let's work on simple string
"Kamil Procyszyn" 

"Kamil Procyszyn" 
| GM

$name = "Kamil Procyszyn"

$name| GM

# Same with numbers

3 

3 | GM

3 + 5

$a = 3

$a | GM

$b = 5

$a + $b

# Let's check what will happen after changing the value of one of properties

$a = 10

$a + $b

# Let's use variables with parameters

$server = "PS-SVR1"

Test-NetConnection $server

Ping $server

Get-Process -ComputerName $server

# Variables give access to methods

$name

$name | GM

$name

$name.ToUpper()

$name.ToLower()

# We can store output of commands to variables and access their properties

$services = Get-Service

$services

$services.name

# We can obviously pipe

$services | where {$_.status -like 's*'}

# Storing multiple objects in variable

$computers = 'ps-svr1','localhost','1.1.1.1','8.8.8.8','kamilpro.com'

$computers

$computers[0]

$computers[2]

$computers[-1]

# Let's try to ping all of them

ping $computers

Test-NetConnection $computers

# If command doesn't allow to provide an array, we can loop through it

foreach ($c in $computers) {ping $c}

foreach ($c in $computers) {Test-NetConnection $c}

foreach ($c in $computers) {nslookup $c}
```
