# Flight in a nutshell

In this tutorial I will be teaching you the basics of Twitter's Flight.js by making a donation widget which also uses Materialize for the front-end and Stripe to handle the payments. We'll be covering Flight's main concepts and methods.

Flight is an event driven framework by Twitter. Based on components Flight maps behaviors to DOM nodes, independently. Unlike other popular frameworks Flight doesn't prescribe a particular approach to how you render or fetch your data, it is however dependent on jQuery. In its essence Flight is all about events, these can be triggered by the DOM or by artificial triggers within other UI components. Flight is basically a framework for making frameworks. To some (most) of you this might sound complicated. Flight isn't as 123 as jQuery but it's learning curve is exaggerated and learning Flight could definitely improve your JavaScript skills. 

### Why use Flight?
- It's only 5kb in size.
- Write better jQuery.
- Write re-useable components.
- Use as much or as little of other libraries as you want.
- A better structure for your code in general.

I've made a template for us to use, it includes everything you need to get started so you can get straight to the coding. You can download it from Github or fork it on Codepen. Want to make your own template? Just copy in these CDN's:
```html
      <script src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
      <script src="http://flightjs.github.io/release/latest/flight.min.js"></script>
      <script src="https://checkout.stripe.com/checkout.js"></script>	
```
## Making your first component

Flight consists of 'components,' components are reusable blocks of code that standalone within your application. Creating a component is a lot like creating an object constructor except for the fact that components cannot be changed or accessed after they have been initialized. Components can only communicate with your app through events. Either by triggering them or listening to them.

If you are using the template(if not copy the markup), the form in our Index.html file should look like this:
```html
      <form action="#">
      <input type="checkbox" id="accept" />
      <label for="accept">By proceeding your agree to our terms, which are completely unfair and well, not very real.</label>
      <br/>
      <button class="waves-effect btn" style="margin-top: 15px;" id="launch" disabled>Let's Go</button>
      </form>
```
We want to enable the button when the check-box has been checked and disable it when that isn't the case. Let's head over to the main.js file and create our first component. 
```javascript
    var checkToEnable = flight.component(function () {
	// Magic
    });
```
Each Flight component consists of a few different methods. Some of these are required, others are not. The first method we'll be adding is Attribute. The Attribute method contains one or more  attributes. Attributes are scoped variables and / or arguments. Empty ones, attributes that are attributed null,  require a value to be passed to them when the component is initialized. Other attributes will use their default value unless instructed otherwise. Insert the following code into your component. Attributes are typically used to hold element references. 
```javascript
    this.attributes({
        button: null 
    });
```
The button attribute will serve as a reference to the button we want to enable. The next method we want to add is the Initialize method. We will pass in the reference to the button as an argument.
```javascript
    this.after('initialize', function () {
       this.on('change', this.enableButton); 
    });
```
The initialize method is like the final callback, for a component, it's the only part of the component which directly interacts with the rest of your app. Inside of the callback function we've added an event listener. This will listen for a change on the element which the component shall be attached to and consequently trigger the native enableButton function, which we'll create now.
```javascript
    this.enableButton = function (e) {
        switch(e.target.checked) {
            case true:
            document.getElementById(this.attr.button).disabled = false;
            break;
            case false: 
            document.getElementById(this.attr.button).disabled = true;
            break;
        }
    };
```
This doesn't do anything to fancy. It's just a simple function which enables the button when the checkbox, which this component will be attached to, is checked and vice versa. In case you didn't know yet, JavaScript has native Switch statements, and they are awesome. Also you may have noticed that we accessed the attribute by calling this.attr. This is bad practice and I will show you a better solution later on.

Our component isn't working just yet. To complete our component we need to attach is to the DOM. This happens 'outside' of the component. A component can be attached to as many elements as you like, it can also be attached to the document, but the component has to come first.
```javascript
    checkToEnable.attachTo('#accept', {
    button: 'launch'
    });
```
Great! You've created your first Flight component. In doubt, it should be looking like this: 
```javascript
    var checkToEnable = flight.component(function () {
   		this.attributes({
    		button: null 
    	});
    	this.enableButton = function (e) {
   	 		switch(e.target.checked) {
    			case true:
    			document.getElementById(this.attr.button).disabled = false;
    			break;
    			case false: 
    			document.getElementById(this.attr.button).disabled = true;
    			break;
   			 }
    	};
    	this.after('initialize', function () {
       		this.on('change', this.enableButton); 
    	});
    });
```
## Diving In

By now you should understand the three most important methods. Attributes, callback functions and Initialize. Most Flight component's consist of just this. Our next one is going to use the core concept of Flight, event bubbling. Event bubbling sounds a little complicated but it's actually not that hard to understand. For exampled let's say I have a button and it's parent is a div. When the button is clicked it's event will bubble up to the div, assuming our component is attached to the div.

This is exactly how our next component will work. It will be attached to the donation widget, in the form of a Materialize modal, but we'll be listening to events on its children. First things first, we need to add the markup for the modal into our Index.html file.  Insert it before the closing body tag.
```html
    <div id="stripe-widget" class="modal">
     <div class="modal-content">
      <h4>Give us your money.</h4>
      <p>We'll use it well, we promise.</p>
      <form action="#">
         <p class="range-field">
         <input type="range" id="stripe-amount" value="10" min="0" max="100" />
        </p>
      </form>
     </div>
     <div class="modal-footer">
       <button class="btn blue waves-effect waves-blue" id="checkout" disabled>Donate <span data-amount=""></span> <i class="fa fa-cc-stripe"></i></button>
      <a href="#!" class=" modal-action modal-close waves-effect waves-red btn-flat">Close</a>
     </div>
    </div>
```
Now let's create our component.
```javascript
    var getAmount = flight.component(function () {
    // Magic
    });
```
To better understand our component we'll be adding its methods in reverse order. First of all add the initialize method.
```javascript
	    this.after('initialize', function () {
	        this.on(this.attr.range,'change', this.onChange); 
	        this.on(this.attr.checkout, 'click', this.onClick);
	    });
```
Looks different doesn't it? Because our component is attached to the donation widget we're passing in two of it's children within our event listeners. This might not make sense now but it soon will. Let's add the attributes.
```javascript
	   this.attributes({
	       checkout: '#checkout',
	       range: '#stripe-amount',
	       display_amount: '[data-amount]'
	   }); 
```
These attributes work with the current markup, you might add this component to some different markup in the future, in which case you can pass in different selectors. Next up we'll be adding the onChange function.
```javascript
	   this.onChange = function (event) {
	      var amount = this.select('range').val();
	      if (amount == 0) {
	         alert('please enter an amount');
	         this.select('checkout').prop('disabled', true);
	      } else {
	          this.select('checkout').prop('disabled', false);
	          this.select('display_amount').text('$' + amount);
	          this.select('checkout').attr('data-stripe-amount', amount);
	      }
	   };
```
The one method that stands out is Select. Select is a lot like jQuery's find method, but it's scope only includes children of the attached element (the donation widget). The hardest thing to understand is that we're adding our selectors, from the attributes, within quotes. This confused me to at first so just keep this in mind, because this is actually one of the shortcuts Flight has created.

Since the button has been enabled it can now listen for events, lets add the onClick function now.
```javascript
	   this.onClick = function (event) {
	       var stripeAmount = this.select('checkout').attr('data-stripe-amount');
	       stripeAmount = stripeAmount + 0 + 0;
	       this.trigger('callStripe', {
	           amount: stripeAmount
	       });
	   };
```
First of all we're fetching the amount from an attribute on the button, then we're making it valid for Stripe by adding two zeros. That's nothing new though. The real magic is happening in the trigger method, which is triggering an artificial event and passing along the amount as data, which is also know as the payload. We're going to create the component listening for that event next. The finished component should look like this:
```javascript
	var getAmount = flight.component(function () {
	   this.attributes({
	       checkout: '#checkout',
	       range: '#stripe-amount',
	       display_amount: '[data-amount]'
	   }); 
	   this.onChange = function (event) {
	      var amount = this.select('range').val();
	      if (amount == 0) {
	         alert('please enter an amount');
	         this.select('checkout').prop('disabled', true);
	      } else {
	          this.select('checkout').prop('disabled', false);
	          this.select('display_amount').text('$' + amount);
	          this.select('checkout').attr('data-stripe-amount', amount);
	      }
	   };
	   this.onClick = function (event) {
	       var stripeAmount = this.select('checkout').attr('data-stripe-amount');
	       stripeAmount = stripeAmount + 0 + 0;
	       this.trigger('callStripe', {
	           amount: stripeAmount
	       });
	   };
	   this.after('initialize', function () {
	       this.on(this.attr.range,'change', this.onChange); 
	       this.on(this.attr.checkout, 'click', this.onClick);
	       });
	});
```
Before we create the final component we have to initialize the previous one. Because the modal itself is initialized dynamically we'll attach it after it has been initialized. The code below is just a simple click event listener for the button we enabled in the first component, ater which we attach our new component. The openModal method belongs to Materialize.
```javascript
	$('#launch').on('click', function (event) {
	     event.preventDefault();
	     $('#stripe-widget').openModal();
	     getAmount.attachTo('#stripe-widget'); 
	});
```
## Meet Stripe

In a nutshell Stripe is the PayPal you always imagined, made with developers in mind. Stripe is used in many websites and apps to handle payments, Twitter and Kickstarter to name a few. They offer a range of services or API's (whatever you want to call them), but the one we're using is [Checkout](https://stripe.com/checkout).  

After Stripe has verified someone's credit card your website will receive a token back, this token then needs to be sent to Stripe along with your secret key. Since your key is secret this can't happen on the front-end, because JavaScript wasn't and isn't meant to be, at least in its original form,  secure. To do this you can use one of Stripe's libraries for [PHP](https://stripe.com/docs/checkout/guides/php), [Sinatra](https://stripe.com/docs/checkout/guides/sinatra), [Python(Flask)](https://stripe.com/docs/checkout/guides/flask) or [Rails](https://stripe.com/docs/checkout/guides/rails).

I mentioned keys right? Well to get a key you need to [sign up](https://stripe.com/) for a free Stripe account. You don't even need a credit card yourself, a plain old bank account will do! 

Everything should be working up to the point when you click on the donate button, and nothing happens. Let's add the last bit of magic. 
```javascript
	var launchStripe = flight.component(function () {
	// Magic
	});
```
This component will launch Stripe for us. In order to do this we first have to create a Stripe instance at the top of our document (not within a component).
```javascript
	var handler = StripeCheckout.configure({
	key: 'pk_test_hue7wHe5ri0xzDRsBSZ9IBEC',
	image: 'http://freedesignfile.com/upload/2014/06/Cup-of-coffee-design-vector-material-03.jpg',
	locale: 'auto',
	token: function(token) {
	    console.log(token);
	    var html = 'Thank you!       <i class="fa fa-beer"></i>';
	    Materialize.toast(html, 3000)
	    // Send to server
	}
	});
```
Now let's get back to our component. We are going to add the initialize method first and make it listen for the event we triggered in our second component.
```javascript
	   this.after('initialize', function () {
	       this.on('callStripe', this.launch);
	   });
```
It's not that complicated. It's simply listening for an artificial, an internally triggered, event rather than a DOM event. All we have to do now is create the callback, which will launch Stripe
```javascript
	   this.launch = function (event, data) {
	       $('#stripe-widget').closeModal();
	           handler.open({
	              name: 'the Coffeehouse',
	              description: 'Thank You!',
	              currency: "usd",
	              amount: data.amount
	            });
	   };
```
Our function is expecting two arguments, the event and the data. The event is the same as usual but the data includes the payload we included upon initially triggering the event in the previous component. The payload is simply our amount, because that's all we added to the event. But in more complex cases this could be a full response from an API call. 

The other thing we're doing is launching Stripe's payment form using the amount we got from the previous component. You can find the full Checkout documentation [here](https://stripe.com/docs/checkout).

Finally we need to initialize the last component. Instead of attaching it to a specific element we're hooking it up to the document. 
```javascript
	launchStripe.attachTo(document);
```
Our finished code should look something like [this](github-link), we've done quite a lot in less than 100 lines of JavaScript.

## Conclusion 

Hopefully Flight makes a little sense by now, whatever the case may be you can find the full documentation here. As you can see Flight's component system results in extremely readable and reusable code. For example you could reuse the launchStripe component anytime you wanted to process a payment, or reuse the enableButton component anytime you wanted a checkbox to be checked before enabling a button. Flight is great like this, and because it doesn't prescribe a particular approach the possibilities are endless. 
