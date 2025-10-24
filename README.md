# Securing AWS Credentials with Secrets Manager after Sanitizing Git History
This demonstrates why hardcoding your credentials in AWS is bad practice, what happens when you try to push it to Github, and how to safely secure credentials using AWS Secrets Manager. **The Credentials in this demo are fake and/or edited!** This matters because hardcoding credentials in your work can put your AWS environment at risk of having data stolen, deleted work, and etc. Therefore the BEST practice is to safely secure your credentials i.e. Secrets Manager.


## How It's Made👨🏿‍💻 :
**Resources used:** GitHub, AWS IAM, and Secrets Manager

<img width="800" height="608" alt="image" src="https://github.com/user-attachments/assets/1e612795-23ba-4a37-b190-31a5dcb842ae" />


I used an existing sample application by Nextwork to replicate a common vulnerability by inserting hardcoded test credentials (in **config.py**) such as the key id and access secret key (**these aren't real credentials** and this is TERRIBLE practice, so don't do it 😂). This is bad practice because bad actors can obtain these credentials, access the AWS Environment, delete, steal data, and etc.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/481ptp0qb4qe039shmdm.png)


Once I entered the test credentials, I decided to launch the web app locally on my machine. To set this up, I launched a virtual env where i needed to install packages such as boto3, fastapi, uvicorn, and etc that the packages main file (**app.py**) uses to connect the web app to AWS. Next, once I was able to see the app on my machine, I went to the "**localhost:8000**" (since I'm on Windows 💀 ), clicked on View My S3 Buckets, and then received an error. That was because I used the test credentials and it couldn't call on the 'ListBuckets' operation.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v2glj8isy1gom52ua4yt.png)



![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4zhhjl3me0sv9i5lpy8n.png)

This is when I decided to use my own AWS Credentials (access key ID & secret access key) so i could properly connect to the web app and have my buckets listed. To set this up I went to IAM, clicked the "Users" tab, and selected "iamadmin"

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4jo3gqv1zr6exkd4d1id.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/res2j1tismda3wk0cu57.png)




- Clicked the "Security credentials" tab

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vq5auwp66b7ivkirf1jt.png)



- This revealed the Access Keys and then I clicked on "Create access key"

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vl5pgl11onrbn2uu7qur.png)



- Once there, I selected Local Code (because I'm running this locally on my machine) and then clicked "Next"


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h93xz008pxwwg2jrcwyh.png)





- The next screen shows a "Set description tag" but it is optional. I did it anyway just so I could keep it organized.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kcdstb9nhep92xwo74uj.png)



- My keys have now been generated and I downloaded them as a .csv file because I knew I would somehow accidentally close the tab with the credentials. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ncjze1v6henusg00vuk9.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r4cv6mn4273v41ourt5s.png)



Now that I have my keys, I shut down the app in my terminal from running on my virtual environment. When that went down, I edited **config.py** again but this time using my REAL credentials because now I should be able to connect into AWS right? So I Started the web app in the terminal again with "python app.py", went to the "**localhost:8000**", clicked on "View My S3 Buckets", and **🪄VOILÀ**...the buckets in my account have appeared!


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5jyyg179dcb2nipxoh5v.png)

Now that my AWS account is connected to the app, I went to Nextwork's Github account that had the original repo and forked that over to my account and then headed back to the terminal to shut down the web app once again so i could configure git and stage the changes I made to the **config.py** earlier.

As my changes were pushing I ran into this problem:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f6vdg53ke7m8n3l2bhr6.png)

   
This is because my actual AWS Credentials were hardcoded in the **config.py** and Git was able to detect this with their "Secrets Scanning" too. It basically said "Hey man, we can't let you push that through because those are credentials that you REALLY don't want out in the open so we're going to halt on that push until you figure out how to securely store these." Now the question I'm asking myself is "How do I safely secure these then?🤔" Well, that's where AWS Secrets Manager comes into play 🤓 !  

 
To set this up I go back to the console and type in the search bar "secrets" and Secrets Manager appeared

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qbwsnzkf2w1k5zvua9hm.png)

- Clicked on "Secrets"

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sphkr418tvclqr2v8bwv.png)


- Store A New Secret

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/diuj0d4escwxuqbc08ol.png)

- Set 2 Key/value

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mg65amwr54arikr57c6k.png)

- Configured my secret


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jsamtah3fzf3xteblnv1.png)

- I then copied the sample code (Python) it generated from my secrets

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oj1u9cj25qn5n4swp0yb.png)


Opened **config.py**, deleted all the lines that had my hardcoded AWS Credentials, and pasted the sample code that I copied in secrets in place of it. I also added some other code including the "import json" on line 3 and lines 27-35.
 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ol1tj686fey4uf6fhd7y.png)

It's time to stage my changes again with "git add ." After I attempt to  push the changes, I'm ONCE AGAIN I'm met with:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f6vdg53ke7m8n3l2bhr6.png) 

I eventually figure out that just because I changed the **config.py** by removing my hardcoded credentials and replacing them with with a secrets code that referenced my keys in AWS, I'm still not able to push my code. That's because that's not enough...Github keeps a history of all the changes you've made to your files and in this case, my commits. Next logical thing to do is some how erase the history of me committing hardcoded credentials which is where "git rebase" is useful.

Looking at the terminal again now had to find which commit had my hardcoded credentials in the history. I spotted it (the 1st 7 digits were all I needed), typed "git rebase -i --root" in the terminal, and found the 7 digits along with "Updated config.py with hardcoded credentials" at the end. This is what I need to delete to be able to push my changes.  

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zqixshkpmmdd8bodian5.png)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/brqprc1d7g0ixmnfh2nk.png)


To get rid of the " pick 8f74393 Updated config.py with hardcoded credentials" I  scrolled down to where it says "pick" next to it and pressed 'd' TWICE and then ':wq' to save the change. 


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1etrrw9glw0df7ai7gty.png)


_The first time I did this I pressed 'd' one time, I didn't see anything happening, I then read somewhere to press the down arrow, I tried that, then that deleted the "Updated config.py with hardcoded credentials" AND "Updated config.py to use secrets manager", didn't think anything of it, and typed ':wq'. THIS RESULTED INTO MY ENTIRE FILES BEING DELETED AND I HAD TO START OVER 🤬🤬 ._ 


After saving the the rebase changes, merging was happening but then I ran into a merging conflict. To resolve this, I had to open the config.py file and delete the lines that contained my hardcoded credentials and 2 other lines (======= and >>>>>>> feature-branch). After that, I saved the file, ran 'git add config.py' and 'git commit -m "Resolved merge conflicts"'. After changes were made, I continued the rebase, pushed the code, and done! The changes successfully went through to Git! 


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j358srurffqbd3nt44qa.png)


To check my work I did go into **config.py** in Git to make sure the secret code generated in my secrets replaced the hardcode from earlier and it did. 


## Lessons Learned 🧠 :

I learned that hardcoding keys is BAD practice because anyone can get into your AWS environment since your keys are right there. After replicating this twice, securing your keys is very easy but the task can be tedious if you push your code in git and it discovers you're hardcoding your credentials because now you have to erase your history of doing so if you securely store them afterwards. It's best to default to using secret manager and secrets in general so there's no panic if credentials accidentally are leaked. I also learned TO NOT d and down arrow if you only mean to delete 1 commit. I did learn that ':q!' prevents the deleting of commit so that saved me the second time around because I almost made the first mistake again. 


