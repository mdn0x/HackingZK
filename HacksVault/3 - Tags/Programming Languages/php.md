PHP is an open-source server-side scripting language that many devs use for web development. It is also a general-purpose language that you can use to make lots of projects, including Graphical User Interfaces (GUIs).

In this article, I will help you explore the world of PHP so you can learn how it works and its basic features. By the end, you will be able to write your first Hello World program in PHP.

## What Does PHP Mean?

The abbreviation PHP initially stood for Personal Homepage. But now it is a recursive acronym for Hypertext Preprocessor. (It's recursive in the sense that the first word itself is an abbreviation, so the full meaning doesn't follow the abbreviation.)

The first version of PHP was launched 26 years ago. Now it's on version 8, released in November 2020, but version 7 remains the most widely used.

PHP runs on the Zend engine, which is the most popular implementation. There are some other implementations as well, like parrot, HPVM (Hip Hop Virtual Machine), and Hip Hop, created by Facebook.

PHP is mostly used for making web servers. It runs on the browser and is also capable of running in the command line. So, if you don't feel like showing your code output in the browser, you can show it in the terminal.

## Advantages of PHP

PHP has some advantages that have made it so popular, and it's been the go-to language for web servers for more than 15 years now. Here are some of PHP's benefits:

- Cross-Platform: PHP is platform-independent. You don't have to have a particular OS to use it because it runs on every platform, whether it's Mac, Windows, or Linux.
    
- Open Source: PHP is open source. The original code is made available to everyone who wants to build upon it. This is one of the reasons why one of its frameworks, Laravel, is so popular.
    
- Easy to learn: PHP is not hard to learn for absolute beginners. You can pick it up pretty if you already have programming knowledge.
    
- PHP syncs with all Databases: You can easily connect PHP to all Databases, relational and non-relational. So it can connect in no time to MySQL, Postgress, MongoDB, or any other database.
    
- Supportive Community: PHP has a very supportive online community. The official documentation provides guides on how to use the features and you can easily get your problem fixed while stuck.
    

## Who Uses PHP

A number of established companies and tech giants use PHP to run their servers and make a lot of incredible things.

- Facebook: Facebook uses PHP to power its site. In turn, the company contributed to the community by creating an implementation known as Hip Hop for PHP.
    
- Wikipedia: one of the world's largest sources of information on any topic, Wikipedia is built in PHP.
    
- Content Management Systems (CMSs): the world's most popular content management system, WordPress, is built in PHP. Other content management systems such as Drupal, Joomla, and Magento are also built in PHP. Shopify runs on PHP too.
    
- Web Hosting Platforms: a lot of Web Hosting Platforms such as BlueHost, Site ground, and Whogohost run their hosting servers using PHP.
    

## Is PHP Dying?

Nowadays, there's an intense debate on whether PHP is on the decline or not. This is because of the advent and increasing popularity of other languages suited for the server-side such as JavaScript (Node JS), Python, Golang, and others.

This has actually led to a lot of funny memes targeting PHP ![php-meme](https://www.freecodecamp.org/news/content/images/2021/08/php-meme.jpg)

But is PHP really dying? The answer is no. Despite some people bashing on it and the claims of decline, PHP is still used to run the servers of almost 80% of all websites today. So, if you visit 10 websites a day, there's a chance that 8 of them use PHP.

In terms of job availability, PHP ranks better than a lot of other programming languages on the job platform Indeed. A lot of PHP developers make a good living making WordPress themes and plugins every year – the average PHP developer in the US makes $86,000 per year.

![php-jobs](https://www.freecodecamp.org/news/content/images/2021/08/php-jobs.jpg)

## How to Write Your First Hello World Program in PHP

Now that you have learned about PHP and its advantages, it's time to write your first Hello World program in it!

First of all, you must have PHP installed on your local machine. You can get that done by installing an XAMP (Cross-Platform, Apache, MySQL, and PHP) or WAMP (Windows, Apache, MySQL, and PHP) server.

XAMP works on all operating systems and WAMP works only in Windows. I will be using WAMP.

Open up the WAMP or XAMP server and make sure all services are running. If you are using WAMP, the WAMP logo should show on your taskbar with the color green.

![wamp-running](https://www.freecodecamp.org/news/content/images/2021/08/wamp-running.jpg)

Open up your `C` drive and look for the installation directory of your WAMP server. In my case, it is `wamp64`.

![wampfolder](https://www.freecodecamp.org/news/content/images/2021/08/wampfolder.jpg)

Open the installation directory, and then the `www` folder.

![wwwfolder](https://www.freecodecamp.org/news/content/images/2021/08/wwwfolder.jpg)

Create a folder right there and name it whatever you want, then open up the folder with your code editor.

Create an `index.php` file and paste in the following code:

```php
<?php

echo "Hello World";

?>
```

You can also put your “Hello World” text in a variable, then use the echo system to display it in the browser.

In PHP, you can declare a variable with the dollar sign ($). Your statements, apart from the last one, must also be terminated by a semi-colon.

```php
<?php

$greeting = "Hello World";
echo $greeting

?>
```

To run your code in the browser, open up the browser and write this in the address bar `localhost/the-folder-of-your-php-file/php-file.php`, then hit enter.

Make sure your WAMP or XAMP server is running, otherwise it won’t work.

![hello-world](https://www.freecodecamp.org/news/content/images/2021/08/hello-world.jpg)

You can see that the code successfully ran in the browser, because I got the file path right.

Another beautiful thing about PHP is that you can embed it in HTML. You can do it like this:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHP Code</title>
</head>
<body>
    <h1> This is the result of a PHP Code embedded in HTML</h1>

        <?php 
            $greeting = "Hello World";
            $campers = "Hello Campers";

            echo $greeting;
            echo "<br>";
            echo $campers
        ?>
</body>
</html>
```

![php-in-html](https://www.freecodecamp.org/news/content/images/2021/08/php-in-html.jpg)

## Conclusion

PHP remains a relevant and widely-used language in web development. Despite the mockery and debate on whether it’s still valuable, PHP developers keep earning good livings from working with the language. So, PHP doesn't seem to be going anywhere anytime soon.

Now, go code some PHP!

Thank you for reading, and keep coding.