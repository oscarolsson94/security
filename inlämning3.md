## Exploit

1. Visit the url where you can see a single flag:  `/flag`
2. To get access to the secret file containing API-keys, enter "../" in the URL after the parameter `?name=`. Like this: `/flag?name=../keys.xml`
3. Press enter, and u will see the contents of the secret XML-file on your screen.

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
We are not checking the users input before actually reading the file. Therefore we allow the user full access to our entire file system. If the user decides to enter `../` either once, or even a couple of times, they would be able to see files in folders where we do not want the user to have access to.

## Fix

To fix this vulnerability, we make use of `toAbsolutePath` followed by `normalize` to get the full path of the user input without extra `../`. We also grab the actual path of the "flags" folder.
```
        String flagName = context.queryParam("name");

        Path path = Path.of("flags/" + filename).toAbsolutePath().normalize();
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

