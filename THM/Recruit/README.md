# Recruit

## Introduction

Hey! My name is Madhav and I just love doing CTFs. This is one of the few CTFs I did without reading any write ups (well there were no write ups released at the time). So this is a medium-level room on [TryHackMe][https://tryhackme.com/room/recruitwebchallenge] in which we have to find vulnerabilities in a web application named **Recruit** which basically is an application for HR to manage CVs of applicants.

# Challenge Overview

- Room: Recruit ([TryHackMe][https://tryhackme.com])
- Difficulty: Medium
- Objective: Gain admin access to the web application

# Methodology & Exploitation

I opened the target application on the browser and saw two things:
	- A Login Form
	- An API docs link for "Access API"

I ran gobuster on it too look for any other directories

![Mail-Dir][./Screenshots/Mail-Dir.png]

This mail directory looked interesting so I opened it and found a file named mail.log

![mail-log][mail-log.png]

Yeah! So found a user named hr and it's password is stored in the **config.php** file. Now we look at the target web page.

![Screenshot][./Screenshots/Login-Page.png]

I thought of bruteforcing the password for hr but I thought first I should try looking for it instead. So I moved on to the API docs for the "AccessAPI"

![AccessAPI][./Screenshots/AccessAPI.png]

I saw this API endpoint and as it also supports external URLs, I tried for SSRF on the file but it had some filters.

![File-PHP][./Screenshots/File-PHP.png]

Then I thought what if I can access config.php file from this API, so I put in the file name and BOOM! we got the password for the hr user!

![HR-PASSWD][hr-password.png]

I have redacted the password for obvious reasons, let's login as hr and get the first flag!!!

![User-Flag][./flag-1.png]

Here we can search for candidates with their names, so I immediately thought of SQL injection.

![SQLi-TRY][./sqli-try.png]

and BOOM!! it's and Error-Based SQL injection, which should not be too hard to exploit. I thought if we can get the config.php from the AccessAPI then why not the dashboard.php too, and guess what? we got the source code for dashboard.php using that API. Here we can see the SQL query used to get the users from the database.

![SQL-QUERY][./sql-query.png]

So keeping in mind the query I tried some payloads.

![TryingPayload1][./Trying-Payload-1.png]

Oh no! why does it not work! I thought for a while and tried different payloads but then I thought maybe something else is used for commenting instead of '--', then I got it! it uses '#' for commenting.

![SuccessPayload1][success-payload-1.png]

Yeah!!! we successfully found and exploit SQL injection. Now it's just the matter of dumping the whole database to get the credentials for the administrator. Let's first get the name of the table, I used **Union Based SQLi** to dump the database.

![TableName][table-name.png]

Found the table 'users', yeah it was kinda obvious ig, let's just dump the database and get the administrator credentials.

![AdminCreds][./Admin-creds.png]

Yeah!! got the administrator credentials, let's go login as administrator and get the root flag!

![AdminFlag][./admin-flag.png]

Yeah!! got the root flag for the challenge!

# Summary

## AccessAPI

- Found an SSRF vulnerability in the AccessAPI through which got the password for the user 'hr' and also found the source code for the dashboard.php which had the SQL query for the search functionality.

## Search Candidate 

- Found an SQL injection vulnerebility on the search bar to search for candidates, dumped the db to get the creadentials for the administrator user and logged in as admin to get the root flag.