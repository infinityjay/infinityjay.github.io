---
title:  Gitlab config automation test with mysql service
categories:
  - CI/CD
tags:
  - Gitlab
  - Automation test
  - Project experience
---
Content

{% include toc %}

# Preface

When using continuous integration (CI) in gitlab, you often need to test the API gateway first, and in the process of testing, you need to use a local database (such as MYSQL, Redis, etc.). In a deployment of automated testing, the author needs to use mysql. After stepping on countless pitfalls, he finally successfully deployed it, involving some invisible pitfalls.

First, in the official document of using mysql in gitlab-ci ([document address](https://docs.gitlab.com/ee/ci/services/mysql.html)), two methods are introduced, one is to use `Docker executor`, and the other is to use `Shell executor`. If you use `Shell executor`, you need to install mysql in the environment, run mysql scripts, etc. Some mirror versions cannot be successfully installed, such as centos 7. This situation will be very tricky, and there are many pitfalls in the connection work after the installation. Therefore, it is recommended that everyone use the `Docker executor` method to deploy a mysql service.

However, in the official document, only a few lines are given to explain some basic deployment matters (as shown below), and the given [sample project](https://gitlab.com/gitlab-examples/mysql) is also very unreliable. The author was also confused after reading this example when using mysql service for the first time. So next, I will explain in detail how to successfully deploy a mysql service and successfully connect to the database.

![image-20210927151729449](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927151729449.png)

# Configure mysql service in .gitlab-ci.yml

## Configuration file

First, let's directly give the specific configuration of .gitlab-ci.yml in a project used by the answerer:

```yaml
variables:
MYSQL_DATABASE: abc
MYSQL_ROOT_PASSWORD: 123
stages:
- Test
- Build
- Deploy
services:
- mysql:5.7
connect:
stage: Test
image: mysql:5.7
script:
- echo "SELECT 'OK';"
- mysql --user=root --password="$MYSQL_ROOT_PASSWORD" --host=mysql "$MYSQL_DATABASE" --ssl-mode=REQUIRED
tags:
- uaek-c1
```

## Tips for avoiding pitfalls - important!!!

Next, I will focus on some points that need attention:

* MySQL is deployed separately

During the previous deployment process, the answerer placed the entire service part in the test part, which eventually led to failure. One thing to note here is that the service needs to be deployed separately, and the connect also needs to be deployed separately. Later in the CI process, the test part and the connect part should be displayed separately:

![image-20210927152157397](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927152157397.png)

* MySQL should specify the version as much as possible

I am using MySQL version 5.7 here. If you do not specify the version and use MySQL directly, the latest version (MySQL 8) will be used by default, and some unexpected bugs may occur. Therefore, in order to control bugs, it is best to use version 5.7 of MySQL.

* Be sure to configure the password

Be sure to configure the password!

Be sure to configure the password!

Be sure to configure the password!

Important things should be said three times. If the password is not configured, this error will be reported in the k8s environment I use:

![image-20210927152600239](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927152600239.png)

Then you will be confused. The pitfall is that the error reported is not the lack of password, but directly tells you that you cannot connect. In order to solve this problem, I even suspected that port 3306 in the environment was not open. I made many useless attempts and finally found that it was necessary to configure the password.

* Very important parameter `--ssl-mode=REQUIRED`! ! ! ! !

This is a very hidden pit. After countless attempts, it kept reporting this error to me. After adding the password, the following two errors (`ERROR 2002` and `ERROR 1045`) will still be reported.

![image-20210927152906091](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927152906091.png)

![image-20210927153710416](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927153710416.png)

That is to say, mysql in docker uses socket by default. However, in the k8s environment, it may be because of the lack of supporting files and the inability to connect using the socket method, so this error is reported endlessly. In order to solve this problem, I have read various articles on large platforms such as CSDN and stackoverflow, but still have not found a solution to the problem.

Finally, I finally saw `ERROR on an inconspicuous website ([website address](https://www.percona.com/blog/2019/07/05/fixing-a-mysql-1045-error/)) 1045`Solution:

![image-20210927153523366](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927153523366.png)

This website lists 7 possible problems and corresponding solutions in great detail. Combining all the previous errors, I immediately saw the 6th one, which is to specify the use of SSL:

![image-20210927154009879](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210927154009879.png)

So I added a `--ssl-mode=REQUIRED` after the statement to test the connection to the database, and finally it showed pass. After going through so many troubles, I couldn't hide my excitement!

Finally, I hope everyone will no longer be tortured by configuration files and environments.
