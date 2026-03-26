## Best practises and Development methodologies

In this document we would like to introduce you our ✨best practices✨ that helps us to make our work more effective and easy. We'll describe six tips that works for us the best.

### Check the task description and design twice (or more)

Task description is one of the most important things in development. Misunderstanding or not clear description can ruin all your work right before deadline. We don't want it to happen.

If something looks unclear, then it is better to clarify this moment in the very beginning. It will help you to avoid deadlock in development due to clarifiation time on customer's side.

Also, it is crutial to get back to description review from time to time throughout all the way of development. Otherwise, you can miss some important requirements or forget to do something.

The next point follows from the previous one.

### Don't be afraid to ask questions

The best way to escape from misalignment during development is to ask all questions that you have as soon as posible. It will help you to better understand customer and make more precise development.

### Create separate threads in Team chat for every ticket

Separate threads for each ticket can structure notes and save ticket information for future tasks. On first glance it may look like too much additional effort but in process of development you'll notice how your work process improves. 

You can easily refresh in mind what you were doing before weekend, review what your colleague has done on last week.

These small notes in thread can help you to develop your own knowlage base that will contain all useful information. So you'll be able to rapidly find how particular feature was done two years ago by other person.

Another topic for thread is TODO list. It is really helpful to mark all things that are done and needs to be done in the scope of your task. So you won't miss nothing.

It is necessary to be consistent in this question, so don't foret to update these threads every day or after every step of development. 

When the task is ready we prefer to send MR to our teammates for check in threads as well. When someone reviews your code he can notice some mistakes that you've missed. Only after that we send our MR to the customer and mark in Discord thread that this task is done ✅.

### MR Check list

But what about MR? How can we avoid fatal errors in main brunch?

Before send MR to review to the customer's side it is important to check all items. Here is an example of MR Check list:

* Check if full local build runs well. Errors and warnings are not allowed;
* Check for code style errors if it is important on your project;
* Remove commented code. If some part of the code in the functions you worked on was commented out a long time ago (you can check this in the git history), it is recommended to delete it;
* Remove temporary debug messages (e.g. OLOLO);
* Don't forget to pull Main brunch into your Feature brunch. If you have a complecated merge conflict then check twice that anything has broken after solving it;
* Flash FW to device and test it;
* Add documentation for the task you are closing: a general overview of the module you worked on, steps to resolve similar issues, etc.;
* Assign someone from your command to review changes before sending MR to customer's side.

If all items are completed successfully, only then send MR to the customer's side.

Don't forget that your own MR Check list can differ from the example above. You can add any other points that you find important to check.

### Review feature workflow after MR

We usually work separately on different features in our team, but find it important to be in the loop of all tasks. So we've created our own practice of reviewing feature workflow all together after the work on this feature is done. A person who worked on it prepares a short summary about the most important moments of development and conducts a small demo to the team.

### Update doca

If we think that the task is unique and has an unusial solution then we prefere to write down the short summary about the most important moments of development from the preveous point. We prefere to keep it in our Documentation repo.

The main thing you should keep in mind while creating a new doca about a task is "Would this information help a person who don't know anything about the workflow of this feature to solve a similar task". Also, it is useful to put some most important pieces of code into the doca.

Sometimes you'll need to get back to the documentation repo and update it. The best way to do it is right after these changes appeared. After few days you can just forget about these changes.

### Team activities (Shortcuts, memes)

For better overall mood of team members we created several side activities that help us brighten up our working days. Here are our favourite ones:

* Weekly hotkey cheat sheet. We choose new for us hotkeys and trying to use them in every suitable situation;
* Meme lotery. We print our favourite memes and put them into a hat. Once a day/week we take a meme out of the hat one by one. Also we have meme wall, we put there our favourite pictures;
* Letter spinning wheel. We send weekly letters to our customer about tasks from last week.We prefer to choose a person who will write this letter by spinning wheel with names of teammates, who wins - writes a letter! 
