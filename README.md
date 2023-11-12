# Django + Stripe

[Sebastian Sanchez Bernal is Code](https://www.buymeacoffee.com/bastiansanber)

## Course Title: Working with Stripe in Django

Welcome to "Working with Stripe in Django"! This course will guide you through the process of integrating Stripe, a powerful online payment platform, into your Django application. We will cover everything from setting up Stripe in Django, to creating checkout sessions, and handling successful and cancelled payments.

We will also delve into the advanced topic of handling Stripe webhooks. These are essential for responding to events in the Stripe system, such as successful payments or subscription changes.

One key aspect we will explore is the subscription model. This involves recurring billing, which is a crucial feature for many online businesses. To give you a better understanding, an example of a subscription model will be provided as an image after this introduction.

By the end of this course, you will have a solid understanding of how to implement Stripe in your Django projects, enabling you to handle online payments with ease and efficiency.

![Screenshot 2023-11-09 at 8.25.12 PM.png](Django%20+%20Stripe%206228afac55b743f79afcdd281e250dc5/Screenshot_2023-11-09_at_8.25.12_PM.png)

### Lesson 1: Introduction to Stripe

- In this course, "Working with Stripe in Django," we begin with an introduction to Stripe. Here, we will dive deep into understanding what Stripe is and how it operates. We will also explore key features and benefits that Stripe offers, providing a thorough overview to prepare you for the more technical aspects to follow.

### Lesson 2: Setting Up Stripe in Django

First, import the Stripe module into your Django project. Then, create a Django setting that will store your Stripe public and secret keys. You can retrieve these keys from your Stripe dashboard.

```python
STRIPE_PUBLISHABLE_KEY = 'your-publishable-key'
STRIPE_SECRET_KEY = 'your-secret-key'

```

This code block imports necessary libraries and modules for the application. Here's a breakdown:

`import stripe` - Imports the Stripe library, allowing the application to use Stripe's features.

`from django.conf import settings` - Imports Django's settings module, allowing the application to access and use Django's settings.

`from django.contrib.auth.decorators import login_required` - Imports the `login_required` decorator from Django's authentication system. This decorator is used to ensure that only logged-in users can access certain views.

`from django.contrib.auth.models import User` - Imports Django's built-in User model. This model is used to manage users in the application.

`from django.http.response import JsonResponse, HttpResponse` - Imports Django's JsonResponse and HttpResponse. These are used to return HTTP responses from views.

`from django.shortcuts import render, redirect` - Imports Django's render and redirect functions. `render` is used to display a template with context data, while `redirect` is used to redirect the user to a different webpage.

`from django.views.decorators.csrf import csrf_exempt` - Imports Django's `csrf_exempt` decorator. This decorator is used to exempt certain views from Django's CSRF protection.

`from .models import StripeCustomer` - Imports the StripeCustomer model from the application's models. This model is used to manage Stripe customers in the application.

`from django.views.decorators.http import require_GET, require_POST` - Imports Django's `require_GET` and `require_POST` decorators. These decorators are used to ensure that a view can only be accessed with the respective HTTP method.

`from docreation.src.contact_email import support_email, enterprise_contact` - Imports `support_email` and `enterprise_contact` from the application's contact_email module. These are likely used for sending emails in the application.

```python
# views.py
import stripe
from django.conf import settings
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User 
from django.http.response import JsonResponse, HttpResponse 
from django.shortcuts import render, redirect
from django.views.decorators.csrf import csrf_exempt
from .models import StripeCustomer
from django.views.decorators.http import require_GET, require_POST
from docreation.src.contact_email import support_email, enterprise_contact
```

The `MEMBERSHIP_CHOICES` is a tuple containing tuples that represent different membership choices for users. Each inner tuple contains two elements: a descriptive name for the membership type and a short code representing the membership type.

Here's a breakdown of each membership choice:

- `('Enterprise', 'ent')`: This represents an Enterprise membership. 'ent' is the short code used to represent this in the database.
- `('Professional', 'pro')`: This represents a Professional membership. 'pro' is the short code used to represent this in the database.
- `('Free', 'free')`: This represents a Free membership. 'free' is the short code used to represent this in the database.

```python
# models.py
from django.contrib.auth.models import User
from django.db import models
from django.conf import settings

MEMBERSHIP_CHOICES = (
    ('Enterprise', 'ent'),
    ('Professional', 'pro'),
    ('Free', 'free')
)

class StripeCustomer(models.Model):
    user = models.ForeignKey(to=User, on_delete=models.CASCADE)
    stripeCustomerId = models.CharField(max_length=255)
    stripeSubscriptionId = models.CharField(max_length=255)
    stripeSubscriptionStatus = models.CharField(max_length=255)

    def __str__(self):
        return self.user.username
```

### Lesson 3: Creating Checkout Sessions

Next, create a function to handle checkout sessions. This function first retrieves the current user's subscription status. If the user is already subscribed, the function modifies the user's subscription. If not, the function creates a new checkout session for the user.

GitHub Copilot: Sure, here's a detailed explanation of the two functions `create_checkout_session` and `create_checkout_session2`:

1. **create_checkout_session function**

This function is responsible for creating a new Stripe checkout session for the authenticated user and returning the session ID as a JSON response.

- It first sets the domain URL and the Stripe API key.
- It then tries to retrieve the Stripe customer associated with the authenticated user and their subscription.
- If the customer and subscription exist, it modifies the subscription to use a different price ID (STRIPE_PRICE_ID2). It also sets the subscription to cancel at the end of the billing period and allows the use of promotion codes.
- If the customer or subscription doesn't exist, it creates a new checkout session with the price ID (STRIPE_PRICE_ID2), and returns the session ID.
- If there's an error during the creation of the checkout session, it returns the error message.
    1. **create_checkout_session2 function**

This function is similar to `create_checkout_session`, but it modifies the user's existing Stripe subscription to use a different price ID (STRIPE_PRICE_ID).

- It first sets the domain URL and the Stripe API key.
- It then tries to retrieve the Stripe customer associated with the authenticated user and their subscription.
- If the customer and subscription exist, it modifies the subscription to use a different price ID (STRIPE_PRICE_ID). It also sets the subscription to cancel at the end of the billing period and allows the use of promotion codes.
- If the customer or subscription doesn't exist, it creates a new checkout session with the price ID (STRIPE_PRICE_ID), and returns the session ID.
- If there's an error during the creation of the checkout session, it returns the error message.

In both functions, the `cancel_url` is set to a specific URL in your application where users are redirected if they decide to cancel the payment process. The `success_url` is where users are redirected after a successful payment. The `{CHECKOUT_SESSION_ID}` placeholder in the `success_url` is automatically replaced by Stripe with the ID of the checkout session.

```python
# views.py
@csrf_exempt
def create_checkout_session(request):
    if request.method == 'GET':
        domain_url = 'https://your_site/'
        stripe.api_key = settings.STRIPE_SECRET_KEY

        try:
            stripe_customer = StripeCustomer.objects.get(user=request.user)
            subscription = stripe.Subscription.retrieve(stripe_customer.stripeSubscriptionId)
            # new_plan =
            
            stripe.Subscription.modify(
                subscription.id,
                cancel_url=domain_url + 'memberships/cancel_now/',
                cancel_at_period_end=True,
                proration_behavior='create_prorations',
                #the new subscription
                items=[{
                    'id': subscription['items'].data[0].id, 
                    'price': settings.STRIPE_PRICE_ID2
                }],
                allow_promotion_codes=True
                )

            # return JsonResponse({'sessionId': new_plan['id']})
        except:
            pass

        try:
            
            checkout_session = stripe.checkout.Session.create(
                client_reference_id=request.user.id if request.user.is_authenticated else None,
                success_url=domain_url + 'memberships/success?session_id={CHECKOUT_SESSION_ID}',
                cancel_url=domain_url + 'memberships/cancel/',
                payment_method_types=['card'],
                mode='subscription',
                line_items=[
                    {
                        'price': settings.STRIPE_PRICE_ID2,
                        'quantity': 1,
                    }
                ],
                allow_promotion_codes=True
            )
            return JsonResponse({'sessionId': checkout_session['id']})
        except Exception as e:
            return JsonResponse({'error': str(e)})

@csrf_exempt
def create_checkout_session2(request):
    if request.method == 'GET':
        domain_url = 'https://your_site/'
        stripe.api_key = settings.STRIPE_SECRET_KEY

        # Error, no esta entrando al modify
        try:
            
            stripe_customer = StripeCustomer.objects.get(user=request.user)
            subscription = stripe.Subscription.retrieve(stripe_customer.stripeSubscriptionId)

            # new_plan = 
            stripe.Subscription.modify(
                subscription.id,
                cancel_at_period_end=True,
                cancel_url=domain_url + 'memberships/cancel_now/',
                proration_behavior='create_prorations',
                #the new subscription
                items=[{
                    'id': subscription['items'].data[0].id, 
                    'price': settings.STRIPE_PRICE_ID
                }],
                allow_promotion_codes=True
                )
        except:
            pass
        
        try:
            
            checkout_session = stripe.checkout.Session.create(
                client_reference_id=request.user.id if request.user.is_authenticated else None,
                success_url=domain_url + 'memberships/success?session_id={CHECKOUT_SESSION_ID}',
                cancel_url=domain_url + 'memberships/cancel/',
                payment_method_types=['card'],
                mode='subscription',
                line_items=[
                    {
                        'price': settings.STRIPE_PRICE_ID,
                        'quantity': 1,
                    }
                ],
                allow_promotion_codes=True
            )
            return JsonResponse({'sessionId': checkout_session['id']})
        except Exception as e:
            return JsonResponse({'error': str(e)})

```

### Lesson 4: Handling Successful and Cancelled Payments

1. **Handling Successful Payments**

This function is decorated with `@login_required`, which means that the user must be authenticated to access this view. If the user is not authenticated, they will be redirected to the login page.

The purpose of this function is to handle the scenario where a user successfully completes a payment. When this happens, the user is redirected to this view.

The function takes a `request` object as an argument, which contains information about the current web request that triggered this function.

The function then uses the `render` function to generate a response. The `render` function takes the `request` object, the name of the template file (`'success.html'`), and returns an HTTP response with that rendered text. In this case, the `'success.html'` template would be used to display a success message to the user.

```python
# views.py
@login_required
def success(request):
    return render(request, 'success.html')

```

1. **Handling Cancelled Payments**

This function is very similar to the `success` function. It's also decorated with `@login_required`, so the user must be authenticated to access this view.

The purpose of this function is to handle the scenario where a user cancels a payment. When this happens, the user is redirected to this view.

Like the `success` function, it takes a `request` object as an argument and uses the `render` function to generate a response. However, in this case, the `'cancel.html'` template is used to display a cancellation message to the user.

```python
# views.py
@login_required
def cancel(request):
    return render(request, 'cancel.html')

```

### Lesson 5: Handling Stripe Webhooks

Webhooks in Stripe are used to notify your application when an event happens in your account. To handle these events, create a function that listens for webhooks. This function will handle different types of events, like when a checkout session is completed.

GitHub Copilot: Sure, this function is named `stripe_webhook` and it's designed to handle webhook events sent by Stripe to your application. Here's a detailed breakdown:

1. **Setting up the function**

The function is decorated with `@require_POST` and `@csrf_exempt`. The `@require_POST` decorator ensures that this view can only be accessed via POST requests. The `@csrf_exempt` decorator exempts this view from the CSRF protection, which is necessary because the request will be coming from Stripe, not from a form on your site.

1. **Verifying the webhook signature**

Stripe signs the webhook events it sends to your endpoint. The function retrieves the signature from the `Stripe-Signature` header and uses it along with the payload and your endpoint's secret to construct the event. If the signature or the payload is invalid, it returns a 400 status code.

1. **Handling the `checkout.session.completed` event**

If the event type is `checkout.session.completed`, it means a checkout session has been completed. The function retrieves the session object from the event data.

1. **Fetching data from the session**

The function fetches the client reference ID, customer ID, subscription ID, and subscription status from the session object.

1. **Updating or creating a `StripeCustomer`**

The function then tries to get a `StripeCustomer` object associated with the user ID (client reference ID). If it exists, it deletes the existing Stripe subscription, updates the `StripeCustomer` object with the new customer ID, subscription ID, and subscription status, and saves the object. If the `StripeCustomer` object does not exist, it creates a new one with the fetched data.

1. **Returning a response**

Finally, the function returns an HTTP response with a status code of 200 to acknowledge receipt of the webhook event.

This function allows your application to react to events in the Stripe system, such as successful payments or subscription changes.

```python
# views.py
@require_POST
@csrf_exempt
def stripe_webhook(request):

    stripe.api_key = settings.STRIPE_SECRET_KEY
    endpoint_secret = settings.STRIPE_ENDPOINT_SECRET
    payload = request.body
    sig_header = request.META['HTTP_STRIPE_SIGNATURE']
    event = None

    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, endpoint_secret
        )
    except ValueError as e:
        # Invalid payload
        return HttpResponse(status=400)
    except stripe.error.SignatureVerificationError as e:
        # Invalid signature
        return HttpResponse(status=400)

    # Handle the checkout.session.completed event
    if event['type'] == 'checkout.session.completed':
        session = event['data']['object']

        # Fetch all the required data from session
        client_reference_id = session.get('client_reference_id')
        stripe_customer_id = session.get('customer')
        stripe_subscription_id = session.get('subscription')
        stripe_subscription_status = session.get('status')

        try:
            obj = StripeCustomer.objects.get(user_id=client_reference_id)
            subscription = stripe.Subscription.retrieve(obj.stripeSubscriptionId)
            stripe.Subscription.delete(subscription.id)
            obj.stripeCustomerId = stripe_customer_id
            obj.stripeSubscriptionId = stripe_subscription_id
            obj.stripeSubscriptionStatus = stripe_subscription_status
            obj.save()
            user = User.objects.get(id=client_reference_id)
            print(user.username + ' just change the plan.')
        except StripeCustomer.DoesNotExist:
            user = User.objects.get(id=client_reference_id)
            StripeCustomer.objects.create(
                            user=user,
                            stripeCustomerId=stripe_customer_id,
                            stripeSubscriptionId=stripe_subscription_id,
                            stripeSubscriptionStatus=stripe_subscription_status
                        )
            print(user.username + ' just subscribed.')

    return HttpResponse(status=200)

```

### Lesson 6: Cancelling Subscriptions

This function is named `cancel_now` and it's designed to handle the cancellation of a user's Stripe subscription. Here's a detailed breakdown:

1. **Checking user authentication**

The function first checks if the user is authenticated. If not, it redirects the user to the login page.

1. **Retrieving the Stripe customer**

If the user is authenticated, the function retrieves the `StripeCustomer` object associated with the user. This object contains information about the user's Stripe customer ID and subscription ID.

1. **Setting the Stripe API key**

The function then sets the Stripe API key, which is necessary to make requests to the Stripe API.

1. **Retrieving the subscription**

The function retrieves the user's Stripe subscription using the subscription ID stored in the `StripeCustomer` object.

1. **Modifying the subscription**

Inside a try block, the function modifies the subscription to cancel it at the end of the billing period. This is done using the `stripe.Subscription.modify` method, which takes the subscription ID and a parameter `cancel_at_period_end` set to `True`.

1. **Updating the `StripeCustomer` object**

The function then updates the `StripeCustomer` object's `stripeSubscriptionStatus` field to "cancel" and saves the object. This reflects the new status of the user's subscription in your application's database.

1. **Rendering the cancellation page**

The function then renders the 'cancel.html' template, which could display a message informing the user that their subscription has been cancelled.

1. **Handling exceptions**

If there's an exception during the cancellation process (for example, if the subscription ID is invalid), the function catches the exception and returns a JSON response with the error message and a status code of 403.

This function allows your application to handle subscription cancellations and update the user's subscription status in your database.

```python
# views.py
@csrf_exempt
def cancel_now(request):
    if request.user.is_authenticated:
        stripe_customer = StripeCustomer.objects.get(user=request.user)
        stripe.api_key = settings.STRIPE_SECRET_KEY
        subscription = stripe.Subscription.retrieve(stripe_customer.stripeSubscriptionId)
        try:
            # stripe.Subscription.delete(subscription.id)
            stripe.Subscription.modify(
            subscription.id,
            cancel_at_period_end=True
            )
            # stripe_customer.delete()
            stripe_customer.stripeSubscriptionStatus = "cancel"
            stripe_customer.save()
            return render(request, 'cancel.html')
        except Exception as e:
            return JsonResponse({'error': (e.args[0])}, status =403)

    return redirect('/login')

```

<aside>
💡 Goodbye, and happy coding!

</aside>

[Sebastian Sanchez Bernal is Code](https://www.buymeacoffee.com/bastiansanber)