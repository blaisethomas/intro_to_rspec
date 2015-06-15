#RSPEC in RAILS
--



> Understanding testing is one of the final fundamental components of understanding your first framework... #RAILS

>  Frameworks make testing **easy** and **accurate** 

> Good tests === Great code

### ROADMAP
1. Intro (Blaise)
	* recap on testing 
	* unit specs (controller, model) VERSUS integration / acceptance tests (feature, background, scenario : tests that match our user stories)
	* setting up RSPEC in RAILS
	* "rspec.describe", "it", writing pending examples, and forming our first suite
	* testing convention with convention : scaffold tests
	* running RSPEC in the Terminal (rspec + filepath)
	* expect matchers
	* some model specs if we have time
2. Model Specs (John) 
	* testing convention with convention (scaffold)
	* testing validation
	* testing methods
3. Controller Specs (Blaise) 
	* testing convention with convention (scaffold)
	* testing our controller CRUD actions
	* look at view testing if we have time (then we will have tested all MVC)
4. Lab (John + Blaise)
	* start forming the unit spec suites for project 3 
5. Integration // Acceptance testing with Capybara (the REAL deal - a 2 part lesson tomorrow with Ira) 
6. TDD your API (in later weeks)
<br><br><br>

--
# Intro
--

##1. [recap on testing](https://github.com/ga-students/WDI_LA_16/tree/master/07-week/intro_to_tdd)

-> **test** and **spec** are often used interchangeably. 

-> What do we know about testing so far?

->In class review lab. 10 minutes max.


##2. unit specs (model, view, controller) VERSUS integration / acceptance tests (feature, background, scenario : tests that match our user stories)

>Ok, so now lets do the same thing in our Rails app.



Unit tests are for testing models, views and controllers . Basically, they are the simplest of your tests, exercising very specific functionality. Each example should test one specific thing. 

Integration tests, on the other hand, test the entire Rails stack. Each request in an integration test mimics a real web request and exercises routing recognition, actually parses incoming requests, uses real sessions, and so forth. As a result, integration tests are significantly slower than spec tests, but they are excellent at testing cross-controller stories. Want to make sure the flash you set in the “create” action is being properly displayed in the “index” action? Sounds like you need an integration test. **You can even use integration tests to exercise entire stories:** “user logs in, views the catalog, views a product, adds it to their cart, checks out, enters credit card, submits payment, sees invoice.” ~ [source](http://weblog.jamisbuck.org/2007/1/30/unit-vs-functional-vs-integration.html)

###Below is an example of some simple **unit** specs for models and controllers. 

```ruby
#demo model valditaion spec
  it "is invalid without a last name" do
    user = User.new(first_name: "Roger", last_name: nil, email: "roger@example.com")
    expect(user).to be_invalid 
  end
  
  it "returns a user's full name as a string" do
    user = User.new(first_name: "Roger", last_name: "Smith", email: "roger@example.com")
    expect(user.full_name).to eq "Roger Smith"
  end
  
##demo controller spec
  it "renders the index template" do
    expect(response).to render_template("index") 
  end
  it "response should be a success" do
    # expect(response).to be_success
    expect(response).to have_http_status(200)
  end
  it "assigns @items to include items" do
    expect(assigns(:items)).to include(@item1, @item2)
  end
  
```

###Below is an example of acceptance tests for User login. 


> If you are having trouble getting your Rails app to "do stuff" for you, **properly**, then try writing some tests like this...


```
feature "Signing in" do
  background do
    User.make(:email => 'user@example.com', :password => 'caplin')
  end

  scenario "Signing in with correct credentials" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', :with => 'user@example.com'
      fill_in 'Password', :with => 'caplin'
    end
    click_button 'Sign in'
    expect(page).to have_content 'Success'
  end

  given(:other_user) { User.make(:email => 'other@example.com', :password => 'rous') }

  scenario "Signing in as another user" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', :with => other_user.email
      fill_in 'Password', :with => other_user.password
    end
    click_button 'Sign in'
    expect(page).to have_content 'Invalid email or password'
  end
end
```

##3. setting up RSPEC in RAILS
(assuming you do a rails new with -T)

1. Add the RSpec Rails gem to our Gemfile:
    
    ```ruby
    group :development, :test do
      gem 'rspec-rails', '~> 3.2.1'
    end
    ```

2. Run `$ bundle install` to add this gem.

3. Install RSpec in our project:  

  ```
  $ rails g rspec:install
  ```

  This command generates **four things** in our Rails application:
  1. The **.rspec** file: This is where the RSpec run options go. There are three defaults in this file.
		* The first is `--color`, which gives us colored output in our terminal (red for failing examples, green for passing examples, and yellow for pending examples).
		* The second is `--warnings` (this gives us warnings about our test suite (eg deprecation warnings about `should` matchers etc...) 
		* The third default is `--require spec_helper`, which ensures that our `spec_helper` file is always required.
		* **let's swith up our configuration so that it looks like this:**


		```
		--color
		--warnings
		--require spec_helper

		to 

		--color
		--format=documentation
		--require spec_helper
		```
	
		* `--format=documentation` gives us a clearer output in terminal when we run RSPEC. There are other options, eg: `--format Fuubar`
	
	2. The **spec** directory, which is where all our specs will go. We will create subdirectories like `model` and `controller` when we need them.
	3. The **spec_helper.rb** file, which contains general RSpec settings.
	4. The **rails_helper.rb** file, which requires `spec_helper`, loads the Rails environment, and contains settings that depend on Rails.

##4. "rspec.describe", "it", writing pending examples, and forming our first suite

1.  Make a new directory inside the `spec` folder called `models`.

2.  Make a new file inside `spec/models/` called `user_spec.rb`.

3. Let's start by setting up our first spec:

	  ```ruby
	  require "rails_helper"

	  RSpec.describe User do
	    it "is invalid without a first name" do
	    end 
	  end
	  ```

  Before we run this, let's break down each of these parts:

    * The `"require rails_helper"` on line one refers to the file that contains all the Rails-related configuration that is common to all our tests. We are going to leave everything in here at it's defaults for now.
    * On line 3, we are using the `RSpec.describe` method. In RSpec, the `describe` method is used to define a suite of tests that share a common setup. The `describe` method takes one argument (usually a class name or a string), followed by a block. The argument describes what the test suite will cover, and the block contains the test suite itself.
    * Our actual test (a.k.a. "example", a.k.a. "spec") is defined with the `it` method. The `it` method takes a string argument that is used to describe what this particular spec is testing. The other thing the `it` method takes is a block, which is where the body of the spec is declared.



##5. testing convention with convention : scaffold tests

###AN IMPORTANT NOTE! 

With **unit specs**, we are going to be testing **basic** (unit) properties of our models, views and controllers. If this code is sticking to Rails convention (such as the code that we can scaffold), then we can also use conventional tests. Notice that if you install Rspec **before** running a `rails g scaffold` the generator will run even more code this time as it will scaffold default tests that apply to the standard CRUD code that we have generated. This is an interesting exercise but we need to learn to do this manually. 

At first, unit specs might seem a little trivial, and hard to understand, but the real goal is to learn enough about unit specs so that we can get on to learning about Integration tests. This will happen tomorrow and everything will make a lot more sense.




##6. running RSPEC in the Terminal (rspec + filepath)

Now let's run our test and see what happens. There are a few ways we can run our test:

  ```
  $ rspec
  ```

  This command will run all of our spec files for our whole application. Depending on the version of RSpec you have installed globally, you might have to prepend `rspec` with `bundle exec`, like so:

  ```
  $ bundle exec rspec
  ```

  Sometimes we don't want to run all of our specs for our entire application. This can be slow if we have lots of tests and that overhead is unneccessary if we are currently working on one spec file. We have the option to only run spec files that are in a specific directory, or, alternately, we can run a single spec file by specifying it after the `rspec` command:

  ```
  $ rspec spec/models/
  ```

  or

  ```
  $ rspec spec/models/user_spec.rb
  ```


##7. [expect matchers](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers)

rspec-expectations ships with a number of built-in **matchers**. Each matcher can be used
with `expect(..).to` or `expect(..).not_to` to define positive and negative **expectations** respectively on an object. 

Most matchers can also be accessed using the `(...).should` and
`(...).should_not` syntax; By default, **both** expect and should syntaxes are available. In the future, the default may be changed to **only enable the expect syntax**. [AVOID USING THE "SHOULD" SYNTAX](https://github.com/rspec/rspec-expectations/blob/master/Should.md)

Here is an example of some expect matchers. These will live within our examples (declared using "it")

```
#following this pattern
expect(operator or method) assert

-- 

expect(result).to   eq(3)
expect(list).not_to be_empty

expect(actual).to be(expected) # passes if actual.equal?(expected)

expect(actual).to eq(expected) # passes if actual == expected
```

##8. some model specs if we have time