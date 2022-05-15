---
title:  "Make Reports Great Again - a tool to write quick and beautiful pentest reports"
---
A wise guy in Discord once said "a pentester is good as his report". I agree. 

I'm a big beliver that a good report make all the difference in a good or bad pentest. 

So I decided to write a tool to assist the creation of reports. 

![](/assets/images/2022-05-15-20-44-53.png)


Make Reports great again is usefull when:

1. you want to keep your pentest organized and want to use a web interface
2. you want to keep a score of all your findings
3. you don't want to store your notes online (MRGA it's a local-only webapp - use at production at your own risk)
4. you want to provide to your client something interactive 


It creates a browserable interface for the client project (please don't use in production!)

## screenshots
![](/assets/images/2022-05-15-19-29-58.png)
![](/assets/images/2022-05-15-19-47-50.png)
![](/assets/images/2022-05-15-19-48-36.png)
![](/assets/images/2022-05-15-20-38-13.png)
![](/assets/images/2022-05-15-20-39-28.png)
![](/assets/images/2022-05-15-20-44-53.png)
![](/assets/images/2022-05-15-20-46-40.png)
![](/assets/images/2022-05-15-20-47-30.png)

## functions

1. support markdown 
2. support dataleaks
3. interactive dashboard
4. export functions from webui (zip, pdf, csv, etc.)
5. Table with searchable and sortable found vulnerabilities  
6. Admin panel 
7. User panel 
8. whitelabel reports (use your logo)
9. multiple project for each user 
10. assign a score on each vulnerability (1/10)
11. you can add a mitigation for each found vulnerability 


# personalization

you can navigate on "templates/core" folder and edit everything you like. 

For example, on the "project_general_report.html" file you can add your company logo and personalize the CSS. 

# installation 

[github repo](https://github.com/goodguyandy/make_reports_great_again)
clone the repo and run ./install.sh and follow the instruction on screen to create the superuser. 

Then, launch ./run.sh to start the server on localhost:9000 (again, do not use in production!)

# usage 

Navigate to http://127.0.0.1:9000/admin/ , login with the user you created before and create your  first project. 

Then, from the admin panel, you can create entires (vulnerabilities you found) and assign to the projects. 
Or/and you can create reports, that are basically step by step exploitation reports where you can use markdown. 


You can also create addictional, low privileges user that can access only specific projects.

In the next days I will create a more detailed documentation. 



# usage
# next version 
1. automap to CVE database (autoupdate)
2. automap to MITRE Att&ck framework 
3. add company logo from UI