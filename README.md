## Required Project Dependencies
Python 3 is required for this tutorial because Python 2 will no longer be supported after December 31, 2019. We’ll use Python 3.8 to build this tutorial.

We will also use the following application dependencies in our application:

Django web framework, version 2.2.x
pip and virtualenv
The Twilio Python helper library, version 6.32.0 or greater
A free Twilio account. Sign up to get an account so you can use the Twilio APIs
If you need help getting your development environment configured before running this code, take a look at this guide for setting up Python 3 and Django on Ubuntu 16.04 LTS.

# This blog post's code is also available on GitHub within the django/djsms directory of the python-twilio-example-apps Git repository on GitHub.

# Installing Python Project Dependencies
Our code will use the Twilio helper library to make it easier to send text messages from Python. We are going to install the helper library from PyPI into a virtualenv.

But first we need to create the virtual environment. In your terminal use the following command to create a new virtualenv and activate it:

python3 -m venv smspy3
source smspy3/bin/activate
Install Django and the Twilio helper library:

pip install django==2.2.6 twilio==6.32.0
When the command is finished you should see some output similar to the following lines:

Successfully installed django-2.2.6 pytz-2019.3 sqlparse-0.3.0
PyJWT-1.7.1 certifi-2019.9.11 chardet-3.0.4 idna-2.8 requests-2.22.0 six-1.12.0 twilio-6.32.0 urllib3-1.25.6
At this point, we’ve installed the required dependencies and can now use them to create our project.

## Creating the Django Project
We'll start our project by using Django's django-admin tool to create a boilerplate code structure.



Run the following command to start a Django project named djsms and change into the newly-created directory:

django-admin startproject djsms
cd djsms
Create a new Django app named broadcast within the djsms project:

python manage.py startapp broadcast
Django generates a new folder named broadcast after the command finishes. We need to update the project's settings and URLs files, so make sure the broadcast app is available.

## Open djsms/settings.py:

ALLOWED_HOSTS = []


TWILIO_ACCOUNT_SID = os.getenv("TWILIO_ACCOUNT_SID")
TWILIO_AUTH_TOKEN = os.getenv("TWILIO_AUTH_TOKEN")
TWILIO_NUMBER = os.getenv("TWILIO_NUMBER")
SMS_BROADCAST_TO_NUMBERS = [ 
    "", # use the format +19735551234
    "", 
    "", 
]


# Application definition

INSTALLED_APPS = [ 
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'broadcast',
]
Make sure you change the default DEBUG and SECRET_KEY values in settings.py before you check in your files to Git or deploy any code to production. Secure your app properly with the information from the Django production deployment checklist so that you do not add your project to the list of hacked applications on the web.

## Save and close settings.py. Open the djsms/urls.py file and add the broadcast app's URLs to your main project URLs list:

from django.conf.urls import include
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('', include('broadcast.urls')),                                                                                                                                                 
    path('admin/', admin.site.urls),
]
The above two new lines of code connect the main urls.py file to the broadcast/urls.py file which will write next.

Coding the Broadcast App
Next change into the broadcast directory. Create a new file named urls.py to contain a URL route for sending SMS:

cd broadcast
touch urls.py
Open the empty urls.py file and add these lines of code:

from django.conf.urls import url                                                                                                                                                         
from . import views

urlpatterns = [ 
    url(r'broadcast$', views.broadcast_sms, name="default"),
]
In the above code, we have a single URL route that matches broadcast as a path that maps to a yet-to-be-written broadcast_sms view function.

We need to write that function within broadcast/views.py to handle sending the messages. Save urls.py and open views.py, then update it with the following lines:

from django.conf import settings                                                                                                                                                       
from django.http import HttpResponse
from twilio.rest import Client


def broadcast_sms(request):
    message_to_broadcast = ("Have you played the incredible TwilioQuest "
                                                "yet? Grab it here: https://www.twilio.com/quest")
    client = Client(settings.TWILIO_ACCOUNT_SID, settings.TWILIO_AUTH_TOKEN)
    for recipient in settings.SMS_BROADCAST_TO_NUMBERS:
        if recipient:
            client.messages.create(to=recipient,
                                   from_=settings.TWILIO_NUMBER,
                                   body=message_to_broadcast)
    return HttpResponse("messages sent!", 200)
In this above code we import Django's settings file so we can access our project's settings as well as the HttpResponse class to return a simple HTTP 200 status code with a text response. We also import the Twilio Python helper library.

The broadcast_sms function handles the bulk of the work. It specifies a message to send, instantiates the helper library client, and loops through each phone number listed in the SMS_BROADCAST_TO_NUMBERS variable. If a phone number is specified (not blank) it will call the Twilio API to send the SMS to the recipient's phone number. After the loop completes we return an HTTP 200 response with the text "messages sent!".

Before we test our application we need to obtain a phone number from Twilio and credentials to authenticate ourselves to the API.

Access the Twilio SMS API
Sign into your existing Twilio account or sign up for a free Twilio account.

After the registration (or login) process you’ll be in the Twilio Console where you can access your Account SID and Auth Token, as shown below:

How to get Account SID and Auth Token from the Twilio Console

Set the Account SID and Auth Token as environment variables for your Django application to read and use. Create a new file named .env and paste in the following contents.

(You can also copy the template.env file included with the Git repository under the django/djsms/ directory and rename it to .env.)

# Django SECRET_KEY for sessions                                                                                                                              
SECRET_KEY='development key' # change this to a secret string when deploying

# Twilio credentials and phone number
TWILIO_ACCOUNT_SID=' ' # obtained from twilio.com/console
TWILIO_AUTH_TOKEN=' ' # also obtained from twilio.com/console
TWILIO_NUMBER=' ' # use the number you received when signing up or buy a new number
Populate the TWILIO_ACCOUNT_SID and TWILIO_AUTH_TOKEN variables with your credentials from the Console. You should also have a Twilio phone number from the registration process, or you can purchase a new phone number to use for this tutorial.

Invoke .env file to set these variables:

source .env
Note that if you are on Windows, setting environment variables is slightly different so take a peek at this tutorial for those detailed instructions.

We have our code and credentials in place, now it's time to give the project a spin.

Testing an SMS Broadcast
Fire up the Django development server:

python manage.py runserver
Go to https://localhost:8000/broadcast in your web browser to kick off sending the messages. In the browser you should see:

Messages sent message in the browser