# udacitynx
Linux setup project
i. The IP address and SSH port so your server can be accessed by the reviewer.
18.188.44.128 port 2200
ssh grader@18.188.44.128 -p 2200 -i UdacityXCourse

ii. The complete URL to your hosted web application.

iii. A summary of software you installed and configuration changes made.
upgraded packages using apt-get 
added user grade
added public key to authorized key
restricted root login
add grader to sudoers.d
changed port 22 to 2200
enable ufw for 2200, 80 and 123
installed apache2
installed mod wsgi and tested
python-pip
tested flask

iv. A list of any third-party resources you made use of to complete this project.
