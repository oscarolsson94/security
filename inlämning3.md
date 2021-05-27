## Exploit

1. Visit the url where you can see a single flag:  `/flag`
2. Click on one of the flags.
3. To get access to the secret file containing API-keys, enter "../" in the URL after the parameter `?name=`. It should look like this: `/flag?name=../keys.xml`
4. Press enter, and u will see the contents of the secret XML-file on your screen.

## Vulnerability

The vulnerable lines of code are in the `singleFlagPage` method:
```
private static void singleFlagPage(Context context) throws IOException {
        String flagName = context.queryParam("name");
        Path path = Path.of("flags/" + flagName);
        String svg = Files.readString(path);
        context.contentType("image/svg+xml; charset=UTF-8");
        context.result(svg);
    }
```
We are not checking the users input before actually reading the file and displaying it in the browser. Therefore we allow the user full access to our entire file system. If the user decides to enter `../` either once, or even a couple of times, they would be able to see files in folders where we do not want the user to have access to.

## Fix

To fix this vulnerability, we make use of `toAbsolutePath` followed by `normalize` to get the full path of the user input without extra `../`. We also grab the actual path of the "flags" folder.
```
        String flagName = context.queryParam("name");

        Path path = Path.of("flags/" + flagNameame).toAbsolutePath().normalize();
        Path flagFolder = Path.of("flags").toRealPath();   
        ...
```
Before showing any result in the browser, me make sure to compare the path of the folder which we want the user to have access to, with the path entered by the users. If the path does not match, we throw an error telling the user they cannot access files outside the flag folder.
```
        if (!path.startsWith(flagFolder)) {
            context.status(403);
            context.result("You cannot access files outside the flag folder.");
            return;
        }

        String svg = Files.readString(path);
        context.contentType("image/svg+xml; charset=UTF-8");
        context.result(svg);

    }
```
We have now protected ourselves against malicious users, who may try to either retrieve our secret assets. We only allow the user to view files inside the flags-folder.  


## Exploit

1. Visit the url for the search page:  `/search`.
2. Enter the following into the search bar: `<script>alert('XSS!')</script>`.
3. Press enter.
4. An alert will show up, which means that we are able to write javascript directly into the input field and make it run in the browser.

## Vulnerability

The vulnerable lines of code are in the `searchPage` method:
```
 if (context.queryParam("search") != null) {
            // Show what term the user searched for.
            content +=
                "<p>Search results for: " + context.queryParam("search") + "</p>" +
                "<ul>";

            try (Connection c = db.getConnection()) {
                // Make sure to only get the quizzes that are public or belong to the current user.
                PreparedStatement s = c.prepareStatement(
                    "SELECT quiz.id AS quiz_id, title, username, public " +
                    "FROM quiz " +
                    "JOIN user ON quiz.user_id = user.id " +
                    "WHERE instr(title, ?) " +
                    "AND (public = TRUE OR user.id = ?)"
                );
                s.setString(1, context.queryParam("search"));
                s.setInt(2, context.sessionAttribute("userId"));
                ....
```
The user input coming from the search input field is directly injected into the rest of the html, and by using a <script> tag, it will therefore also be interpreted by the browser as an html-element. To be more concrete, a tag used to run javascript. Here we are simply showing that running scripts in the browser is possible by showing an innocent browser alert, but if an attacker really wanted to, they could cause serious harm to your website visitors. For example by creating fake password forms, malicious links, and a whole lot more.

## Fix

To fix this vulnerability, we make use of `Encoder`, which is a class from the Owasp library which specializes in web security. The Encoder changes the way the `<script>` tag is interpreted by the browser. Import the following: `import org.owasp.encoder.Encode`. All we need to do other than to import, is to wrap our input inside of an Encoder-object, and the input will be interpreted as just a plain String by the browser:
```
 ....       
 "<p>Search results for: " + Encode.forHtml(context.queryParam("search")) + "</p>" + "<ul>";  
 ....       
```
We have now protected ourselves against malicious users, who may try to run damaging code inside the search input field on our website. 
        
        
## Exploit
        
1. Register an account at `/register`.
2. Log in to the website.
3. Go to `/create` and fill in the form to make a quiz.
4. In the title-input, enter `<script>alert('XSS!')</script>`.
5. Create the quiz.
6. Go to the play-page at `/play`.
7. As soon as the page loads, the script will run as it gets pulled from the database.
        
## Vulnerability
        
One could argue that you should not be able to save something like `<script>alert('XSS!')</script>` to the database in the first place, but here we are going to focus on the part of the code that actually triggers the script. The vulnerable lines of code are in the `showQuiz` function in the `main.js` file:
        
```
        
....        
questionNode.innerHTML =
            '<h1 class="quiz-title">Quiz: ' + quiz.title + (quiz.public ? '' : ' [private]') + '</h1>' +
            '<figure class="flag">' +
                '<img src="/flag?name=' + question.image_path + '">' +
            '</figure>' +
            '<h2 class="prompt">' + question.prompt + '</h2>'   
        
....        
        
```    
       
The `quiz.title` variable, which we now know can contain a malicious script, is grabbed from the database and then directly injected into the html with the use of `innerHTML`. This means that at this point, the potentially malicious code is now stored in our database, and will run every single time ANY user visits the website. This is called a *Persisted* XSS exploit, as it is saved to the server, and hence it will run for anyone making a request to the site. 
        
## Fix        
        
In order to fix this issue, I will provide two different solutions. The first one is very similar to the last XSS exploit we talked about, as we will use something very similar to the `Encoder`-class which we used in Java. Since that is a Java library however, we cannot use that here. As Javascript is "the language of the web", there is a very neat way in which we can accomplish the same thing. In the standard Javascript library we have a function called `encodeURI`. This will accomplish the same thing, escaping dangerous characters that we don't want on our website.
        
```
....         
questionNode.innerHTML =
            '<h1 class="quiz-title">Quiz: ' + encodeURI(quiz.title) + (quiz.public ? '' : ' [private]') + '</h1>' +
            '<figure class="flag">' +
                '<img src="/flag?name=' + question.image_path + '">' +
            '</figure>' +
            '<h2 class="prompt">' + question.prompt + '</h2>'
....               
```
This way, all the potentially harmful characters will be replaced with new characters, whom in turn cannot be inte        
        

Another way of protecting against this exploit, would be to replace the usage of `innerHtml`, and instead creating your elements with DOM methods together with `textContent`. In my opinion, this is a bit tedious as it requires a lot of boiler plate code and the solution is not very modern. Solving it this way would look something like the following:
        
```
const h1 = document.createElement('h1');  
h1.classList.add("quiz-title"); 

const h1Content = 'Quiz: ' + quiz.title;
if(!quiz.public) h1Content += "[private]";
        
h1.textContent = h1Content;
        
const figure = document.createElement('FIGURE'); 
figure.classList.add("flag");
        
const img = document.createElement('IMG'); 
img.src = "/flag?name=" + question.image_path;        
figure.appendChild(img)
        
const h2 = document.createElement('h2');
h2.textContent = question.prompt;  
        
questionNode.appendChild(h1); 
questionNode.appendChild(figure);  
questionNode.appendChild(h2);          
        
```
By doing it this way, `textContent` basically provides the same protection when it comes to swapping out the potentially harmful characters.

## What are the key points and principles I will take away from this course?
        
This course has been a real eye-opener for me when it comes to cyber security. I have learnt a whole lot, even though at the same time I realize that we have nothing but scratched the surface of the area which is web security. Just the sheer amount of already existing exploits really makes you think about the current state of the web, and how secure or not secure todays applications really are. On top of the already known exploits and vulnerabilities, hackers and other types of malicious users are constantly evolving, and experimenting to find new ways to exploit.
        
It is in my, and many others opinion pretty much impossible to make and application completely secure. Sure, you can implement an unlimited amount of steps for the user to identify themselves, to try and make sure that no malicious user can get access to some kind of private information. However, a key point which I will definitely keep in mind from this course, is that security always has to be balanced together with user experience. Absolutely no one wants to go through ten steps of verification in order to log in to a simple application. On the other hand, some applications, such as online banking and things alike, might not get by without a very robust security layer. Finding the perfect balance between security and user experience is a difficult task, and one that I believe can only come from experience.
        
After finishing this course, my view is that as a developer it should be mandatory to have atleast some kind of general knowledge about how to protect an application before you actually start working on any kind of real life application. Before taking this course, I had done some research on my own on the subject of securing an application. I had read about path traversal and SQL-injections, and also heard about some XSS exploiting techniques. During the last couple of months I have improved my knowledge a lot within the subject, and I also feel like I started thinking differently while writing code. Writing functional code that works is one thing, but having security in mind while coding changes the entire flow of it. It was a necessary step to take, and even though it might come along with a few headaches, I know that it will be benefitial for me and the work I do in the future.
        
I thoroughly enjoyed the quest lecture we had regarding real life exploiting and the work that goes into testing the security of real life companies. I think everyone has heard about or seen the infamous black hat hackers from the news or as they are portrayed in action movies. What I found really interesting was the work of white hat hackers, and how companies would either hire them, or put out bounties for anyone finding flaws in their systems.         
        
