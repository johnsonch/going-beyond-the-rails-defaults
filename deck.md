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

![fit left](images/GettyImages-138132680.jpg)

* These are **my** experiences and opinions, your mileage may vary
* I'm not here to argue, if you want to do that buy me a beer afterwards
* Ask questions

---
#What this talk is about

* A high level overview of a couple patters/practices for scaling your rails application
* Some thoughts about code extraction

---
#What this talk is **not** about

* A recipe for success
* A "silver bullet" solution
* Plug 'n Play code for your project

^ Just like Coors Light isn't the beer to end all beers, these ideas aren't either.

---
#A word about 'patterns'

^ I'm using this word with caution, I don't like the way people use them to segregate code.
We are craftspeople and we need to challenge the status quo by developing new ways to solve
problems

---
#Don't decide and jump to a 'pattern' from the start
  
##Everything is a nail when you have a hammer

---
#Have a plan, but don't be afraid for it to be wrong

---
#"Patterns" to discuss
  * CQRS / Command Pattern
  * Micro Services

---
#CQRS 
##Command Query Responsibility Segregation

> CQRS is simply the creation of two objects where there was previously only one. The separation occurs based upon whether the methods are a command or a query 

---
#Let's focus on Commands

> "Basically it's a way of wrapping up a distinct piece of program activity into its own class..." - shindigital.com

---
#Scenario
##Transferring Money


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

---
#Commands
##Real Time vs Background

^ Real time commands keep your response time the same as it was before.
Background commands can lessen the load on each request, but at the cost of not everything being fully executed

---
#Pitfalls
  * Increased cognitive load
  * Mocking strategies (may not shorten test runs)

^ On my current project because of the size of the team and pace of development
it is not feasible to mock out relationship between commands and ActiveRecord models. YMMV


---
#Wins
  * Decreased coupling with ActiveRecord model
  * Increased isolated unit testing

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

---
#Examples 
  ![fit inline](images/bank_micro_services_with_queue.png)

---
#Pitfalls
  * Increased cognitive load
  * Integration testing becomes more important

---
#Wins
  * Applications are focused in have a single small domain 
  * Increased isolated unit testing
  * Applications can be scaled and deployed at different intervals

---
#OK, I want to go down one of these paths

---
#How to I choose what to extract/re-factor?

---
#Metrics
  * Rank areas of the application most used and slowest 
  * Evaluate each one of these on a feasibility of extracting

---
#Then explore other languages and frameworks

Often there are other languages or frameworks that may make your portion of the
application be more performant or easier to maintain.

^ Even if you are just extracting to a command is there another utility that can
do the work?  Can you use some 3rd party service?

---
#I don't think either of these approaches are for me

---
#That is awesome

---
#Use your problem to develop a new "pattern" 

---
#What does DHH say?

> "There's no one pattern that's going to turn shitty code into magic mushrooms..."

---
#Again start building and evaluate

^ If your application is making money and taking time to develop always search
for whatever you can do to make it easier to work with both as a developer and 
a user.

---
