---
title: Powershell – resolve full names to usernames from CSV file
author: Kamil

date: 2017-10-21T14:03:58+00:00
url: /powershell-resolve-full-names-usernames-csv-file/
categories:
  - IT
  - Powershell
  - Windows
tags:
  - AD
  - cvs
  - fullname
  - powershell
  - script
  - username

---
It&#8217;s unbelievable how long it took me to figure out this simple Powershell script, but yet it does the trick :).

Often you receive a request to do a certain action with a bunch of accounts, and (obviously) the list provided contains the full names rather usernames. Pain to do it manually, but yet we can utilise Powershell here.

What you need: CSV file, with only one column, and the header of the column must be called &#8220;Name&#8221;, if you prefer to use something different, simply amend Name in line 6 to reflect your header.

So what the actual script does:

Line 3 = creates variable called **$list** and tells it to import a CSV file in **C:\temp\names.csv** path.

Line 5 = creates a loop which will go through all records in from the CSV file

Line 6 = We retrieve usernames, filtered by their Full Name (**Name** in AD object property) and tell it should search under **Name** header.

This is it, a few lines of code which will probably save you plenty of time in search of usernames :).

<pre class="lang:ps decode:true" title="Resolve full names to usernames">#Finds username based on full name. Header in CSV file is Name.

$list = Import-Csv C:\temp\names.csv

foreach ($i in $list) {
    Get-ADUser -Filter "Name -like '$($i.Name)'"
}
</pre>
