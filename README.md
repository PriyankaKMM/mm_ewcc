# mm_ewcc
Find and Display some User specific Missed Articles from Editor's pick section on the site based on their IP Address and Article visit.

# use case:
Find and Display some User specific Missed Articles from Editor's pick section on the site based on their IP Address and Article visit.

# Team: 
Raja Vijay Singh G
Joseph Mathew
Tom C Antony
Priyanka K
Randeep R

# Description:
We have an Editor's pick section on the Home page of our site https://ewcc09.ewcc.in/home.html where the the hand picked articles get displayed. 
These article list gets changed over time based on the Editor's choice.
Sometimes a user might miss some articles from these Editors Pick if he visits the site later.
So our idea is to implement a logic where we will keep track of the Editor's pick and each user's article visits through Akamai Edgeworker.
For identifying the user we use the Client IP address and we keep reference of Visited and Missed articles to this user IP whenever the user hits on aN Editor's Pick article URL.
We find the missed articles by comparing the User visited URLs and the Editor's Pick URLs available on the EdgeKV item and displayes on the visited Editor's Pick article page.
we also remove older article URLs based on the timestamp from the EdgeKV items on each users visit.

# Highlight: 
This user specific analytic feature was one of the most interested use-cases that our editorial team was looking for and we were able to implement this without changing  anything on the original website components and data. This is completely handled by using Akamai Edgeworker and EdgeKV. This was a completely new experience for us to implement such functionality on a lighter way which could otherwise be an additional load on our server. 

# Steps to reproduce the use-case:

 1. Go to https://ewcc09.ewcc.in/home.html where you can see the Editor's Pick section. Here, we set the Unique UserID from the client IP and the Editors pick data parsing the home page HTML and store in cookies.
 2. Click on any of the articles from the Editor's Pick , you will get redirected to the selected article page.
 3. you might have to refresh the page for a few times to load the article content. This seems to be a timeout issue with the Edgeworker.
 4. you will see the missed Editor's pick article list on the bottom right corner of the page. this is appicable only for the Editor's pick articles. Other articles remiain the same.
 5. Click on any article from this list and that article page gets loaded and it gets removed from your missed article list. the missed article list will be populated with remaining set of missed articles.
 6. User can also visit any Editor's pick article directly without touching the homepage( sometimes redirected from social media sites)
 7. on this scenario, if the user is accessing the site for the very first time, then the missed articles gets populated only on the second visit of the user.
 8. This is because, we keep track of the user based on the IP Address and it gets set on the User's first visit only(In normal scenario this gets updated on the homepage itself).


# Technical details and Functionality:
 
namespace: ewcc
groupname: mmonline
items: daily_pick, user_visits

Edgeworker 1: EWCC09

  this edgeworker is used for the homepage where, on the responseProvider we inject a new script to the page body. 
  On this script we fetch the user IP address and the current editors pick article urls(parsing the html) and store as cookies. 
  
Property for EWCC09: if the url path matches /home.html
  
Edgeworker 2: EWCC09-2

 this edgeworker is used on the article pages. 
 here on the responseProvider we fetch the user IP(akamai_user_id) and Editor's Pick(akamai_daily_pick) cookies which were saved from the home page. 
 if the User IP cookie is not available we set the cookie the same way we did on the homepage on the response.
 when the cookies are available, we fetch the data from the edgeKV items - Editor's Pick(daily_pick) and User Visit(user_visits) 
 if the cookies are available we update the Editor's Pick item with the corresponding Editor's Pick cookie if the article urls from the cookie is not available on the fetched items, we add just create a new entry for it on the Editors pick item.
 we also remove the older article urls  from the editors pick item by checking the timestamp difference.
 similarly if the user cookie is availabe and the visited article url is present on the Editors Pick item data, we check if there is an entry for this user on the UserVisit item. if not we create a new entry for this user with the visited node consisting of the path and timestamp.
 if the user entry is already there, then we loop the User Visit item data and check if the visited url is present on the 'visited' node of this user data.
 if the user has already visited the url, we will update entry with the current timestamp.
 if the user has not visited the url, then we create a new entry on the visited node of the user.
 here also, we remove the older article urls from the user's visited node by checking the timestamp difference.
 after updating both the items we find the missed articles by comparing the Editor's pick data with the user's visited node data.
 we sort the missed articles based on the timestamp and a max of 10 missed artilces are populated for the user.
 we save this missed articles data as a missed node to the corresponding user entry. 
 we pass this missed article populated to the HTML stream which appends this data to the article HTML Body as a popup through the response.
 
 Property for EWCC09-2: if the url path matches one of the below conditions
 /news/*, /entertainment/*, /sports/*, /lifestyle/*, /career-and-campus/*, /food/*, /travel/*
