# Metarank Demo




## Introduction:

The recommendation engine used is
[<u>metarank</u>](https://github.com/metarank/metarank) which relies on
telemetry data in the form of json events to train the model and to make
the predictions. When deployed, Metarank is hosted on a server and works
by sending API event requests to its URLs and the engine returns
recommendations for the received events.

## Event format for the recommendation engine:

The recommendation engine works by sending various events to the APIs
and receiving responses back from them.

There are 4 types of events that can be sent to the engine:

1.  Item metadata events. They describe what should be known about items

2.  User metadata events. They describe what we may know about visitors

3.  Ranking events: What was presented to the visitor

4.  Interaction events. What visitors did with the ranking (clicks)

<!-- -->

###  Item metadata events :

This event is to provide metadata for any item. Metadata of all items
need to be included in training and if predictions for any new item
(item that wasn’t included in training) is required, their metadata
event should be sent to the feedback API.


The keys are :

**Event**: This is to describe which type of event it is, it should
always be ‘item’ for item events

**item**: This is the id of the item about which the information is
provided. It should match the item name in the ranking and interaction
events.

**id**: This is an index for the item field and should be unique. It can
be same as the item

**timestamp**: Timestamp is in epoch time (milliseconds). The timestamp
for item/user events should be before they appear in any of the rankings
and have any interactions

**fields:** This gives additional information about the features of the
item. Each feature like name,tag_2 etc are fed to name and the
corresponding value is fed to ‘value’. If there are multiple values for
a feature, they are made into a list

###  User metadata events:

The event is to provide metadata for any new users that the model wasn’t
trained on. The format/ structure of the event is the same as the item
event except that the event tag is ‘user’ instead of ‘item’



**event**: This is to describe which type of event it is, it should
always be ‘user’ for item events

**user**: This is the id of the user about which the information is
provided.It should match the item name in the ranking and interaction
events

**id**: This is an index for the user field and should be unique. It can
be same as the user

**timestamp**: Timestamp is in epoch time (milliseconds). The timestamp
for item/user events should be before they appear in any of the rankings
and have any interactions

**fields:** This gives additional information about the features of the
user. Each feature like district, crops grown etc are fed to name and
the corresponding value is fed to ‘value’. If there are multiple values
for a feature, they are made into a list

###   Ranking event:

The event provides the list of items shown to the user.


**event**: This is to describe which type of event it is, it should
always be ‘ranking’ for ranking events

**user**: The user to which the ranking list was shown

**items**: List of items that were shown to the user. If we have some
previous knowledge of the relevancy of the items, that can be fed here.
However, we always consider relevancy as 0

**id**: All ranking events have an associated id which should be unique

**session:** This provides the session id for the user. A user can have
multiple sessions. In our current model, users and sessions are the
same, each user is considered to be having one session only

**Tenant**: This is to allow for multi-tenancy cases. In our case, it
always has the value as ‘default’

**Fields**: This is for any additional fields for the ranking event.
This is always an empty list in our model for ranking events

**Timestamp:** Timestamp is in epoch time (milliseconds). The timestamp
of a ranking event should be before the corresponding interaction. User
must view the list of items before interacting with any of them

###   Interaction event:

The event provides the response of the user (interactions) to the list
of items shown to the user. Only click events are supported currently
for the IVRS model.


**event**: This is to describe which type of event it is, it should
always be ‘interaction’ for interaction events

**user**: The user to which the ranking list was shown and the user
interacted

**item**: The item with which the user interacted. The item should be a
part of the list of items in the corresponding ranking event

**ranking**: Ranking id of the list of events shown to the user

**id**: All interactions events have an associated id which should be
unique

**session:** This provides the session id for the user. A user can have
multiple sessions. In our current model, users and sessions are the
same, each user is considered to be having one session only

**Tenant**: This is to allow for multi-tenancy cases. In our case, it
always has the value as ‘default’

**type**: This is for the type of interaction event. This should always
be ‘click’ for our model

**Timestamp:** Timestamp is in epoch time (milliseconds). The timestamp
of an interaction event should always be after the corresponding
interaction. User must view the list of items before interacting with
any of them

**Fields**: This is for any additional fields for the interaction event.
This is always an empty list in our model for interaction events

## Hosting of the recommendation engine and feedback URL’s 

The recommendation engine is hosted at ***localhost:8080***

It has two URLs for user interaction (where \<ip\> is the IP address of the server where the recommendation engine is hosted):

1.  Feedback URL: ***http://\<ip\>:8080/feedback***

2.  Ranking URL: ***http://\<ip\>:8080/rank/xgboost***

There are two types of events that one can use to interact with the recommendation engine:

1.  Feedback events : Any of the events above can be sent to the ***feedback url.*** These update the model with new information. We send interaction events to let the model know about the latest interactions of the user and the model modifies the ranking/recommendations for the user accordingly. These can also be used to add new items/users metadata to the model

2.  Recommendation events: Only ranking events can be sent to the ***ranking url.*** These can be sent as a POST request. For each ranking event, the recommendation engine returns the list of items in the ranking event in the order of the recommendation/ranking with the corresponding scores. These can be used as the recommendations of content for each user to be used for the IVRS calls.

## Deploying the recommendation engine 

The metarank recommendation engine can currently be run through docker
using the created repo

Before running the recommendations, the [events
folder](https://github.com/DataGov-SamagraX/KO_Metarank_implementation/tree/main/metarank/events) should be updated with the latest data before each run

Events folder needs to be updated with gzip json files which have :

- Latest user metadata

- Latest content metadata

- User interactions for the past year

To create these gzip files in the event location, one can use the
created [<u>python
script</u>](https://github.com/DataGov-SamagraX/KO_Metarank_implementation/blob/main/event_creation/Creating%20interactions%20and%20ranking%20events.ipynb).
The script 4 inputs:

- Location of csv file with the latest user metadata

- Location of csv file with content metadata

- Location of directory containing the user interactions for the past year

- Location of events directory where the created events need to be stored

These are fed as variables in the python script

Summarizing the above as sequential steps for running the system:

1.  Clone the github repofor running metarank

2.  Download the [<u>python
script</u>](https://github.com/DataGov-SamagraX/KO_Metarank_implementation/blob/main/event_creation/Creating%20interactions%20and%20ranking%20events.ipynb) for creating events

3.  Specify the file location for the content /user metadata and the interactions files along with the event location of the downloaded metarank repo within the python script

4.  Run the Python script to create the jsonl.gz files within the events folder of metarank repo

5.  Run ``` docker-compose -f compose_metarank_bootstrap.yml up ``` to run the bootstrapping. This will create the bootsrap folder too.

6.  Run ``` docker-compose -f compose_metarank_train.yml up ``` to run the model training.  This will create a trained model. 

7.  The above steps are the offline training of the model and need to be done at pre-decided frequency (weekly/monthly etc)

8.  Before deploying the model, one needs to first start redis and upload the model there.  This is done by running ``` docker-compose -f compose_metarank_upload.yml up ```

9. To deploy the model, run ```docker-compose  up  ```. Metarank is up and running now 


10.  The user/content for which predictions need to be made should be pushed using the above event format to feedback/ranking URL and metarank will return the recommendations.

## Changing model parameters:

There are 3 kinds of changes that one can make to the model :

###  Features required to be modelled:

All features to be included in the model need to be specified in the
config file. This is done in models → xgboost → features as shown below

- <img src="doc/media/image1.png"   style="width:2.88021in;height:3.17178in" />

Any new features can be added to the list as required.

###  Modifying the features:

 One can also modify the definition of each features in the config file
 under features:

 Example:

 <img src="doc/media/image2.png" style="width:2.39063in;height:3.90975in" />

 where:
-  Scope: The metadata that needs to be accessed for the feature (session/item). Session is same as user
-  Type: The type of features that can be created :
-  Rate: features that have numerator and denominator
-  Interacted with: features that check if user has interacted with a feature in the time period (specified inside duration)
-  Interaction count: Feature that number of interactions for the item
-  Window count : Features that count number of interactions for the item within defined time frameworks (past 2 months, past 1 week etc)

Bucket/Period: Used to define window count/impression feature characteristics. E.g. Window count with bucket 24h and \[30,60\] means features counting the average clicks in past 30 days and past 60 days

Count: used to establish how often to refresh the click metrics. E.g. If count is 5, metarank will check the number of clicks in last 5 seconds every 5 seconds and update the model

###  Setting the cutoff of binary clicks:
 This is defined in the variable eng_ratio_cutoff in the interaction  event creation notebook. Its currently set to 0.858

