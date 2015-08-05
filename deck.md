footer:@johnsonch :: Chris Johnson :: Writing For Love and Money :: http://spkr8.com/t/61311
autoscale: true

#Title
##Sub Title

---
#About me

![fit right](http://www.johnsonch.com/images/me.jpg)

* Chris Johnson
* Senior Software Engineer and Scrum Master at GettyImages
* Author at Pragmatic Bookshelf
* Part-Time Instructor at Madison College
* Owner at JohnsonCH, LLC
* @johnsonch => most places on the internet

---
#About GettyImages

> GettyImages is among the world’s leading creators and distributors of award-winning still imagery, video, music and multimedia products, as well as other forms of premium digital content, available through its trusted house of brands, including iStock© and Thinkstock©.

![inline](http://cyberpunklibrarian.com/wp-content/uploads/2014/03/getty_images_logo.jpg)

---
![](https://www.youtube.com/watch?v=4AEsCjalSBc)

---
#Shameless Plug
![full](images/wbdev2_xlargebeta.jpg)

---
![fit right](images/wbdev2_xlargebeta.jpg)

* http://bit.ly/web-development-recipes-2nd-edition
* @webdevrecipes
* http://webdevelopmentrecipes.com

---
#Disclaimer/Rules

* These are **my** experiences, your mileage may vary
* I'm not here to argue, if you want to do that buy me a beer afterwards
* Ask questions

---
#What this talk is about

---
#What this talk is **not** about

---
#A word about 'patterns'

---
#Don't decide and jump to a 'pattern' from the start
  - "Everything is a nail when you have a hammer"

---
#Have a plan, but don't be afraid for it to be wrong

---
#"Patterns"
  * CQRS 
  * Micro Services

---
#CQRS 
##Command Query Responsibility Segregation

> CQRS is simply the creation of two objects where there was previously only one. The separation occurs based upon whether the methods are a command or a query 

---
#Examples 
  * User sign up flow (create user, confirmation email)

---
#Micro Services
  - [Start building the monolith first](http://martinfowler.com/bliki/MonolithFirst.html)

---
#How to I choose what to refactor?

---
# Metrics
  * Rank areas of the application most used and slowest 

---
#Examples 
  * Checkout flow (decrement inventory, shipping, email confirmation, credit card charge) 

---
