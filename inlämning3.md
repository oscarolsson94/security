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
 ...       
 "<p>Search results for: " + Encode.forHtml(context.queryParam("search")) + "</p>" + "<ul>";  
 ...       
```
We have now protected ourselves against malicious users, who may try to run damaging code inside the search input field on our website. 
        
        
## Exploit
        
1. Register an account at `/register`.
2. Log in to the website.
3. Go to `/create` and fill in the form to make a quiz.
4. In the title-input, enter `<script>alert('XSS!')</script>`.
5. Go to the play-page at `/play`.
6. As soon as the page loads, the script will run as it gets pulled from the database.
        
## Vulnerability
        
One could argue that you should not be able to save something like `<script>alert('XSS!')</script>` to the database in the first place, but here we are going to focus on the part of the code that actually triggers the script. The vulnerable lines of code are in the `showQuiz` function in the `main.js` file:
        
```
        
        
```        
        
        
        
        
