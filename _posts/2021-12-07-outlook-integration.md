---
title: Getting Outlook Calendars using Python
date: 2021-12-07 12:25:00 +0500
categories: [Programming, Python]
tags: [microsoft 365, outlook, personal experience]
pin: false
comments: true
---

In this post, I would like to share my experience that I received while working on a university project. 

![Meme](/assets/img/meme/working_on_project.jpg){: .shadow}
_I'm working on this project_

## A little bit about my project

The work in this project was close to real. We had a [customer](https://www.parma.ru/) and a nice task: to develop a telegram bot integrated with the Outlook calendar, which helps an HR manager to hire employees.
My role was a Python Developer and my task was to write a small service which can fetch and modify events from someone's Outlook calendar in organization.

![Microsoft Graph](/assets/img/sample/outlook-integration/microsoft.png){: .shadow}
_Microsoft Graph_

In order to complete the task I decided to use [O365 Python library](https://github.com/O365/python-o365), which makes communicating with Microsoft Outlook API very straight forward.
I recommend reading the documentation before using it. So, next comes tutorial on how to get events from the Outlook calendar.

## Step 1: Create an organization

An easy way to get an organization with all Office 365 apps licenses is joining [the Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program).
It's free for 90 days. You will be required to register account if you don't have one. Then fill some information about your company. 
At the end of setting, I recommend you to choose `Instant sandbox` just to save time and focus on future development.

![Choice](/assets/img/sample/outlook-integration/choice.png){: .shadow}
_Choice_

Also, on the [dashboard](https://developer.microsoft.com/en-us/microsoft-365/profile) you can find some interesting articles, quick starts and tutorials.

## Step 2: Creating an app in Azure Portal

Now you should [sign in](https://portal.azure.com) with admin account of your organization, then open the `App registrations`:

![App registrations](/assets/img/sample/outlook-integration/app_registrations.png){: .shadow}
_Azure Portal_

Create a new registration:

![New registration](/assets/img/sample/outlook-integration/new_registration.png){: .shadow}
_App registration window_

On the next page, name the app as you want, choose supported account types as `Any Azure AD directory` and set the web redirect URI to:
<https://login.microsoftonline.com/common/oauth2/nativeclient>

![Webredirect](/assets/img/sample/outlook-integration/webredirect.png)
_The registration page_

You've registered an app and now you should pay attention on two tokens that will be required to use the API. 
You can find them in `App registration`, then choose your app and click on `Overview` on the left toolbar.

![Required tokens](/assets/img/sample/outlook-integration/required_tokens.png){: .shadow}
_App overview page_

Also you need to generate a secret key.

![Left toolbar](/assets/img/sample/outlook-integration/left_toolbar.png){: .shadow}
_Toolbar_

Then click on `new client secret`, fill the description and set expiration. A new line will be appear under Client secret section.
**Copy the secret key in the value column and save it. You should store it in secure location.**

Now click on `API permissions` on the left toolbar and add a permisions.

![API permissions](/assets/img/sample/outlook-integration/api_permissions.png){: .shadow}
_Microsoft Graph API_

After you've done it, don't forget to grant admin consent.

## Step 3: Coding a service

Open your prefered IDE for me it's Pycharm. Install the library with following command.

```console
pip install O365
```

At the beginning, you import the library and set tokens grabbed from Azure Portal.

```python
from O365 import Account

CLIENT_ID = 'your client id'
SECRET_ID = 'your secret id'
TENANT_ID = 'your tenant id'
credentials = (CLIENT_ID, SECRET_ID)
```

Then try to authenticate.

```python
account = Account(credentials, auth_flow_type='credentials', tenant_id=TENANT_ID)
if account.authenticate():
    print('Authenticated!')
```

If everything is okay you will get an access token. Just a txt file in your working folder. Now you should choose some user in your organization. 
You can see them all in [Azure Active Directory](https://portal.azure.com/?l=en.en-us#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/MsGraphUsers).
Now you can easily get someone's Outlook events.

```python
schedule = account.schedule(resource='user principle name from azure active directory')
calendar = schedule.get_default_calendar()
events = calendar.get_events(include_recurring=False)
for event in events:
    print(event)
```
Make sure that you've signed in with someone's account and filled events.

## What's next? 

As I said, read the documentation of the library. Try to parse events for providing more detailed and clean information about them.
I'll just show you what I wrote for our project.

```python
import datetime as dt
import asyncio
from functools import wraps, partial
from os import environ
from O365 import Account


def async_wrap(func):
    @wraps(func)
    async def run(*args, loop=None, executor=None, **kwargs):
        if loop is None:
            loop = asyncio.get_event_loop()
        pfunc = partial(func, *args, **kwargs)
        return await loop.run_in_executor(executor, pfunc)
    return run


class CalendarHelper:

    def __init__(self):
        self._account = Account(credentials=(environ['CLIENT_ID'], environ['SECRET_ID']),
                                auth_flow_type='credentials', tenant_id=environ['TENANT_ID'])

    @async_wrap
    def __get_schedule(self, hr_email):
        return self._account.schedule(resource=hr_email)

    @async_wrap
    def __get_calendar(self, schedule):
        return schedule.get_default_calendar()

    @async_wrap
    def __new_query(self, calendar):
        query = calendar.new_query('start').greater_equal(dt.datetime.today())
        query.chain('and').on_attribute('end').less_equal(dt.datetime.today() + dt.timedelta(days=8))
        return query

    @async_wrap
    def __auth(self):
        self._account.authenticate()

    @async_wrap
    def __get_events(self, calendar, query):
        return calendar.get_events(query=query)

    @async_wrap
    def __get_event(self, calendar, event_id):
        return calendar.get_event(event_id)

    @async_wrap
    def __event_save(self, event):
        event.save()

    async def weekly_free_time(self, hr_email):
        """
        Get weekly free time in Outlook calendar.
        :param hr_email: Provide the email of the person whose events you want to receive.
        :return: List of events with event.subject = 'Свободное время'.
        """
        if not self._account.is_authenticated:
            await self.__auth()
        schedule = await self.__get_schedule(hr_email)
        calendar = await self.__get_calendar(schedule)
        query = await self.__new_query(calendar)
        return [event for event in await self.__get_events(calendar, query) if event.subject == 'Свободное время']

    async def update_event(self, hr_email, event_id, new_subject):
        """
        Updates an existing event by its ID.
        :param hr_email: Provide an email of the person whose event you want to update.
        :param event_id: ID of event.
        :param new_subject: A new subject of event that will replace the old one.
        :return:
        """
        if not self._account.is_authenticated:
            await self.__auth()
        schedule = await self.__get_schedule(hr_email)
        calendar = await self.__get_calendar(schedule)
        event = await self.__get_event(calendar, event_id)
        event.subject = new_subject
        await self.__event_save(event)
```