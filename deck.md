footer:@johnsonch :: Chris Johnson :: Going Beyond The Rails 'Defaults' :: http://spkr8.com/t/61521
autoscale: true

#Going Beyond The Rails 'Defaults'
##Without yak shaving

![full](images/GettyImages-484589991.jpg)

---

![full](images/SpeakerSlides/SpeakerSlides.001.jpg)

---
![full](images/SpeakerSlides/SpeakerSlides.002.jpg)

---
#About me

![fit right](http://www.johnsonch.com/images/me.jpg)

* Chris Johnson
* Senior Software Engineer and "Scrum Master" at GettyImages
* Part-Time Instructor at Madison College
* Author at Pragmatic Bookshelf
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

![fit left](images/GettyImages-138132680.jpg)

* These are **my** experiences and opinions, your mileage may vary
* I'm not here to argue, if you want to do that buy me a beer afterwards
* Ask questions

---
#What this talk is about

* A high level overview of a couple patters/practices for scaling your rails application
* Some thoughts about code extraction

![full](images/GettyImages-482222116.jpg)

---
#What this talk is **not** about

* A recipe for success
* A "silver bullet" solution
* Plug 'n Play code for your project
* How to deploy

![full](images/GettyImages-483426128.jpg)

^ Just like Coors Light isn't the beer to end all beers, these ideas aren't either.

---
#A word about "patterns"

^ I'm using this word with caution, I don't like the way people use them to segregate code.
We are craftspeople and we need to challenge the status quo by developing new ways to solve
problems

---
#Don't decide and jump to a 'pattern' from the start

---  
#Everything is a nail when you have a hammer

![fit inline](images/GettyImages-482576956.jpg)

---
#Have a plan, but don't be afraid for it to be wrong

![full](images/GettyImages-471638849.jpg)

---
#"Patterns" to discuss
  * CQRS / Command Pattern
  * Micro Services

---
#CQRS 
##Command Query Responsibility Segregation

![full](images/GettyImages-157478186.jpg)

---
> CQRS is simply the creation of two objects where there was previously only one. The separation occurs based upon whether the methods are a command or a query 


---
#Let's focus on Commands

> "Basically it's a way of wrapping up a distinct piece of program activity into its own class..." - shindigital.com

![full](images/GettyImages-108269369.jpg)

---
#Scenario
##Transferring Money

![full](images/GettyImages-465856505.jpg)

---
##This is simplified sample code from a contrived example

![fit inline](images/GettyImages-479438360.jpg)

---
#Controller version

```ruby
def send_funds
  sending_account = Account.find(params[:account_id])
  recieving_account = Account.find(params[:transfer][:target_account_id])
  ammount = params[:transfer][:amount].to_i
  if sending_account && recieving_account
    if sending_account.has_balance_greater_than?(ammount)
      @sent_transfer = sending_account.send_transfer(recieving_account, ammount)
    end
  end
  respond_to do |format|
    if sent_transfer 
      format.html { redirect_to @sent_transfer, notice: 'Your money has been sent' }
      format.json { render :show, status: :ok, location: @sent_transfer  }
    else
      format.html { render :edit }
      format.json { render json: @sent_transfer.errors, status: :unprocessable_entity }
    end
  end
end
```

---
#Yuck

![full](images/GettyImages-482136075.jpg)

---
#Let's shift the logic to a command

![full](images/GettyImages-556936763.jpg)

---
#Commands

```ruby
def send_funds
  @sent_transfer = TransferFundsCommand.new.execute(params[:account_id], params[:transfer])

  respond_to do |format|
    if sent_transfer 
      format.html { redirect_to @sent_transfer, notice: 'Your money has been sent' }
      format.json { render :show, status: :ok, location: @sent_transfer  }
    else
      format.html { render :edit }
      format.json { render json: @sent_transfer.errors, status: :unprocessable_entity }
    end
  end
end
```

---
#Commands

```ruby
class TransferFundsCommand < Command
  def execute(account_uid, attributes)
    sending_account = Account.find(account_uid)
    recieving_account = Account.find(attributes[:target_account_id])
    ammount = attributes[:amount].to_i

    if sending_account && recieving_account
      if sending_account.has_balance_greater_than?(ammount)
        sent_transfer = sending_account.send_transfer(recieving_account, ammount)
      end
    end

    return send_transfer
  end
end
```

---
#Commands

```ruby
class Command

  class Command::NotImplementedError < StandardError; end

  def execute(*args)
    raise NotImplementedError, "#{self.class.name} does not implement an execute method"
  end

end
```

---
#Commands

```ruby
class Account < ActiveRecord::Base

  def send_transfer(recieving_account, ammount)
    ##some call to bank API
  end
end

```

---
#Commands

* This is a simple example, what what about if you were transferring funds with different currencies
* Yes this isn't a full "Gang of Four implementation" 

![full](images/GettyImages-108225179.jpg)

---
#Commands
##Real Time vs Background

^ Real time commands keep your response time the same as it was before.
Background commands can lessen the load on each request, but at the cost of not everything being fully executed

![full](images/GettyImages-78631114.jpg)

---
#Queue/Worker options
* Delayed Job
* Sidekiq
* Active Job

---
#Pitfalls
  * Increased cognitive load
  * Mocking strategies (may not shorten test runs)

^ On my current project because of the size of the team and pace of development
it is not feasible to mock out relationship between commands and ActiveRecord models. YMMV

![full](images/GettyImages-160528242.jpg)

---
#Wins
  * Decreased coupling with ActiveRecord model in controller
  * Increased isolated unit testing

^ Again the coupling is moved to the command from the controller and that also allows you to isolate your unit testing, though you can also run into issues of having business logic in multiple places if you aren't careful

![full](images/GettyImages-187199063.jpg)

---
#Micro Services

![full](images/GettyImages-104525348.jpg)

---
#Micro Services

* "... a software architecture style in which complex applications are composed of small, independent processes ..."
* Each application should be an isolated piece of functionality 


^ [Start building the monolith first](http://martinfowler.com/bliki/MonolithFirst.html)

---
#Examples 
  ![fit inline](images/bank_micro_services.png)

---
#Let's take it a step further and add a queue

^ RabbitMQ is a popular example

![full](images/GettyImages-175280844.jpg)

---
#Examples 
  ![fit inline](images/bank_micro_services_with_queue.png)


---
#Pitfalls
  * Increased cognitive load
  * Integration testing becomes more important

![full](images/GettyImages-521812209.jpg)

---
#Wins
  * Applications are focused in have a single small domain
  * Increased isolated unit testing
  * Applications can be scaled and deployed at different intervals

![full](images/GettyImages-166230058.jpg)

---
#OK, I want to go down one of these paths

![full](images/GettyImages-182656049.jpg)

---
#How to I choose what to extract/re-factor?

^ A full rewrite is a naive way to execute this, start small and extract behaviors

![full](images/GettyImages-200464106-001.jpg)

---
#Metrics

![full](images/GettyImages-143382456.jpg)

---
#Metrics
  * Rank areas of the application most used and slowest 
  * Evaluate each one of these on a feasibility of extracting

---
#Metrics

![fit inline](images/refactor_graph.png)

---
#Then explore other languages and frameworks

![full](images/GettyImages-472042929.jpg)

Often there are other languages or frameworks that may make your portion of the
application be more performant or easier to maintain.

^ Even if you are just extracting to a command is there another utility that can
do the work?  Can you use some 3rd party service?

---
#I don't think either of these approaches are for me

![full](images/GettyImages-483070250.jpg)

---
#That is awesome

![full](images/GettyImages-483556248.jpg)

^ I searched for "awesome" and got a bearded man drinking coffee...

---
#Use your problem to develop a new "pattern" 

![full](images/GettyImages-490637041.jpg)

---
#Again start building and evaluate

^ If your application is making money and taking time to develop always search
for whatever you can do to make it easier to work with both as a developer and 
a user.

![full](images/GettyImages-545878793.jpg)

---

#What does DHH say?

> "There's no one pattern that's going to turn [expletive deleted] code into magic mushrooms..."

![full inline](images/IMG_6059.jpg)

---
#Thank You

* Please rate me on Speaker Rate, and provide feedback in the comments 
* [http://spkr8.com/t/61521](http://spkr8.com/t/61521)
* Materials available at [http://bit.ly/1PizpGG](http://bit.ly/1PizpGG)

![full](images/GettyImages-116846436.jpg)

---
^ All images available at GettyImages.com
