# The automation process stages

This will be a high level experience I&#8217;ve been gaining whenever automating manual processes.

## What does it mean to automate?

Simply put, is to remove a person from equation in order to complete the task. It can go as far as automating the whole process, or just part of it. The main goal here is to release the resource &#8211; that is a person&#8217;s time and energy &#8211; and redirect it toward something more valuable. They are also side benefits of automating like consistency, audit ability and cost.

## Let&#8217;s pick some example

Whenever there&#8217;s something to be done, there must be a certain process to be followed. Set of steps which will lead to an accomplishment of the task. The process might be either verbal &#8211; that is people pass the knowledge to each other &#8211; or written &#8211; that is there&#8217;s a document/diagram listing step by step what must be done.

Let&#8217;s take the backup checks (and if you don&#8217;t do backups at the moment, please <a aria-label="read this (opens in a new tab)" rel="noreferrer noopener" href="https://kamilpro.com/do-you-even-back-up/" target="_blank">read this</a>) as an example. Imagine that you have a bunch of clients for whom you perform daily backups, and check the result every day, creating a service desk ticket per each failed backup. The way the task is performed is fairly straightforward &#8211; you open a predefined spreadsheet which lists all the computers being backed up, login to the application which is performing backups, and look at the reports, looking for failed and /or missed backups. While identifying these, you then create a ticket for each of the findings and record the ticket number with the state of the backup against the computer. Depending on the fleet of computers and the actual success of the backups, you might end up with various amounts of possible tickets to be created. 

When everything else has failed, backups become the last resort &#8211; they are invaluable. The problem with backups is that their value is often overseen, as they are taken for granted &#8211; like a water in the tap. Customers will suddenly ask for the backup if the data is gone.

Personally, I couldn&#8217;t imagine being caught &#8220;red handed&#8221; when failed to report a broken backup, when customer asks for a data to be restored. 

I can identify a few issues between the critical nature and repetitiveness of the work:

  * The same task is performed daily, over and over
  * The person performing checks might not really see a reward or value since backups don&#8217;t provide value until recovery is needed. This state can go for months, if not years.
  * It&#8217;s possible to overlook an issue
  * There&#8217;s nobody shouting about broken backups, thus priority might be given to other incoming work.

## Step one: gathering information

In order to automate anything, there must be an understanding what actually must happen, what&#8217;s the desired outcome. 

This can be compared to the plant work &#8211; scarce materials on the one side, finished product on the other. Whatever gets us from one the other, is the process. The better the process, the sooner, consistent and better the end product is. 

To really understand how it&#8217;s currently done, there&#8217;s no better way like to live it &#8211; either follow the person performing the task or get hands dirty.

The key here is to experience every step, understand why it must be done. Ask questions, even if something seems obvious. If there are any exceptions &#8211; ask. Understanding the process can save a lot time later, and might allow to improve it as well.

## Step two: create initial procedure or algorithm

Having now written steps, it is time to combine them all together. 

This will be a fundamental for the automated task, thus the form can be either a list of steps, a procedure or an algorithm. 

Therefore there should be things like:

  * Step 1: Go to website
  * Step 2: Login to the website with that account
  * Step 3: If there are failed backup do this, if missed do that 

As you can see on one hand we are creating an easily human readable procedure, on the other we can use it as an algorithm for writing a code.

## Step three: How can I login to this? How to test it?

While it&#8217;s fairly obvious to user to login somewhere, it might not be for the script to do so. 

Does the app you&#8217;re trying to automate support API? If there&#8217;s an API, can it actually be elevated to use what you&#8217;re trying to achieve? What if there&#8217;s no API, can you create an account, or need to wait for someone? What sort of permissions are needed?

While these are seem to be obvious things, we often forget about them &#8211; ask them in advance. There&#8217;s nothing worse than being stalled just because you can&#8217;t hack to the mainframe.

## Step four: Automating!

Yes, this is the fun part. Now all the information gathered can be put into something that computer understands. 

The common problem is that, what seems fairly straightforward in manual way, might be not that obvious in how actually it was programmed under the hood.

The list created in step 2 will let you to keep on track, and know what script should be doing next.

Another obstacle might be the actual execution of the code &#8211; is it going to be fully automated? If so, what will execute your code? A server, VM or maybe some serverless application like Azure Function? If the code is going to be executed by a person, then that question might not be relevant.

But let&#8217;s assume the code is executing and now we are getting to the point what we people love, and machines not really &#8211; that is&#8230;

## Step five: Exceptions

Yes, something what we as human beings can just get by, the automation won&#8217;t &#8211; unless it&#8217;s hard coded in there. 

Coming back to the backups checks example &#8211; let&#8217;s assume there are some old servers that haven&#8217;t been backed up for over a year, yet they still exist as the job in the backup software.

Do they need to be there? Do these computers still existing operate? Do we actually need data?

The process of automation is a perfect opportunity to standardise processes, housekeeping and clarifying what was an open loop for way too long.

On one hand, we get a clear picture, on the other, the person performing checks has one less thing to remember while going through checks and/or reports. It&#8217;s simply a win/win situation.

## Step six: Testing

Testing should be really done on every step of development, but here I&#8217;d like to stress something different.

When we have a ready product, and are ready to go live, it&#8217;s worth to carry on doing manual work and comparing it against the result of the automation. The goal is here to get same results as per manual task. E.g. if there were five missed backups and seven failed reported manually, then the automation should return exactly same results.

There&#8217;s really no point into automating anything if we can&#8217;t trust its work, and the best way to enhance trust is to prove it side by side.

It might be worth setting a goal upfront that if e.g. if for two weeks all automated tests will cover results of manual tests, then we will stop doing manual checks. If the testing would fail at any moment, it must be pushed backed, reviewed, fixed and the testing cycle starts again.

## Step seven: Go live

At this point, when the automation has been proved and has been working as expected, there&#8217;s really not much more to do than put in production. We can communicate and confirm the system is live and no manual checks are longer needed.

It might be worth to e.g. initially for the first month perform manual check once a week, thereafter less frequently.

## Let&#8217;s summarise

I&#8217;ve been using the above seven steps and found it to be useful. On one hand we gain insights and understanding of how the process of question is working &#8211; it alone can lead to improvement. From this point, it&#8217;s very easy to document the process. And automating something that is well documented is a pure joy. 

I hope you&#8217;ll find it useful, and looking forward for your thoughts.

Photo by [Us Wah][1] on [Unsplash][2]

 [1]: https://unsplash.com/@uswah?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [2]: https://unsplash.com/search/photos/press?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText

