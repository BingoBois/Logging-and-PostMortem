# Logging-and-PostMortem

# The results


## Summary and description of the assignment.


We've built our system using Kubernetes to facilitate hosting and building of our backend and frontend, as well as our DevOps. We've done this assignment by implementing "Elasticsearch" as the systems logging, and both grafana and Kibana to display the systems current health and state.

While we seemed to have build a solid and secure system, actually getting the thing to crash was a tougher challenge than initial expected. Since we hadn't encounter any particular system crashing errors during development, we had to come up with a way to manually crash the system. 

Our solution to force a crash in the system, was to try and shutdown the MySQL database and see how the backend would react.

This would result in the frontend not being able to login, post data or comments, or for Helge's simulator to post any comments or post, or check-in on /latest.


![grafana screenshot](https://github.com/BingoBois/Logging-and-PostMortem/blob/master/Dolphin-Grafana.png)
*Screenshot of the grafana-dashboard*

![Kibana screenshot](https://github.com/BingoBois/Logging-and-PostMortem/blob/master/Dolphin-Kibana.png)
*Screenshot of the Kibana-dashboard*

As can be seen on the grafana dashboard screenshot above, the 4 pods responsible for hosting and containing the backend went offline 3 times: 18:13-18:14, and subsequently 2 times (briefly) at 18:47 and 19:08

As expected, the crash resulted in post and comments not loading in the frontend and the inability to login to the Frontend.

However, the crash also exposed a serious and critical error-handling mistake and the crash did not just "crash" the system, it totally broke it and the team [group 5] was scratching our heads as to why.

The crash was so bad, that Kibana did not log any data regarding the crash, so something very serious was happening when we shutdown the MySQL server and forced the crash.

To sum up:

*   Successful (manual) crash of the backend
*   Achieved by shutting down the backend
*   The crash exposed serious flaws with error-handling
*   Crash was registered in Grafana
*   Errors was not logged in Kibana

The team also had some trouble adding alerts to grafana, due to the file "grafana.ini" being inaccessible. 

But what was the cause for the system to totally shut down and failure of logging the errors?


## The devil in the machine… and detail

We spent 2-3 hours investigating why the system would just crash and shut down, once we shut down the MySQL-server and why the system did not log the error in Kibana.

We first suspected some kind of request incoming to the system was the cause of the crash, since we knew that Helge was using a simulator to post and retrieve certain data from the live-system. We looked at the incoming request to the live-system and could see that something was not being handled correctly. This led us to further investigate the most used MySQL-queries used in the backend, such as /latest and post. We would then try to recreate the crash running the project locally on our machines and this quickly let us to focus our attention to the methods running our MySQL-queries.  
 
We discovered that if the backend was running and was disconnected from the MySQL-server (achieved by pointing the connection information for the MySQL server in the backend to an empty string) and a request such as a Post or Get was made, the system would just totally crash and burn.

This basically told us, that the system was not error-handling correctly, which also prevented the system from logging error data to Kibana.

After having been running up and down the various MySQL-queries and methods, poking and testing them with console.log to see where the breaking point was, we discovered that the crash was most likely caused by Helge's simulator, when it would try to post a new comment or post. The method handling this request uses various functions, and the team noticed that the when they tried to execute a post to the backend, it would totally crash itself, exactly like when the system was online.

Upon further investigating the issue, the team noticed that the error messages being displayed by the local-versions in the terminal pointed towards a lack of basic error handling by the functions handling mysql-queries.

This baffled the team, since the team have had a big focus on handling and catching of errors in both the frontend and backend.

As an example, the code for error-handling mysql-queries looked like this:

```
(error, results, fields) => { 
        if (error != null) { 
          reject(error); 
        } 
          resolve(results);
}
```
This really made the team scratch their heads, since when looking at the code, the team fully expected the error handling to automatically stop running anything else if an error occurred, since the code used "reject".

However, the team then (to our shock and horror) discovered, that the error-handling actually did not work or function as intended. The team refactored the above code into:

```
(error, results, fields) => { 
        if (error != null) { 
          reject(error); 
        } else { 
          resolve(results);
  }
}
```

This solved the issue! Now, when the backend would receive a request and the MySQL-server was shut down, the backend would log the error to kibana, continue to work and most importantly not crash!

The team then realised, that while they expected the "reject"-function to automatically stop the rest of the function if an error occurred, the right way to make that code to work with the if-else statement simply required the "reject"-function to be returned, if an error was found.

This meant that the code was once again refactored, now with the missing "return"-statement.

```
(error, results, fields) => { 
        if (error != null) { 
         return reject(error); 
        } 
          resolve(results);
     }
```

This resulted in the team going through all of its mysql-queries and fix the missing "return"-statement, as can be seen in the commit below:

[Added return for rejects](https://github.com/BingoBois/DolphinNewsNode/commit/e40dadd5dba45de51bc1cec8ef266ca7a20b0c38)

The team also made a custom solution for alerting without grafana. Instead of going through grafana, the team created a function using "gmail-send" in the "DevOps" part of the project, to send out emails if certain criteria or thresholds were reach, in order to warn about system errors and crashes. This can be seen here:

[https://github.com/BingoBois/DolphinNewsDevOps/blob/master/alert-watcher/alertwatcher.js](https://github.com/BingoBois/DolphinNewsDevOps/blob/master/alert-watcher/alertwatcher.js)


## The lesson

After inserting the missing "return"-statements to the mysql-function, the system correctly logged errors to kibana and no longer totally crashed the backend if a request came in and the MySQL-server was offline.

The important lesson of this assignment is to be even more thorough and be sure to check and test the systems error-handling, especially if you plan to add functionality on top of your normal error-handling, such as error-logging.

That such an oversight could happen to this group and this project is a bit funny, since the team opted to write this project in Typescript in order to enforce strict typing in Javascript, in order to prevent mistakes such as missing code precisely as this.


## Side note…

We tried to get in touch with the group that we monitors (group f) in order to do this assignment, but was unsuccessful, despite previous success.

We raised the following issues with them:

[URGENT: We need to crash your party!](https://github.com/edipetres/HackerNews-GroupF/issues/55)

[Access Prometheus & Grafana](https://github.com/edipetres/HackerNews-GroupF/issues/52) \


The team themself had posted the following issues:

[Install prometheus on EC2 instance](https://github.com/edipetres/HackerNews-GroupF/issues/39) [Prio 1 - must have](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Prio+1+-+must+have%22) [Size 2 - medium](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Size+2+-+medium%22)

[Add monitoring to backend on AWS](https://github.com/edipetres/HackerNews-GroupF/issues/15) [Prio 1 - must have](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Prio+1+-+must+have%22) [Size 1 - small](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Size+1+-+small%22)

The team had done following commits, in relation to logging:

[add prometheus binaries and start it in npm scripts;](https://github.com/edipetres/HackerNews-GroupF/commit/8d03c3fc8ff5326c0531b9960b0d96f8e245c465) …

[fix prometheus start with sudo command](https://github.com/edipetres/HackerNews-GroupF/commit/eb0b81638917f8e6a23820e5b5f24e762cc5a025)

[fix prometheus install script](https://github.com/edipetres/HackerNews-GroupF/commit/8719e0b42a863fc0302c388d7424b34dc8d41c84)

[add permissions and remove sudo to fix prometheus setup](https://github.com/edipetres/HackerNews-GroupF/commit/ad2261ca2acd4d254b3c1a7dcf6f10e2734b38a0)

[Adding winston logs](https://github.com/edipetres/HackerNews-GroupF/commit/6d5d798ba4da34d435828ca52a155a1db20f2f73)

[continued work on updating console to logger](https://github.com/edipetres/HackerNews-GroupF/commit/4be4d63eccc19fd42e9ae56b8cc85425183acc00)

[Updated all existing console messages to logger](https://github.com/edipetres/HackerNews-GroupF/commit/07e3f5c94d159e6dc489d7d9584d1f10f1cb9514)

We had a small conversation with the group that is (supposed) to be monitoring us regarding our crash, in which we exchanged experience with the assignment, and we walked them through how we had crashed the system, our grafana and Kibana and how we ultimately fixed the issue. 
