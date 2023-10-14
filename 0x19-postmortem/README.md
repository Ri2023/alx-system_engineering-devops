
# Postmortem:

Upon the release of ALX's System Engineering & DevOps project 0x19 at approximately 06:00 West African Time (WAT) in Nigeria, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server resulted in 500 Internal Server Errors, whereas the expected response was an HTML file defining a simple Holberton WordPress site.

## Debugging Process:

Our bug debugger, Bamidele (let's go with my real initials - BL),
encountered the issue after opening the project around 19:20 PST and was 
directed to address it. He promptly initiated the problem-solving process.

1. Checked running processes using the "ps aux" command. Two "apache2" processes, 
one running as "root" and the other as "www-data," were found to be running properly.

2. Inspected the "sites-available" folder in the "/etc/apache2/" directory. 
It was determined that the web server was serving content located in "/var/www/html/".

3. In one terminal, ran "strace" on the Process ID (PID) of the "root" Apache process. 
In another terminal, performed a "curl" on the server, hoping for useful information. 
Unfortunately, "strace" provided no valuable insights.

4. Repeated step 3, this time focusing on the PID of the "www-data" process. 
Expectations were more conservative this time, but it paid off. "Strace" revealed 
an error of "-1 ENOENT" (indicating "No such file or directory") when attempting 
to access the file "/var/www/html/wp-includes/class-wp-locale.phpp".

5. Examined files in the "/var/www/html/" directory individually, using Vim pattern 
matching to identify the erroneous ".phpp" file extension. The issue was located 
in the "wp-settings.php" file (specifically, Line 137: "require_once
( ABSPATH . WPINC . '/class-wp-locale.php' );").

6. Removed the extra "p" from the line.

7. Tested the server with another "curl." It returned a 200 status code - all good!

8. Crafted a Puppet manifest to automate the resolution of this error.

## Summation:

In summary, the outage was caused by a simple typo. Specifically, the WordPress 
application encountered a critical error in "wp-settings.php" when attempting 
to load the file "class-wp-locale.phpp." The correct file name, situated in the 
"wp-content" directory of the application folder, was "class-wp-locale.php." 
The fix involved a straightforward correction of the typo by removing the trailing "p."

## Prevention:

This outage was not a web server error but an application error. 
To prevent such outages in the future, please consider the following:

* Thorough Testing: Ensure comprehensive testing of the application before deployment. 
Early testing could have revealed and addressed this error.

* Status Monitoring: Implement a status monitoring service, such as UptimeRobot, 
to promptly alert you to website outages.

Note that in response to this error, I created a Puppet manifest called 
"0-strace_is_your_friend.pp" to automate the resolution of identical errors 
should they occur again in the future. This manifest replaces any "phpp" 
extensions in the file "/var/www/html/wp-settings.php" with "php."

Of course, we all strive for error-free work, but as programmers, 
we do acknowledge that occasional errors can happen! 😉
