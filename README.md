# Logging-and-PostMortem
# The results



*   **Summary; briefly describing what happened, at what time (beginning/ending) the system was affected, time-zone, who (which users) was affected **


We've built our system using Kubernetes to facilitate hosting and building of our backend and frontend, as well as our DevOps.

While we seemed to have build a solid and secure system, actually getting the thing to crash was a tougher challenge than initial expected. Since we hadn't encounter any particular system crashing errors during development, we had to come up with a way to manually crash the system. 

Our solution to force a crash in the system, was to try and shutdown the MySQL database and see how the backend would react.

This would result in the frontend not being able to login, post data or comments, or for Helge's simulator to post any comments or post, or check-in on /latest.



![alt_text](images/Logging-and0.png "image_tooltip")


*Screenshot of the grafana-dashboard*

As can be seen on the grafana dashboard screenshot above, the 4 pods responsible for hosting and containing the backend went offline 3 times: 18:13-18:14, and subsequently 2 times (briefly) at 18:47 and 19:08

We raised the following issues:

[URGENT: We need to crash your party!](https://github.com/edipetres/HackerNews-GroupF/issues/55)

[Access Prometheus & Grafana](https://github.com/edipetres/HackerNews-GroupF/issues/52) 


The team themself had posted the following issues:

[Install prometheus on EC2 instance](https://github.com/edipetres/HackerNews-GroupF/issues/39) [Prio 1 - must have](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Prio+1+-+must+have%22) [Size 2 - medium](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Size+2+-+medium%22)

[Add monitoring to backend on AWS](https://github.com/edipetres/HackerNews-GroupF/issues/15) [Prio 1 - must have](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Prio+1+-+must+have%22) [Size 1 - small](https://github.com/edipetres/HackerNews-GroupF/issues?q=is%3Aissue+is%3Aopen+label%3A%22Size+1+-+small%22)

Commits 

[add prometheus binaries and start it in npm scripts;](https://github.com/edipetres/HackerNews-GroupF/commit/8d03c3fc8ff5326c0531b9960b0d96f8e245c465) …

[fix prometheus start with sudo command](https://github.com/edipetres/HackerNews-GroupF/commit/eb0b81638917f8e6a23820e5b5f24e762cc5a025)

[fix prometheus install script](https://github.com/edipetres/HackerNews-GroupF/commit/8719e0b42a863fc0302c388d7424b34dc8d41c84)

[add permissions and remove sudo to fix prometheus setup](https://github.com/edipetres/HackerNews-GroupF/commit/ad2261ca2acd4d254b3c1a7dcf6f10e2734b38a0)

[Adding winston logs](https://github.com/edipetres/HackerNews-GroupF/commit/6d5d798ba4da34d435828ca52a155a1db20f2f73)

[continued work on updating console to logger](https://github.com/edipetres/HackerNews-GroupF/commit/4be4d63eccc19fd42e9ae56b8cc85425183acc00)

[Updated all existing console messages to logger](https://github.com/edipetres/HackerNews-GroupF/commit/07e3f5c94d159e6dc489d7d9584d1f10f1cb9514)
