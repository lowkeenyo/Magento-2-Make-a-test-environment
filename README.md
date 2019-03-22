# COPY OF YOUR MAGENTO 2.2.x ENVIRONMENT #
| Copy live environment to test through SSH to other hosting: 

__Magento 2.2+ how to clone your live webshop with minimal risk? RSYNC! Very fast way to get a staging environment going on either a sub-domain or different test host. This example goes through the cloning from host to host. Sub-domains require some more steps that are not in this example, only the rsync command.__ 
 - - - -
## STEP 1 ##
From your TEST environment backup the app/etc/env.php file for later use in step 3. 

EXAMPLE SSH COMMAND FROM BEING IN THE PARENT FOLDER OF PUBLIC_HTML: 
``` Rsync -avz public_html chemtec1@chemtechniek.mag2.skyberatedev.nl:public_html```

Copy live environment to sub-folder with SSH and exclude crap: 

``rsync -vza --delete --delete-excluded \
--exclude /var/report/ \
--exclude /var/session/ \
--exclude /var/log/ \
--exclude \*.sql \
--exclude \*.zip \
/data/web/magento2/ /data/web/magento2_staging/``
 
 Possible errorino:
 - mkdir not allowed -> on the staging make an empty folder, 750/755 folder permissions
  - Dont want to delete the main folder? rename it!
 - running out of server space -> clean up the live environment and exclude media folder?
 
--------------------------------------------------
## STEP 2 ##
(im assuming there is already an empty magento installation working on our TEST server)

Gotta copy that DATABASE from live to test! 

Dump the whole database through ssh with: 

`` mysqldump --opt -Q -h dbintxxxxx -u uxxxxx_xxxx -p dbxxxx_xxxx > dump.sql(give the database pass)``
  ("dbintxxxxx" is the hostname of live site, "uxxxxx_xxxx" is db username, "dbxxxx_xxxx" is db name)

Now now now, the dump.sql is in the root of the hosting, not the public_html(or other root of website folder). 

Next up is copy pasting that dump.sql to the test environment server, through filezilla or whatsoever. 

With ssh go to the correct folder and use: (root of host used in this example)
``mysql -h dbintxxxxx -u uxxxxx_xxxx -D dbxxxx_xxxx -p``

   ("dbintxxxxx" is the hostname of live site, "uxxxxx_xxxx" is db username, "dbxxxx_xxxx" is db name)
   
Enter DB password and you should enter the mysql side of things, try the following commands: 
1. SET FOREIGN_KEY_CHECKS = 0;
2. source dump.sql;
3. SET FOREIGN_KEY_CHECKS = 1;

Done with this now. typ exit to leave the mysql SSH thing. 

 ---------------------------------------------------------
 
 We now should have done the following: 
 1) Copy of the LIVE hosting documents and files to our TEST hosting with the thank of rsync
 2) Copy of the LIVE database into our TEST database
 
 --------------------------------------------------
## STEP 2.5 ##
 Lets take half a step, we got to adjust our LIVE database in our TEST hosting. 
 1) go to your database (phpadmin in my case)
 2) find core_config_data table and select it
 
 2.5) find web/unsecure/base_url and web/secure/base_url and make them into the TEST domain url - https for both if enabled.
 
 3) in the table go to search and input: "%cookie%" to filter on
 4) delete/replace the old URL values but be careful, keep sub-domains in mind.
 
 My site is https://www.test.mag2.skeskedev.nl/ but (IN MY SITUATION, ALWAYS TEST YOUR OWN)
 cookie_domain value is test.mag2.skeskedev.nl to make it work
 
"Delete the VALUE of the following (if you do not see some of them, just ignore. It might not have been setup yet, so you can omit it):
* web/cookie/cookie_domain
* web/cookie/cookie_httponly
* web/cookie/cookie_lifetime
* web/cookie/cookie_path"
 
  ---------------------------------------------------------
  
 So now we have: 
 1) A copy of the LIVE hosting to TEST hosting including adjusted database.
 
  ---------------------------------------------------
## STEP 3 ##
Almost there! We need to connect magento to our database now. 


app/etc/env.php - You sould have backed this file up in step 1 from the TEST hosting. 
Download the newly added rsync'd file app/etc/env.php from your TEST hosting and open both files. 

1) app/etc/env.php backup file - backed up one has information like: dbname, username, password that we need to put back into our newly added env.php file on the hosting. 
2) app/etc/env.php - new one has the LIVE data, change those three lines back to the information in the backup env.php file.

At the bottom also update the 'http_cache_hosts' to your TEST url without http or www (same as cookie_domain)

Updated your env.php file? Put it back in the app/etc/ folder on your TEST hosting. 

  ---------------------------------------------------------
 
 So now we have: 
 1) A copy of the LIVE hosting to TEST hosting including adjusted database.
 2) Connected magento with the database by adjusting ENV.PHP file to correct database information. 
 
  ---------------------------------------------------
## STEP 4 ##
Lets use SSH once more to get our test environment working correctly, going to frontend now should load the website but could be not so functional.


php bin/magento setup:upgrade

php bin/magento setup:di:compile

php bin/magento setup:static-content:deploy 
(add -f in case of developer mode and static content missing) 

 Should be all! 
 
 
## Extra informations:
1) https://www.byte.nl/kennisbank/website-uitbreidingen/testomgeving-website-maken
2) https://support.hypernode.com/knowledgebase/using-a-basic-staging-environment-magento2/#How_to_make_a_copy_of_a_live_site
3) https://www.byte.nl/kennisbank/back-ups/magento-back-up-maken-en-terugplaatsen-via-shell
4) https://magento.stackexchange.com/questions/160775/solved-magento-2-unable-to-login-to-admin-no-error-message-stuck-at-login
