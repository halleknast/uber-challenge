Uber challenge: San Francisco Movies
------------------------------------

[Project description](https://github.com/uber/coding-challenge-tools/blob/master/coding_challenge.md):

> Create a service that shows on a map where movies have been filmed in San Francisco. The user should be able to filter
  the view using autocompletion search.
> 
> The [Film Locations data set](https://data.sfgov.org/Arts-Culture-and-Recreation-/Film-Locations-in-San-Francisco/yitu-d5am)
> is available on [DataSF](http://www.datasf.org/).

### Implementation

The project was implemented in Go and [deployed](https://uber-challenge-148819.appspot.com) on Google's App Engine as my
first experience with both of these technologies.

The data is stored in a MySQL database, which on the App Engine is called CloudSQL.

Currently, the only part of the application that has been implemented (and only partially), is the part of the backend
that initializes and updates the movie data.

### Life cycle

When the application starts up, it checks if the database is empty and initializes it with data from a cached file if it
isn't. When the `/update` endpoint is hit with a HTTP POST-request, the database is cleared and reinitialized with fresh
data from the data set link above. For robustness, the "original" initialization also happens if the database is
suddenly empty (i.e., we can delete and recreate it from the console without restarting the application).

The tables sizes of the SQL database and the log/error of the last (re)initialization/update are accessible on the
`/status` endpoint.

### Setup

0.  Install the Go App Engine SDK (has batteries included) using instructions found elsewhere.
1.  Create app engine trial account, give Google your credit card info, and make the sign of the cross.
2.  Create project in the console
3.  Create CloudSQL instance in the console. Make sure it's in the correct location (cannot be changed later) and make
    sure that your app is authorized to access it (should be automatic).
4.  Open the Cloud Shell (upper right corner of the console). Connect to the database using the command
    `gcloud beta sql connect [DB-instance-ID]`. Then create the database `locations` using the MySQL command
    `create database locations;`.
5.  After changing the `application` field of `app.yaml` to the ID of your project, deploy the app using the local
    command `goapp deploy`, executed from the root directory of the project.

Running locally:

*   Install a MySQL server (the project was developed with version 5.7, which is also the version used on CloudSQL).
*   Add a file named `data-source-name` in the `res` (resource) directory. The contents on the file should be a string of
    the format `root:[root-password]@/locations` without any newlines (assuming that you use the server's root user).
*   [TODO Add description of running using `goapp` or IntelliJ with appropriate plugin(s).]

### Problems

During development, a number of problems were encountered and solved. These lessons learned have been written down in
`problems.md` for later retrieval.
